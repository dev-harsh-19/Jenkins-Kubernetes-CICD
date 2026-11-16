pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'devharsh1912/jenkins-kubernetes-cicd'
        DOCKER_CLI = 'C:\\Users\\DEV HARSH\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        KUBECTL = 'C:\\Users\\DEV HARSH\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\kubectl.exe'
        KUBECONFIG = 'C:\\Users\\DEV HARSH\\.kube\\config'
    }

    stages {

        stage('Checkout') {
            steps {
                echo 'Checking out source code'
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                bat '''
                "%DOCKER_CLI%" build -t %DOCKER_IMAGE%:%BUILD_NUMBER% .
                "%DOCKER_CLI%" tag %DOCKER_IMAGE%:%BUILD_NUMBER% %DOCKER_IMAGE%:latest
                '''
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DOCKERHUB_USER',
                        passwordVariable: 'DOCKERHUB_PASSWORD'
                    )
                ]) {
                    bat '''
                    echo %DOCKERHUB_PASSWORD% | "%DOCKER_CLI%" login -u %DOCKERHUB_USER% --password-stdin
                    '''
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                bat '''
                "%DOCKER_CLI%" push %DOCKER_IMAGE%:%BUILD_NUMBER%
                "%DOCKER_CLI%" push %DOCKER_IMAGE%:latest
                '''
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                bat '''
                set "KUBECONFIG=%KUBECONFIG%"

                "%KUBECTL%" apply -f kubernetes-config/service.yaml

                "%KUBECTL%" set image deployment/website-deployment website-container=%DOCKER_IMAGE%:%BUILD_NUMBER%

                "%KUBECTL%" rollout status deployment/website-deployment --timeout=180s
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                bat '''
                set "KUBECONFIG=%KUBECONFIG%"

                "%KUBECTL%" get deployments
                "%KUBECTL%" get pods
                "%KUBECTL%" get services
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }
    }
}