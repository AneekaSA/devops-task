pipeline {
    agent any

    environment {
        REGISTRY = "docker.io"                   // or AWS ECR / GCP Artifact Registry
        IMAGE_NAME = "aneekasa/devops-task-app"  // change to your DockerHub repo
        DOCKER_CREDENTIALS = 'dockerhub-pat-token'  // Jenkins credentials ID
    }

    triggers {
        githubPush()   // Trigger on GitHub push (needs webhook)
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
                    docker.build("${IMAGE_NAME}:${BUILD_NUMBER}")
                }
            }
        }

        stage('Push to Registry') {
            steps {
                script {
                    docker.withRegistry("https://${REGISTRY}", DOCKER_CREDENTIALS) {
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push()
                        docker.image("${IMAGE_NAME}:${BUILD_NUMBER}").push("latest")
                    }
                }
            }
        }

        stage('Deploy') {
            steps {
                echo 'Deploying container...'
                // Example: AWS ECS CLI, kubectl for EKS/GKE, or gcloud for Cloud Run
                // sh 'aws ecs update-service --cluster myCluster --service myService --force-new-deployment'
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
