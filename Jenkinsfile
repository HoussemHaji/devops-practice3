pipeline {
  agent {
    kubernetes {
      label 'my-kubernetes-agent'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    jenkins/label: my-kubernetes-agent
spec:
  containers:
  - name: docker
    image: docker:20.10.24
    command:
    - cat
    tty: true
    volumeMounts:
    - name: docker-sock
      mountPath: /var/run/docker.sock
  volumes:
  - name: docker-sock
    hostPath:
      path: /var/run/docker.sock
"""
    }
  }

  environment {
    DOCKERHUB_CREDENTIALS = credentials('dockerhub-credentials') // Replace with your DockerHub credentials ID
    KUBECONFIG = credentials('kubernetes-config') // Replace with your kubeconfig credentials ID
  }

  stages {
    stage('Clone repository') {
      steps {
        git 'https://github.com/HoussemHaji/devops-practice3'
      }
    }

    stage('Build and push Docker image') {
      steps {
        container('docker') {
          script {
            sh 'docker login -u $DOCKERHUB_CREDENTIALS_USR -p $DOCKERHUB_CREDENTIALS_PSW'
            sh 'docker build -t lh0ss/hello-kubernetes-app:latest .'
            sh 'docker push lh0ss/hello-kubernetes-app:latest'
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        container('docker') {
          script {
            // Set up kubectl using the kubeconfig credentials
            writeFile file: '/root/.kube/config', text: KUBECONFIG
            sh 'kubectl apply -f kubernetes/deployment.yaml'
          }
        }
      }
    }
  }
}
