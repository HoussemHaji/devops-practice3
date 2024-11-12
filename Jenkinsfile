pipeline {
    agent any
    
    environment {
        DOCKER_IMAGE = 'lh0ss/hello-kubernetes-app'
        DOCKER_TAG = "${BUILD_NUMBER}"
        KUBECONFIG = credentials('kubeconfig')
        GIT_REPO = 'https://github.com/HoussemHaji/devops-practice3'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'master', url: "${GIT_REPO}"
            }
        }
        
        stage('Build Docker Image') {
            steps {
                script {
                    sh """
                        docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .
                        docker tag ${DOCKER_IMAGE}:${DOCKER_TAG} ${DOCKER_IMAGE}:latest
                    """
                }
            }
        }
        
        stage('Push to DockerHub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKERHUB_USER', passwordVariable: 'DOCKERHUB_PASS')]) {
                    sh """
                        echo "${DOCKERHUB_PASS}" | docker login -u "${DOCKERHUB_USER}" --password-stdin
                        docker push ${DOCKER_IMAGE}:${DOCKER_TAG}
                        docker push ${DOCKER_IMAGE}:latest
                    """
        }
    }
        }
        
        stage('Update Kubernetes Manifests') {
            steps {
                script {
                    // Update deployment.yaml with new image tag
                    sh """
                        sed -i 's|image: ${DOCKER_IMAGE}:[^ ]*|image: ${DOCKER_IMAGE}:${DOCKER_TAG}|' kubernetes/deployment.yaml
                    """
                }
            }
        }
        
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh """
                        export KUBECONFIG=\${KUBECONFIG}
                        kubectl apply -f kubernetes/deployment.yaml
                        kubectl apply -f kubernetes/service.yaml
                    """
                }
            }
        }
        
        stage('Verify Deployment') {
            steps {
                script {
                    sh """
                        export KUBECONFIG=\${KUBECONFIG}
                        kubectl rollout status deployment/hello-kubernetes-app
                        kubectl get services hello-kubernetes-service
                    """
                }
            }
        }
        
        stage('Run Ansible Playbook') {
            steps {
                ansiblePlaybook(
                    playbook: 'ansible/configure-nagios.yml',
                    inventory: 'ansible/inventory.ini'
                )
            }
        }
        
        stage('Health Check') {
            steps {
                script {
                    sh """
                        # Get NodePort
                        export NODE_PORT=\$(kubectl get svc your-app-service -o jsonpath='{.spec.ports[0].nodePort}')
                        # Health check using curl
                        curl -f http://localhost:\${NODE_PORT}/health || exit 1
                    """
                }
            }
        }
    }
 
}