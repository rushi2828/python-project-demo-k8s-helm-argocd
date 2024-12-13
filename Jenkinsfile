pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'rushi2323'
        IMAGE_NAME = 'python-project-demo-k8s'
        IMAGE_TAG = '1.0.0'
        DOCKER_REGISTRY_CREDENTIALS = 
    }

    stages {
        stage('Checkout Code') {
            steps {
                // Checkout source code from the Git repository
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Test Application') {
            steps {
                script {
                    // Run the Docker container and perform simple tests
                    sh "docker run --rm -d -p 5000:5000 ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                    sh "sleep 5" // Wait for the app to start
                    sh "curl -f http://localhost:5000"
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    try {
                        withCredentials([usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                            sh "sleep 10" // Wait for the app login
                            sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
                        }
                    } catch (Exception e) {
                        echo "Docker push failed: ${e}"

                        // Add Cleanup Step Here
                        echo "Performing cleanup of local Docker images and containers..."
                        sh """
                        # Remove any running containers using the image
                        docker ps -a --filter "ancestor=${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}" -q | xargs --no-run-if-empty docker rm -f

                        # Remove the local image
                        docker rmi -f ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG} || true
                        """
                        
                        error("Stopping the pipeline due to Docker push failure.")
                    }
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}

