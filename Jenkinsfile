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
                sh '''
                    docker build -t my-website:latest .
                '''
            }
        }

        stage('Stop Old Container') {
            steps {
                sh '''
                    docker stop my-website || true
                    docker rm my-website || true
                '''
            }
        }

        stage('Deploy Container') {
            steps {
                sh '''
                    docker run -d \
                      --name my-website \
                      -p 80:80 \
                      my-website:latest
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker deployment successful!'
        }

        failure {
            echo 'Docker deployment failed!'
        }
    }
}
