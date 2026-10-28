pipeline {
    agent any

    environment {
        IMAGE_NAME = 'prithika2007/Kubernets'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Fetching source code from GitHub'
                checkout scm
            }
        }

        stage('Validate Project') {
            steps {
                echo 'Checking project files'

                bat '''
                    if not exist index.html exit /b 1
                    if not exist Dockerfile exit /b 1
                    if not exist deployment.yaml exit /b 1
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                echo 'Building Docker image'

                bat 'docker build -t %IMAGE_NAME%:%BUILD_NUMBER% .'

                bat 'docker tag %IMAGE_NAME%:%BUILD_NUMBER% %IMAGE_NAME%:latest'
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-new',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    bat 'echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat 'docker push %IMAGE_NAME%:%BUILD_NUMBER%'

                bat 'docker push %IMAGE_NAME%:latest'
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                echo 'Deploying application to Kubernetes'

                bat 'kubectl apply -f deployment.yaml'

                bat 'kubectl set image deployment/jenkins-demo web=%IMAGE_NAME%:%BUILD_NUMBER%'

                bat 'kubectl rollout status deployment/jenkins-demo --timeout=180s'
            }
        }

        stage('Verify Deployment') {
            steps {
                bat 'kubectl get deployments'

                bat 'kubectl get pods'

                bat 'kubectl get services'
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'Pipeline failed. Check the Jenkins console output.'
        }
    }
}