pipeline {
    agent {
        label 'jenkins-linux-agent-TLS'
    }

    environment {
        // Docker Hub configuration
        DOCKERHUB_REPOSITORY = 'amundead/jenkins-inbound-agent-docker'  // Docker Hub repository
        IMAGE_NAME_DOCKERHUB = "${DOCKERHUB_REPOSITORY}"  // Full image name for Docker Hub
        
        // GitHub Packages configuration
        GITHUB_REPOSITORY = 'amundead/jenkins-inbound-agent-docker'  // GitHub repository (user/repo format)
        IMAGE_NAME_GITHUB = "ghcr.io/${GITHUB_REPOSITORY}"  // Full image name for GitHub Packages
        
        // Common tag
        TAG = 'latest'  // Tag for the Docker image
    }

    stages {
        stage('Checkout') {
            steps {
                // Checkout the source code from your repository using credentials securely
                git branch: 'main', url: "https://github.com/amundead/jenkins-inbound-agent-docker.git", credentialsId: 'github-credentials-id'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Build Docker image using docker.build with --no-cache option
                    docker.build("${IMAGE_NAME_DOCKERHUB}:${TAG}", "--no-cache .")
                }
            }
        }

        stage('Tag and Push Docker Image to Docker Hub') {
            steps {
                script {
                    // Use docker.withRegistry for secure login and push to Docker Hub
                    docker.withRegistry('https://index.docker.io/v1/', 'dockerhub-credentials-id') {
                        docker.image("${IMAGE_NAME_DOCKERHUB}:${TAG}").push()
                    }
                }
            }
        }

        stage('Tag and Push Docker Image to GitHub Packages') {
            steps {
                script {
                    // Tag the built Docker image for GitHub Packages
                    sh "docker tag ${IMAGE_NAME_DOCKERHUB}:${TAG} ${IMAGE_NAME_GITHUB}:${TAG}"

                    // Authenticate to GitHub Packages and push
                    docker.withRegistry('https://ghcr.io', 'github-credentials-amir') {
                        docker.image("${IMAGE_NAME_GITHUB}:${TAG}").push()
                    }
                }
            }
        }

        stage('Clean up') {
            steps {
                script {
                    // Remove unused Docker images to free up space
                    sh "docker rmi ${IMAGE_NAME_DOCKERHUB}:${TAG}"
                    sh "docker rmi ${IMAGE_NAME_GITHUB}:${TAG}"
                }
            }
        }
    }

    post {
        always {
            // Clean up workspace after the pipeline
            cleanWs()
        }
    }
}