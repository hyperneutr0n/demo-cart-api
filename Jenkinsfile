def deployTo(String ns) {
  // Entri prod dipakai kalau prod digabung ke deploy otomatis (continuous deployment penuh)
  def ports = [dev: '30081', qa: '30082', prod: '30080']
  withEnv(["NS=${ns}", "NODE_PORT=${ports[ns]}"]) {
    withCredentials([file(credentialsId: 'k3s-kubeconfig', variable: 'KUBECONFIG')]) {
      sh '''
        kubectl create namespace "$NS" --dry-run=client -o yaml | kubectl apply -f -
        sed -e "s|IMAGE_PLACEHOLDER|$IMAGE:$TAG|" -e "s|NODEPORT_PLACEHOLDER|$NODE_PORT|" k8s/app.yaml | kubectl apply -n "$NS" -f -
        kubectl rollout status deployment/$APP -n "$NS" --timeout=120s
      '''
    }
  }
}

pipeline {
  agent any
  environment {
    APP   = 'express-demo'
    IMAGE = "docker.io/randylemuel/${APP}"
  }
  stages {
    stage('Test') {
      agent {
        docker {
          image 'node:22-alpine'
          reuseNode true
        }
      }
      steps {
        sh 'npm ci --cache .npm'
        sh 'npm run test:ci'
      }
      post {
        always {
          junit testResults: 'reports/junit.xml', allowEmptyResults: true
        }
      }
    }

    stage('Build & Push') {
      when { anyOf { branch 'dev'; branch 'qa'; branch 'prod' } }
      steps {
        script {
          env.TAG = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        sh 'docker build -t $IMAGE:$TAG .'
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
            usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
          sh 'echo "$REG_PASS" | docker login -u "$REG_USER" --password-stdin'
          sh 'docker push $IMAGE:$TAG'
        }
      }
      post {
        always { sh 'docker logout || true' }
      }
    }

    stage('Deploy dev / qa') {
      when { anyOf { branch 'dev'; branch 'qa' } }
      steps {
        script { deployTo(env.BRANCH_NAME) }
      }
    }
  }

  post {
    success {
      script {
        if (env.BRANCH_NAME in ['dev', 'qa']) {
          echo "Pipeline berhasil: ${env.APP}:${env.TAG} berjalan di namespace ${env.BRANCH_NAME}"
        } else if (env.BRANCH_NAME == 'prod') {
          echo "Pipeline berhasil: ${env.IMAGE}:${env.TAG} siap dirilis lewat job deploy-prod"
        } else {
          echo 'Pipeline berhasil: test lolos'
        }
      }
    }
    failure {
      echo 'Pipeline gagal: cek stage yang berwarna merah'
    }
  }
}
