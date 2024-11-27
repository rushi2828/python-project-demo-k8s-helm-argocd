pipeline {
    agent any

    environment {
        DOCKER_REGISTRY = 'rushi2323'
        IMAGE_NAME = 'python-project-demo-k8s'
        IMAGE_TAG = 'latest'
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG .'
                }
            }
        }

        stage('Test Docker Image') {
            steps {
                script {
                    sh 'docker run --rm -p 5000:5000 -d $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
                    sh 'sleep 5' // Give the container time to start
                    sh 'curl -f http://localhost:5000'
                }
            }
        }

        stage('Push to Docker Registry') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'DOCKER_REGISTRY_CREDENTIALS', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                        sh 'docker login -u $DOCKER_USER -p $DOCKER_PASS'
                        sh 'docker push $DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG'
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    kubectl set image deployment/$IMAGE_NAME $IMAGE_NAME=$DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG -n default || kubectl create deployment $IMAGE_NAME --image=$DOCKER_REGISTRY/$IMAGE_NAME:$IMAGE_TAG -n default
                    kubectl expose deployment $IMAGE_NAME --type=LoadBalancer --port=80 --target-port=5000 -n default || true
                    '''
                }
            }
        }
    }

    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo 'Pipeline succeeded.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
