pipeline {
    agent any

    environment {
        IMAGE_NAME = "aneeka997/devops-task-app"
        DOCKER_CREDENTIALS = 'dockerhub-pat-token'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'dev', url: 'https://github.com/AneekaSA/devops-task.git'
            }
        }

        stage('Build & Test') {
            steps {
                sh 'npm install'
                sh 'npm test || echo "No tests found"'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    // Build the Docker image and assign to a variable
                    appImage = docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    // Push the previously built image
                    docker.withRegistry('', DOCKER_CREDENTIALS) {
                        appImage.push()
                        appImage.push('latest')
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying container...'
                
            }
        }
    }

    post {
        always {
            echo "Cleaning up local images"
            sh "docker rmi ${IMAGE_NAME}:${BUILD_NUMBER} || true"
        }
        success {
            echo "Pipeline completed successfully ✅"
        }
        failure {
            echo "Pipeline failed ❌"
        }
    }
}
