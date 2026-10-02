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
                sh 'docker build -t backend-app:latest .'
                sh 'docker tag backend-app:latest $DOCKER_IMAGE'
            }
        }

        stage('Docker Push') {
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

                        docker push $DOCKER_IMAGE

                        docker logout
                    '''
                }
            }
        }

        stage('Deploy MySQL') {
            steps {
                sh '''
                    docker rm -f mysql 2>/dev/null || true

                    docker run -d \
                        --name mysql \
                        --network devops-network \
                        -e MYSQL_ROOT_PASSWORD=root \
                        -e MYSQL_DATABASE=timesheet-devops-db \
                        mysql:8.0

                    sleep 20
                '''
            }
        }

        stage('Deploy Backend') {
            steps {
                sh '''
                    docker rm -f backend-app 2>/dev/null || true

                    docker pull $DOCKER_IMAGE

                    docker run -d \
                        --name backend-app \
                        --network devops-network \
                        -p 8082:8082 \
                        $DOCKER_IMAGE
                '''
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    sleep 20
                    docker ps
                    docker logs backend-app
                '''
            }
        }
    }
}
