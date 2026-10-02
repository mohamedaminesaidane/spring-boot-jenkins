pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'mohamedamine2603/backend-app:latest'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            steps {
                sh 'mvn clean package'
            }
        }

        stage('Docker Build') {
            steps {
                sh '''
                    docker build -t backend-app:latest .
                    docker tag backend-app:latest $DOCKER_IMAGE
                '''
            }
        }

        stage('Docker Push & Deploy') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-credentials',
                        usernameVariable: 'DOCKER_USERNAME',
                        passwordVariable: 'DOCKER_PASSWORD'
                    )
                ]) {
                    sh '''
                        echo "$DOCKER_PASSWORD" | docker login \
                            -u "$DOCKER_USERNAME" \
                            --password-stdin

                        echo "=== Pushing image to Docker Hub ==="
                        docker push $DOCKER_IMAGE

                        echo "=== Removing old backend container ==="
                        docker rm -f backend-app 2>/dev/null || true

                        echo "=== Pulling image from Docker Hub ==="
                        docker pull $DOCKER_IMAGE

                        echo "=== Starting backend container ==="
                        docker run -d \
                            --name backend-app \
                            --network devops-network \
                            -p 8082:8082 \
                            $DOCKER_IMAGE

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                sh '''
                    if docker ps --format '{{.Names}}' | grep -q '^mysql$'; then
                        echo "MySQL is already running. Keeping existing container."
                    else
                        echo "MySQL is not running. Starting MySQL..."

                        docker rm -f mysql 2>/dev/null || true

                        docker run -d \
                            --name mysql \
                            --network devops-network \
                            -e MYSQL_ROOT_PASSWORD=root \
                            -e MYSQL_DATABASE=timesheet-devops-db \
                            mysql:8.0

                        echo "Waiting for MySQL..."
                        sleep 20
                    fi
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    echo "Waiting for backend..."
                    sleep 20

                    echo "=== Docker Containers ==="
                    docker ps

                    echo "=== Backend Logs ==="
                    docker logs backend-app
                '''
            }
        }
    }
}

