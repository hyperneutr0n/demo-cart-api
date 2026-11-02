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
                sh'echo "hello jenkins"'
            }
        }
    }
}