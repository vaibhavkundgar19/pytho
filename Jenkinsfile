// pipeline{
//     agent any
//     environment{
//         DOCKERHUB_USERNAME = 'kundgar19'
//         IMAGE_NAME = 'pytho'
//         IMAGE_TAG = "${BUILD_NUMBER}" //har build ka alag tag hoga ex. 1,2,3,...
//     }
//     stages{
//         stage('checkout code'){
//             steps{
//                 echo 'code checkout automatically done via scm'

//             }
//         }
//         stage('Build docker image'){
//             steps{
//                 script{
//                     echo 'build docker image ...'
//                     //image ke name format hoga username/image:tag
//                     sh "dockerbuild -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:/${IMAGE_TAG}"
//                     //ek latest tag banate hai taaki deploy karna easy hojaye
//                     sh "dockerbuild -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:/latest"
//                 }
//             }
//         }
//         stage('push to dockerhub'){
//                 steps{
//                     script{
//                         echo 'pushing to dockerhub.......'
//                         withCredentials([usernamePassword(credentialsId:'docker-hub-creds',passwordVariable: 'DOCKER_PASS',usernameVariable:'DOCKER_USER')]){
//                             //login command
//                             sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
//                             //push commad
//                             sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:/${IMAGE_TAG}"
//                             sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:/latest"
//                         }
//                     }
//                 }
//             }
//         stage('deploy the container'){
//             steps{
//                 script{
//                     echo 'deploying the container...'
//                     sh "docker rm -f pytholab || true"
//                     sh "docker run -d --name pytholab -p 80:80 ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:/latest"
//                 }
//             }
//         }  
//     }
// }

pipeline {
    agent any 

    environment {
        DOCKERHUB_USERNAME = 'kundgar19'
        IMAGE_NAME = 'pytho'
        IMAGE_TAG = "${BUILD_NUMBER}" //har build ka alag tag hoga for ex. 1,2,3.....
        // sonarqube scanner tool ka path load kar rahe he
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('checkout code') {
            steps {
                echo 'code checkout  done automatically via SCM'
            }
        }
        stage('Security Scans (FS & Dependencies)') {
            parallel {
                // Parallel execution: OWASP aur Trivy FS saath mein chalenge time bachane ke liye
                stage('OWASP Dependency Check') {
                    steps {
                        echo "Scanning Dependencies..."
                        dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
                    }
                }
                
                stage('Trivy FS Scan') {
                    steps {
                        echo "Trivy Scanning File System..."
                        // Ye poore folder (.) ko scan karega bugs/secrets ke liye
                        sh "trivy fs . > trivy-fs-report.txt"
                    }
                }
            }
        }
        // stage('OWASP security scan'){
        //     steps{
        //         echo "scanning for vulnerabilities...."
        //         // DP-Check oo nam he tool me dia tha.....
        //         // pehili bar me chalne be 22-25 mins lega because (database update hone me...)
        //         dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'
        //     }
        // }
        stage('sonarqube analysis'){
            steps{
                withSonarQubeEnv('sonar-server'){
                    // code ko scan karke report server par send karta he....
                    sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=pytho-project -Dsonar.source=."
                }
            }
        }

        stage('build docker image') {
            steps {
                script {
                    echo 'build docker image ......'

                    // image ke name format hoga usename/image:tag
                    sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."

                    // ek latest tag bhi banata he taki deploy karna asan ho jaee..
                    sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest ."
                }
            }
        }
        stage('trivy image scan'){
            steps{
                echo "scanning docker image for vulnerabilities"
                sh "trivy image ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} > trivy-image-report.txt"
            }
        }

        stage('push to dockerhub') {
            steps {
                script {
                    echo 'pushing to dockerhub.....'

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            passwordVariable: 'DOCKER_PASS',
                            usernameVariable: 'DOCKER_USER'
                        )
                    ]) {

                        // login command
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"

                        // push command
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('deploy the container') {
            steps {
                script {
                    echo "deploying application.."

                    sh "docker rm -f pytholab || true"

                    sh "docker run -d --name pytholab -p 80:80 ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }
    }
    // Report generate kar lo
    post{
        always{
            // OWASP ke report graph ke rup me dikhana....
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            // Trivy reports ko archive karna taaki download kar sakein
            archiveArtifacts artifacts: 'trivy-fs-report.txt, trivy-image-report.txt', allowEmptyArchive: true
        }
        failure{
            echo "build failed sending emial to developer"
            mail to: 'vaibhavkundgar181@gmail.com', subject: "build failed: ${JOB_NAME}", body: "check jenkins logs"
        }
    }
}