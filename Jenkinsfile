pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'dockerhub-credentialss'
        KUBECONFIG = '/var/jenkins_home/.kube/config'
    }
    stages {
        stage('Checkout Code') {
            steps {
                git 'https://github.com/HoussemHaji/devops-practice3'
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    docker.withRegistry('https://registry.hub.docker.com', DOCKER_CREDENTIALS_ID) {
                        def app = docker.build("lh0ss/hello-kubernetes-app:${env.BUILD_NUMBER}")
                        app.push()
                    }
                }
            }
        }
        stage('Deploy to Kubernetes') {
            steps {
                sh 'kubectl apply -f kubernetes/deployment.yaml'
            }
        }
        stage('Configure with Ansible') {
            steps {
                sh 'ansible-playbook playbook.yml'
            }
        }
    }
    post {
        always {
            echo 'Pipeline finished'
        }
    }
}
