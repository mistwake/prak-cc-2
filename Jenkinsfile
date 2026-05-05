pipeline {
    agent any

    environment {
        DOCKERHUB_USERNAME = 'mistwake'
        DOCKERHUB_CREDENTIALS = 'dockerhub-creds'
        KUBECONFIG_CREDENTIALS = 'kubeconfig-creds'
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Push Backend') {
            steps {
                script {
                    dir('backend') {
                        // login ke docker hub
                        withCredentials([usernamePassword(credentialsId: "${DOCKERHUB_CREDENTIALS}", usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                            bat 'echo %DOCKER_PASS%| docker login -u %DOCKER_USER% --password-stdin'
                        }
                        
                        // build image
                        bat "docker build -t ${DOCKERHUB_USERNAME}/backend-inventory:latest ."
                        
                        // push image
                        bat "docker push ${DOCKERHUB_USERNAME}/backend-inventory:latest"
                    }
                }
            }
        }

        stage('Build & Push Frontend') {
            steps {
                script {
                    dir('frontend') {
                        // build image
                        bat "docker build -t ${DOCKERHUB_USERNAME}/frontend-inventory:latest ."
                        
                        // push image
                        bat "docker push ${DOCKERHUB_USERNAME}/frontend-inventory:latest"
                    }
                }
            }
        }

        stage('Deploy to AKS') {
            steps {
                script {
                    // deploy menggunakan kubeconfig dari credentials jenkins
                    withKubeConfig([credentialsId: "${KUBECONFIG_CREDENTIALS}"]) {
                        // apply file manifest secara langsung
                        bat "kubectl apply -f deployment.yaml -f ingress.yaml"
                    }
                }
            }
        }
    }
    
    post {
        always {
            // logout docker setelah selesai
            bat "docker logout"
        }
    }
}

//punten ie mah ngetes webhook