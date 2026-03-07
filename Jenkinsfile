pipeline {
    agent any

    environment {
        DOCKERHUB_REPO = "aarushisuba/webserver"
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_REGION = "ap-south-1"
        EKS_CLUSTER = "my-eks-cluster"
    }

    stages {



        stage("Build"){
            steps {
                script {
                    sh "docker build -t webserver ."
                }
            }
        }

        stage("Login to DockerHub") {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-credentials',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')]) {
                    sh "echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin"
                }
            }
        }

        stage("Tag & Push Docker Image") {
            steps {
                sh """
                docker tag webserver $DOCKERHUB_REPO:$IMAGE_TAG
                docker push $DOCKERHUB_REPO:$IMAGE_TAG
                """
            }
        }

        
    }
}
