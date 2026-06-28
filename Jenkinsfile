pipeline {
    agent any

    environment {
        CONTAINER_NAME = 'my-public-nginx'
        PUBLIC_IMAGE   = 'nginx:alpine'
        HOST_PORT      = '80'
    }

    stages {
        stage('Pull Public Image') {
            steps {
                // Ensure the latest version of the public Nginx image is pulled
                sh "docker pull ${PUBLIC_IMAGE}"
            }
        }

        stage('Stop Existing Container') {
            steps {
                // Safely remove any older running instance of the container
                sh "docker stop ${CONTAINER_NAME} || true"
                sh "docker rm ${CONTAINER_NAME} || true"
            }
        }

        stage('Deploy Nginx') {
            steps {
                // Run the official public image and expose it on the host port
                sh "docker run -d -p ${HOST_PORT}:80 --name ${CONTAINER_NAME} ${PUBLIC_IMAGE}"
            }
        }
    }

    post {
        success {
            echo "Public Nginx image successfully deployed on port ${HOST_PORT}!"
        }
        failure {
            echo "Deployment failed. Please check the Docker logs."
        }
    }
}
