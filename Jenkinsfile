pipeline {
    agent any

    environment {
        IMAGE_NAME = "koushiksiripuram/orangehrm-app"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }
    
    tools {
        dockerTool 'my-docker'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/koushiksiripuram/orange_hrm.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                // Fixed path from ./app to . because Dockerfile is in the root directory
                sh 'docker build -t $IMAGE_NAME:$IMAGE_TAG .'
                sh 'docker tag $IMAGE_NAME:$IMAGE_TAG $IMAGE_NAME:latest' 
            }
        }

        stage('Docker Hub Login') {
            steps {
                // Kept secure using --password-stdin to avoid exposing your password in logs
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_NAME:$IMAGE_TAG'
                sh 'docker push $IMAGE_NAME:latest'
            }
        }

        stage('Redeploy Containers') {
            steps {
                // Added --remove-orphans to keep the server clean during recreation
                sh 'docker compose down --remove-orphans'
                sh 'docker compose pull'
                sh 'docker compose up -d'
            }
        }
    }
}
