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

    stage('Deploy (dev/qa/staging + manual prod)') {
      steps {
        script {
          // Robust branch detection for "Pipeline script from SCM" (often detached HEAD)
          def branch = env.GIT_BRANCH ?: env.BRANCH_NAME ?: "unknown"
          echo "Branch detected: ${branch}"

          // Auto-Deploy: always dev; additionally qa/staging only on master
          def isMaster = branch.contains('master')
          def autoNamespaces = isMaster ? ['dev', 'qa', 'staging'] : ['dev']

          withCredentials([file(credentialsId: 'config', variable: 'KUBECONFIG')]) {

            // Auto deploys
            for (ns in autoNamespaces) {
              sh "helm upgrade --install app ./charts -n ${ns}"
            }

            // PROD: manual + only from master
            if (isMaster) {
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