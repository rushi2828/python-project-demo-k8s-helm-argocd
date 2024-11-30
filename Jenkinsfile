pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'your-dockerhub-username'
        IMAGE_NAME = 'flask-app'
        IMAGE_TAG = 'latest'
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
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh "docker login -u ${DOCKER_USER} -p ${DOCKER_PASS}"
                        sh "docker push ${DOCKER_REGISTRY}/${IMAGE_NAME}:${IMAGE_TAG}"
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

