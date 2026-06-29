pipeline {
    agent any

    environment {
        // Do not include https://
        JFROG_REGISTRY   = 'yourcompany.jfrog.io'

        // JFrog local Docker repository name
        JFROG_REPOSITORY = 'docker-local'

        IMAGE_NAME       = 'nginx-demo'
        IMAGE_TAG        = "${BUILD_NUMBER}"
    }

    stages {
        stage('Checkout Source Code') {
            steps {
                checkout scm
            }
        }

        stage('Prepare Image Name') {
            steps {
                script {
                    env.FULL_IMAGE = "${env.JFROG_REGISTRY}/${env.JFROG_REPOSITORY}/${env.IMAGE_NAME}:${env.IMAGE_TAG}"
                    env.LATEST_IMAGE = "${env.JFROG_REGISTRY}/${env.JFROG_REPOSITORY}/${env.IMAGE_NAME}:latest"
                }

                echo "Image: ${env.FULL_IMAGE}"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    docker build \
                      --tag "$FULL_IMAGE" \
                      .
                '''
            }
        }

        stage('Validate Image') {
            steps {
                sh '''
                    docker run --rm "$FULL_IMAGE" nginx -t
                '''
            }
        }

        stage('Tag Latest Image') {
            steps {
                sh '''
                    docker tag "$FULL_IMAGE" "$LATEST_IMAGE"
                '''
            }
        }

        stage('Login to JFrog') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'jfrog-docker-credentials',
                        usernameVariable: 'JFROG_USERNAME',
                        passwordVariable: 'JFROG_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$JFROG_PASSWORD" | \
                        docker login "$JFROG_REGISTRY" \
                          --username "$JFROG_USERNAME" \
                          --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to JFrog') {
            steps {
                sh '''
                    docker push "$FULL_IMAGE"
                    docker push "$LATEST_IMAGE"
                '''
            }
        }

        stage('Display Image Information') {
            steps {
                echo """
                Image successfully pushed:

                ${env.FULL_IMAGE}
                ${env.LATEST_IMAGE}
                """
            }
        }
    }

    post {
        success {
            echo 'Docker image was successfully pushed to JFrog Artifactory.'
        }

        failure {
            echo 'The image build or push failed.'
        }

        always {
            sh '''
                docker logout "$JFROG_REGISTRY" || true
                docker image rm "$FULL_IMAGE" || true
                docker image rm "$LATEST_IMAGE" || true
            '''
        }
    }
}
