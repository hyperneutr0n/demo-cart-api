pipeline {
  agent any
  parameters {
    string(name: 'IMAGE_TAG', defaultValue: '',
           description: 'SHA commit yang akan di-deploy. Kosongkan untuk commit terbaru di branch prod.')
  }
  environment {
    APP       = 'express-demo'
    IMAGE     = "docker.io/USERNAME/${APP}"
    NS        = 'prod'
    NODE_PORT = '30080'
  }
  stages {
    stage('Tentukan versi') {
      steps {
        script {
          env.TAG = params.IMAGE_TAG?.trim() ?:
                    sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
          currentBuild.description = "prod ← ${env.TAG}"
        }
      }
    }
 
    stage('Cek image di registry') {
      steps {
        // Gagal kalau image dengan SHA ini belum pernah di-build dan di-push oleh job Multibranch
        sh 'docker manifest inspect $IMAGE:$TAG > /dev/null'
      }
    }
 
    stage('Deploy prod') {
      steps {
        withCredentials([file(credentialsId: 'k3s-kubeconfig', variable: 'KUBECONFIG')]) {
          sh '''
            kubectl create namespace "$NS" --dry-run=client -o yaml | kubectl apply -f -
            sed -e "s|IMAGE_PLACEHOLDER|$IMAGE:$TAG|" -e "s|NODEPORT_PLACEHOLDER|$NODE_PORT|" k8s/app.yaml | kubectl apply -n "$NS" -f -
            kubectl rollout status deployment/$APP -n "$NS" --timeout=120s
          '''
        }
      }
    }
  }
 
  post {
    success {
      echo "Deploy berhasil: ${env.IMAGE}:${env.TAG} berjalan di namespace prod (port ${env.NODE_PORT})"
    }
    failure {
      echo 'Deploy gagal: cek apakah SHA benar dan image sudah ada di registry'
    }
  }
}