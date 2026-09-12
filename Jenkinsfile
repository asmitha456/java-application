pipeline {
    agent any
    
    environment {
        DOCKER_REGISTRY = 'your-registry.com'
        BACKEND_IMAGE = "${DOCKER_REGISTRY}/hotel-backend:${BUILD_NUMBER}"
        FRONTEND_IMAGE = "${DOCKER_REGISTRY}/hotel-frontend:${BUILD_NUMBER}"
        K8S_NAMESPACE = 'hotel-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Unit Tests') {
            parallel {
                stage('Backend Tests') {
                    steps {
                        dir('backend') {
                            sh 'mvn test'
                        }
                    }
                }
                stage('Frontend Tests') {
                    steps {
                        dir('frontend') {
                            sh 'npm test'
                        }
                    }
                }
            }
        }
        
        stage('Build & Package') {
            parallel {
                stage('Build Backend') {
                    steps {
                        dir('backend') {
                            sh 'mvn clean package -DskipTests'
                        }
                    }
                }
                stage('Build Frontend') {
                    steps {
                        dir('frontend') {
                            sh 'npm install && npm run build'
                        }
                    }
                }
            }
        }
        
        stage('Docker Build & Push') {
            parallel {
                stage('Backend Docker') {
                    steps {
                        dir('backend') {
                            sh "docker build -t ${BACKEND_IMAGE} ."
                            sh "docker push ${BACKEND_IMAGE}"
                        }
                    }
                }
                stage('Frontend Docker') {
                    steps {
                        dir('frontend') {
                            sh "docker build -t ${FRONTEND_IMAGE} ."
                            sh "docker push ${FRONTEND_IMAGE}"
                        }
                    }
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                dir('kubernetes') {
                    sh """
                        kubectl set image deployment/hotel-backend \
                            hotel-backend=${BACKEND_IMAGE} \
                            -n ${K8S_NAMESPACE}
                        kubectl set image deployment/hotel-frontend \
                            hotel-frontend=${FRONTEND_IMAGE} \
                            -n ${K8S_NAMESPACE}
                    """
                }
            }
        }
        
        stage('Smoke Tests') {
            steps {
                sh """
                    sleep 30
                    curl -f http://hotel-app.com/api/hotels || exit 1
                    curl -f http://hotel-app.com || exit 1
                """
            }
        }
    }
    
    post {
        success {
            echo 'Deployment successful!'
            slackSend(
                color: 'good',
                message: "Deployment successful: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
        failure {
            echo 'Deployment failed!'
            slackSend(
                color: 'danger',
                message: "Deployment failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}"
            )
        }
    }
}
