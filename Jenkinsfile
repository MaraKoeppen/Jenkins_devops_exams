pipeline {
  agent any

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build & Push Images') {
      steps {
        withCredentials([usernamePassword(
          credentialsId: 'DOCKER_HUB_PASS_U',
          usernameVariable: 'USER',
          passwordVariable: 'PASS'
        )]) {
          sh '''
            set -e
            echo "$PASS" | docker login -u "$USER" --password-stdin

            # Build
            docker build -t "$USER/movie_service:latest" ./movie-service
            docker build -t "$USER/cast_service:latest"  ./cast-service

            # Push
            docker push "$USER/movie_service:latest"
            docker push "$USER/cast_service:latest"
          '''
        }
      }
    }

    stage('Deploy DEV') {
      steps {
        withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {
          sh '''
            set -e
            helm upgrade --install app ./charts -n dev
          '''
        }
      }
    }

  }
}