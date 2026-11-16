def deployTo(String ns) {
  def ports = [dev: '30081', qa: '30082', prod: '30080']
  withEnv(["NS=${ns}", "NODE_PORT=${ports[ns]}"]) {
    withCredentials([file(credentialsId: 'k3s-kubeconfig', variable: 'KUBECONFIG')]) {
      sh '''
        kubectl create namespace "$NS" --dry-run=client -o yaml | kubectl apply -f -
        sed -e "s|IMAGE_PLACEHOLDER|$IMAGE:$TAG|" -e "s|NODEPORT_PLACEHOLDER|$NODE_PORT|" k8s/app.yaml | kubectl apply -n "$NS" -f -
        kubectl rollout status deployment/express-demo -n "$NS" --timeout=120s
      '''
    }
  }
}

pipeline {
  agent none
  environment {
    IMAGE = 'docker.io/USERNAME/express-demo'
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