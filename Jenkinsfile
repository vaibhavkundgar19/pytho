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

// pipeline {
//     agent any

//     environment {
//         DOCKERHUB_USERNAME = 'kundgar19'
//         IMAGE_NAME = 'pytho'
//         IMAGE_TAG = "${BUILD_NUMBER}"
//         //sonarqube scanner tool ka path lad karr rahe hai
//         SCANNER_HOME = tool 'sonar-scanner'
//     }

//     stages {

//         stage('checkout code') {
//             steps {
//                 echo 'code checkout automatically done via scm'
//             }
//         }
//         stage('owasp security scan'){
//             steps{
//                 echo "Scanning for vulnerabilities...."
//                 // DP-Check vo naam hai jo humne tool mein diya tha...
//                 //phele baar chanle mein 10 to 2 mins lega(database update hone mein)
//                 dependencyCheck additionalArguments: '--scan ./ --disableYarnAudit --disableNodeAudit', odcInstallation: 'DP-Check'

//             }
//         }
//         stage('Sonarqube analysis'){
//             steps{
//                 withSonarQubeEnv('sonar-server'){
//                     //code ko scan karke report server par bhejo
//                 sh "${SCANNER_HOME}/bin/sonar-scanner -Dsonar.projectKey=pytho-project -Dsonar.source=."
//                 }
//             }
//         }

//         stage('Build docker image') {
//             steps {
//                 script {
//                     echo 'build docker image ...'

//                     // Build image with build number tag
//                     sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."

//                     // Tag same image as latest
//                     sh "docker tag ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
//                 }
//             }
//         }

//         stage('push to dockerhub') {
//             steps {
//                 script {
//                     echo 'pushing to dockerhub.......'

//                     withCredentials([
//                         usernamePassword(
//                             credentialsId: 'docker-hub-creds',
//                             passwordVariable: 'DOCKER_PASS',
//                             usernameVariable: 'DOCKER_USER'
//                         )
//                     ]) {

//                         // Docker login
//                         sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"

//                         // Push both tags
//                         sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"

//                         sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
//                     }
//                 }
//             }
//         }

//         stage('deploy the container') {
//             steps {
//                 script {
//                     echo 'deploying the container...'

//                     // Remove old container
//                     sh "docker rm -f pytholab || true"

//                     // Run latest container
//                     sh "docker run -d --name pytholab -p 80:80 ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
//                 }
//             }
//         }
//     }
//     //Report generate karlo
//     post{
//         always{
//             //owasp ki report graph ke roop mein dikhana...
//             dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
//         }
//     }
// }

pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'kundgar19'
        IMAGE_NAME = 'pytho'
        IMAGE_TAG = "${BUILD_NUMBER}"

        // SonarQube scanner tool name from Jenkins
        SCANNER_HOME = tool 'sonar-scanner'
    }

    stages {

        stage('checkout code') {
            steps {
                echo 'Code checkout automatically done via SCM'
            }
        }

        stage('owasp security scan') {
            steps {
                script {

                    echo "Scanning for vulnerabilities..."

                    dependencyCheck(
                        odcInstallation: 'DP-Check',
                        additionalArguments: '''
                            --scan .
                            --format XML
                            --out .
                            --disableYarnAudit
                            --disableNodeAudit
                        '''
                    )

                    // Debugging logs
                    sh 'pwd'
                    sh 'ls -la'
                    sh 'find . -name "*.xml"'
                }
            }
        }

        stage('Sonarqube analysis') {
            steps {
                script {

                    withSonarQubeEnv('sonar-server') {

                        sh """
                        ${SCANNER_HOME}/bin/sonar-scanner \
                        -Dsonar.projectKey=pytho-project \
                        -Dsonar.sources=.
                        """
                    }
                }
            }
        }

        stage('Build docker image') {
            steps {
                script {

                    echo 'Building Docker image...'

                    // Build image using build number
                    sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."

                    // Tag image as latest
                    sh "docker tag ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('push to dockerhub') {
            steps {
                script {

                    echo 'Pushing image to DockerHub...'

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            usernameVariable: 'DOCKER_USER',
                            passwordVariable: 'DOCKER_PASS'
                        )
                    ]) {

                        // Docker login
                        sh "echo \$DOCKER_PASS | docker login -u \$DOCKER_USER --password-stdin"

                        // Push build image
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"

                        // Push latest image
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('deploy the container') {
            steps {
                script {

                    echo 'Deploying container...'

                    // Remove old container if exists
                    sh "docker rm -f pytholab || true"

                    // Run latest container
                    // Change internal port if your app runs on another port
                    sh "docker run -d --name pytholab -p 80:80 ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }
    }

    post {

        always {

            echo 'Publishing OWASP Dependency Check report...'

            dependencyCheckPublisher(
                pattern: 'dependency-check-report.xml'
            )
        }

        success {
            echo 'Pipeline executed successfully!'
        }

        failure {
            echo 'Pipeline failed!'
        }
    }
}