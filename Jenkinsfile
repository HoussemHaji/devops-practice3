pipeline {
    agent any
    environment {
        DOCKER_HUB_REPO = 'lh0ss/hello-kubernetes-app'
        DOCKER_CREDENTIALS = '8b7fad91-97b1-4551-a552-b1328a35911c'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'master', url: 'https://github.com/HoussemHaji/devops-practice3'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.build("${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}")
                }
            }
        }
        stage('Push to DockerHub') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS) {
                        docker.image("${DOCKER_HUB_REPO}:${env.BUILD_NUMBER}").push()
                    }
                }
            }
        }
        
    }
    
}