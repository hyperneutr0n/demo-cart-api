pipeline {
  agent any
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

    stage('Build Image') {
      steps {
        sh 'docker compose build'
      }
    }

    stage('Deploy') {
      steps {
        sh 'docker compose up -d --wait'
      }
    }
  }

  post {
    success {
      echo 'Pipeline berhasil: cart-api berjalan di port 3000'
    }
    failure {
      echo 'Pipeline gagal: cek stage yang berwarna merah'
    }
  }
}
