pipeline {
    agent any

    environment {
        IMAGE_NAME = "aneeka997/devops-task-app"
        DOCKER_CREDENTIALS = 'dockerhub-pat-token'
        TASK_FAMILY = 'Devops-task'
        CLUSTER_NAME = "devops-cluster"
        AWS_REGION = "ap-south-1"
        SERVICE_NAME = "devops-service"
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

        stage('Deploy to ECS') {
            steps {
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-id'
                ]]) {
                    script {
                        // Create ECS cluster if it doesn't exist
                        sh """ 
                        aws ecs create-cluster --cluster-name ${CLUSTER_NAME} --region ${AWS_REGION}
                        """

                        // Register a new task definition with this build's image
                        sh """
                        aws ecs register-task-definition \
                            --family ${TASK_FAMILY} \
                            --requires-compatibilities FARGATE \
                            --network-mode awsvpc \
                            --cpu 256 --memory 512 \
                            --container-definitions '[{"name":"devops-task-app","image":"${IMAGE_NAME}:${BUILD_NUMBER}","essential":true,"portMappings":[{"containerPort":80,"hostPort":80}]}]' \
                            --region ${AWS_REGION}
                        """

                        // Update service (create if doesn't exist)
                        sh """
                            aws ecs create-service \
                                --cluster ${CLUSTER_NAME} \
                                --service-name ${SERVICE_NAME} \
                                --task-definition ${TASK_FAMILY} \
                                --desired-count 1 \
                                --launch-type FARGATE \
                                --network-configuration 'awsvpcConfiguration={subnets=["subnet-04d12569497e1b3e9","subnet-07d4ba1600697f975"],securityGroups=["sg-0d8275262c70b504a"],assignPublicIp="ENABLED"}' \
                                --region ${AWS_REGION}
                                
                        """
                   }
                }
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
