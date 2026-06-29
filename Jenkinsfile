pipeline {
    agent any

    environment {
        // Make Docker CLI available to Jenkins on macOS
        PATH = "/Applications/Docker.app/Contents/Resources/bin:/usr/local/bin:/opt/homebrew/bin:${env.PATH}"

        // JFrog is running locally on the same Mac
        JFROG_REGISTRY = '192.168.68.58:8082'
        JFROG_REPOSITORY = 'docker-local'

        IMAGE_NAME = 'nginx-demo'
        IMAGE_TAG  = "${BUILD_NUMBER}"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Create Nginx Website') {
            steps {
                sh '''
                    set -e

                    mkdir -p website

                    cat > website/index.html <<'EOF'
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Jenkins Nginx Deployment</title>
</head>
<body>
    <h1>Hello from Jenkins!</h1>
    <p>This Nginx Docker image was built by Jenkins.</p>
    <p>The image was pushed to JFrog Artifactory.</p>
</body>
</html>
EOF

                    cat > Dockerfile <<'EOF'
FROM nginx:alpine

COPY website/index.html /usr/share/nginx/html/index.html

EXPOSE 80
EOF

                    echo "Created files:"
                    find . -maxdepth 2 -type f -print
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                    set -e

                    FULL_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:$IMAGE_TAG"

                    echo "Building image:"
                    echo "$FULL_IMAGE"

                    docker build \
                      --tag "$FULL_IMAGE" \
                      .
                '''
            }
        }

        stage('Validate Docker Image') {
            steps {
                sh '''
                    set -e

                    FULL_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:$IMAGE_TAG"

                    echo "Checking Nginx configuration:"
                    docker run --rm "$FULL_IMAGE" nginx -t
                '''
            }
        }

        stage('Tag Latest') {
            steps {
                sh '''
                    set -e

                    FULL_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:$IMAGE_TAG"
                    LATEST_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:latest"

                    docker tag "$FULL_IMAGE" "$LATEST_IMAGE"

                    echo "Created tags:"
                    echo "$FULL_IMAGE"
                    echo "$LATEST_IMAGE"
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
                        set -e

                        echo "$JFROG_PASSWORD" | docker login \
                          "$JFROG_REGISTRY" \
                          --username "$JFROG_USERNAME" \
                          --password-stdin
                    '''
                }
            }
        }

        stage('Push Image to JFrog') {
            steps {
                sh '''
                    set -e

                    FULL_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:$IMAGE_TAG"
                    LATEST_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:latest"

                    echo "Pushing build-number image:"
                    docker push "$FULL_IMAGE"

                    echo "Pushing latest image:"
                    docker push "$LATEST_IMAGE"
                '''
            }
        }

        stage('Deployment Information') {
            steps {
                sh '''
                    FULL_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:$IMAGE_TAG"
                    LATEST_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:latest"

                    echo "--------------------------------------------"
                    echo "Images successfully pushed to JFrog:"
                    echo "$FULL_IMAGE"
                    echo "$LATEST_IMAGE"
                    echo "--------------------------------------------"
                '''
            }
        }
    }

    post {
        success {
            echo 'Docker image was successfully built and pushed to JFrog.'
        }

        failure {
            echo 'The Docker image build or push failed. Review the failed stage above.'
        }

        always {
            sh '''
                FULL_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:$IMAGE_TAG"
                LATEST_IMAGE="$JFROG_REGISTRY/$JFROG_REPOSITORY/$IMAGE_NAME:latest"

                echo "Logging out from JFrog:"
                docker logout "$JFROG_REGISTRY" || true

                echo "Removing local image tags:"
                docker image rm "$FULL_IMAGE" || true
                docker image rm "$LATEST_IMAGE" || true
            '''

            cleanWs(
                deleteDirs: true,
                disableDeferredWipeout: true,
                notFailBuild: true
            )
        }
    }
}
