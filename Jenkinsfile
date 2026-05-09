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
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('checkout code') {
            steps {
                echo 'code checkout automatically done via scm'
            }
        }

        stage('Build docker image') {
            steps {
                script {
                    echo 'build docker image ...'

                    // Build image with build number tag
                    sh "docker build -t ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ."

                    // Tag same image as latest
                    sh "docker tag ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG} ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }

        stage('push to dockerhub') {
            steps {
                script {
                    echo 'pushing to dockerhub.......'

                    withCredentials([
                        usernamePassword(
                            credentialsId: 'docker-hub-creds',
                            passwordVariable: 'DOCKER_PASS',
                            usernameVariable: 'DOCKER_USER'
                        )
                    ]) {

                        // Docker login
                        sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"

                        // Push both tags
                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:${IMAGE_TAG}"

                        sh "docker push ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                    }
                }
            }
        }

        stage('deploy the container') {
            steps {
                script {
                    echo 'deploying the container...'

                    // Remove old container
                    sh "docker rm -f pytholab || true"

                    // Run latest container
                    sh "docker run -d --name pytholab -p 80:80 ${DOCKERHUB_USERNAME}/${IMAGE_NAME}:latest"
                }
            }
        }
    }
}