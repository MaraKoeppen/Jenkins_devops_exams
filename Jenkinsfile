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
            docker build -t "$USER/cast_service:latest" ./cast-service

            # Push
            docker push "$USER/movie_service:latest"
            docker push "$USER/cast_service:latest"
          '''
        }
      }
    }

    stage('Deploy (dev/qa/staging + manual prod)') {
      steps {
        script {

          // Multibranch setzt BRANCH_NAME; fallback für andere Job-Typen
          def branch = env.BRANCH_NAME ?: sh(script: "git rev-parse --abbrev-ref HEAD", returnStdout: true).trim()
          echo "Branch detected: ${branch}"

          // Auto-Deploy: immer dev, zusätzlich qa/staging nur auf master
          def autoNamespaces = (branch == 'master') ? ['dev', 'qa', 'staging'] : ['dev']

          withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {

            // Auto deploys
            for (ns in autoNamespaces) {
              sh "helm upgrade --install app ./charts -n ${ns}"
            }

            // PROD: nur manuell + nur master
            if (branch == 'master') {
              input message: "Deploy to PROD (namespace: prod) from master?", ok: "Deploy PROD"
              sh "helm upgrade --install app ./charts -n prod"
            } else {
              echo "Skipping PROD deploy (branch=${branch})."
            }

          }
        }
      }
    }

  }
}