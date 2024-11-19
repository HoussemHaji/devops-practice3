pipeline {
  environment {
    dockerimagename = "lh0ss/hello-kubernetes-app"
    dockerImage = ""
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
        git 'https://github.com/HoussemHaji/devops-practice3'
      }
    }
    stage('Build image') {
      steps{
        script {
          dockerImage = docker.build dockerimagename
        }
      }
    }
    stage('Pushing Image') {
      environment {
          registryCredential = 'dockerhub-credentials'
           }
      steps{
        script {
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }
    stage('Deploy to Kubernetes') {
        steps {
            sh '''
            kubectl apply -f deployment.yaml
            kubectl apply -f service.yaml
            '''
        }
}
  }
}