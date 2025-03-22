pipeline {
    agent any
    environment {
        DOCKER_IMAGE = "jegadeep/docker-app:latest"  // Ensure this repo exists on Docker Hub
        CONTAINER_NAME = "docker-running-app-1"
        REGISTRY_CREDENTIALS = "jega1"  // Ensure this matches Jenkins credentials ID
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jega', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/jegadeeppandiarajan/jenkins.git", branch: 'main'
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $DOCKER_IMAGE .'
                }
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jega1', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        if [ $? -ne 0 ]; then
                            echo "Docker login failed!"
                            exit 1
                        fi
                        '''
                    }
                }
            }
        }

        stage('Push to Container Registry') {
            steps {
                script {
                    sh '''
                    docker push $DOCKER_IMAGE
                    if [ $? -ne 0 ]; then
                        echo "Docker push failed!"
                        exit 1
                    fi
                    '''
                }
            }
        }

        stage('Stop & Remove Existing Container') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$CONTAINER_NAME)" ]; then
                        echo "Stopping and removing existing container..."
                        docker stop $CONTAINER_NAME || true
                        docker rm $CONTAINER_NAME || true
                    fi
                    '''
                }
            }
        }

        stage('Run Docker Container') {
            steps {
                script {
                    sh '''
                    docker run -d -p 5001:5000 --name $CONTAINER_NAME $DOCKER_IMAGE
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Build, push, and container execution successful!"
        }
        failure {
            echo "Build or container execution failed."
        }
    }
}
