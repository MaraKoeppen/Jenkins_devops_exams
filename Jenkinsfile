pipeline {
  agent any

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build Docker Image') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'DOCKER_HUB_PASS_U',
                         usernameVariable: 'USER',
                         passwordVariable: 'PASS')]) {

          sh '''
          docker login -u $USER -p $PASS
          docker build -t $USER/devops-exam-app:latest .
          docker push $USER/devops-exam-app:latest
          '''
        }
      }
    }

    stage('Deploy DEV') {
      steps {
        withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
          sh '''
          helm upgrade --install app charts -n dev
          '''
        }
      }
    }

  }
}
