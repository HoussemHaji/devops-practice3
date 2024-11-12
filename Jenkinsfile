pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'lh0ss/hello-kubernetes-app'
        DOCKER_TAG = "latest"
        GIT_REPO = 'https://github.com/votre-repo/votre-app.git'
        KUBE_CONFIG = credentials('kubeconfig')
        APP_NAMESPACE = 'webapp'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: "${GIT_REPO}"
            }
        }
        
        stage('Setup Kubernetes Namespace') {
            steps {
                script {
                    sh """
                        kubectl create namespace ${APP_NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                    """
                }
            }
        }
        
        
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} ."
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                sh "docker push ${DOCKER_IMAGE}:latest"
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    ansiblePlaybook(
                        playbook: 'k8s-deploy-nodeport.yml',
                        inventory: 'inventory.ini',
                        extras: "-e docker_tag=${DOCKER_TAG} -e namespace=${APP_NAMESPACE}"
                    )
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sh """
                        kubectl -n ${APP_NAMESPACE} get pods -l app=webapp
                        kubectl -n ${APP_NAMESPACE} get svc webapp-service
                        echo "Waiting for pods to be ready..."
                        kubectl -n ${APP_NAMESPACE} wait --for=condition=ready pod -l app=webapp --timeout=300s
                        
                        # Obtenir le NodePort
                        NODE_PORT=\$(kubectl -n ${APP_NAMESPACE} get svc webapp-service -o jsonpath='{.spec.ports[0].nodePort}')
                        echo "Application accessible at: http://localhost:\${NODE_PORT}"
                    """
                }
            }
        }
    }
    
    post {
        always {
            sh 'docker logout'
        }
        success {
            echo 'Deployment successful!'
        }
    }
}