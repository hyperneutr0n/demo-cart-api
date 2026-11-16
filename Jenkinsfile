pipeline {
  agent none
  environment {
    IMAGE = 'docker.io/USERNAME/express-demo'
  }
  stages {
    stage('Test') {
      agent any
      steps {
        script {
          env.TAG = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
        }
        sh 'npm ci && npm test'
      }
    }
    stage('Build & Push') {
      agent any
      when { anyOf { branch 'dev'; branch 'qa'; branch 'prod' } }
      steps {
        sh 'docker build -t $IMAGE:$TAG .'
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
            usernameVariable: 'REG_USER', passwordVariable: 'REG_PASS')]) {
          sh 'echo "$REG_PASS" | docker login -u "$REG_USER" --password-stdin'
          sh 'docker push $IMAGE:$TAG'
        }
      }
      post { always { sh 'docker logout || true' } }
    }
    stage('Deploy dev / qa') {
      agent any
      when { anyOf { branch 'dev'; branch 'qa' } }
      steps { script { deployTo(env.BRANCH_NAME) } }
    }
  }
}