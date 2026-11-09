pipeline {
    agent any

    options {
        skipDefaultCheckout(true)
    }

    environment {
        IMAGE_NAME = 'devops-web'
        CONTAINER_NAME = 'devops-web'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                sh 'ls -la'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                    -t ${IMAGE_NAME}:${BUILD_NUMBER} .
                '''
            }
        }

        stage('Test Docker Image') {
            steps {
                sh '''
                    docker run --rm \
                    --entrypoint sh \
                    ${IMAGE_NAME}:${BUILD_NUMBER} \
                    -c "test -s /usr/share/nginx/html/index.html"
                '''
            }
        }

        stage('Deploy Application') {
            steps {
                sh '''
                    docker rm -f ${CONTAINER_NAME} || true

                    docker run -d \
                    --name ${CONTAINER_NAME} \
                    --restart unless-stopped \
                    -p 8085:80 \
                    ${IMAGE_NAME}:${BUILD_NUMBER}
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    docker exec ${CONTAINER_NAME} \
                    sh -c "wget -qO- http://127.0.0.1 | grep -q 'Jenkins CI/CD Pipeline Successful'"

                    docker ps
                '''
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline completed successfully!'
        }

        failure {
            echo 'CI/CD Pipeline failed. Check Console Output.'
        }
    }
}
