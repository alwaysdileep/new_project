pipeline {
    agent any

    environment {
        // Set your Docker Hub username and password as environment variables
        DOCKER_USERNAME = credentials('docker-username')
        DOCKER_PASSWORD = credentials('docker-password')
        IMAGE_NAME = 'my-app'
        DOCKER_REGISTRY = 'docker.io'  // You can change this if using a different registry
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout code from Git repository
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build the Docker image
                    sh 'docker build -t $DOCKER_USERNAME/$IMAGE_NAME .'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                script {
                    // Login to Docker Hub
                    withCredentials([usernamePassword(credentialsId: 'docker-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "echo $DOCKER_PASSWORD | docker login -u $DOCKER_USERNAME --password-stdin"
                    }
                }
            }
        }

        stage('Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Push the Docker image to Docker Hub
                    sh 'docker push $DOCKER_USERNAME/$IMAGE_NAME'
                }
            }
        }

        stage('Deploy to Remote Server') {
            steps {
                script {
                    // SSH into your remote server and pull and run the Docker image
                    // Make sure you have set up SSH keys for Jenkins or use a credential manager for SSH
                    sshagent(['ssh-credentials-id']) {
                        sh """
                        ssh user@your-remote-server << EOF
                        docker pull $DOCKER_USERNAME/$IMAGE_NAME
                        docker stop my-app || true
                        docker rm my-app || true
                        docker run -d -p 3000:3000 --name my-app $DOCKER_USERNAME/$IMAGE_NAME
                        EOF
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            // Clean up Docker resources
            sh 'docker system prune -f'
        }
    }
}

