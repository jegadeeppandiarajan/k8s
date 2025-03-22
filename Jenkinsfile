pipeline {
    agent any
    environment {
        // Define image names
        BACKEND_IMAGE = "jegadeep/docker-backend:latest"
        FRONTEND_IMAGE = "jegadeep/docker-frontend:latest"
        // Define container names
        BACKEND_CONTAINER = "docker-running-backend"
        FRONTEND_CONTAINER = "docker-running-frontend"
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jega', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/jegadeeppandiarajan/jenkins.git", branch: 'main'
                }
            }
        }

        // ✅ Debugging step to check directory structure
        stage('Verify Directories') {
            steps {
                sh '''
                echo "Current Workspace:"
                pwd
                echo "Listing files:"
                ls -l
                '''
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                sh '''
                if [ -d "$WORKSPACE/backend" ]; then
                    docker build -t $BACKEND_IMAGE -f $WORKSPACE/backend/dockerfile $WORKSPACE/backend
                else
                    echo "Error: Backend directory not found!"
                    exit 1
                fi
                '''
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                sh '''
                if [ -d "$WORKSPACE/frontend" ]; then
                    docker build -t $FRONTEND_IMAGE -f $WORKSPACE/frontend/dockerfile $WORKSPACE/frontend
                else
                    echo "Error: Frontend directory not found!"
                    exit 1
                fi
                '''
            }
        }

        stage('Login to Docker Registry') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'jega1', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh '''
                        if [ -z "$DOCKER_PASS" ]; then
                            echo "Error: Docker password is empty!"
                            exit 1
                        fi
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        '''
                    }
                }
            }
        }

        stage('Push Docker Images') {
            steps {
                sh 'docker push $BACKEND_IMAGE'
                sh 'docker push $FRONTEND_IMAGE'
            }
        }

        stage('Stop & Remove Existing Containers') {
            steps {
                script {
                    sh '''
                    if [ "$(docker ps -aq -f name=$BACKEND_CONTAINER)" ]; then
                        docker stop $BACKEND_CONTAINER || true
                        docker rm $BACKEND_CONTAINER || true
                    fi
                    if [ "$(docker ps -aq -f name=$FRONTEND_CONTAINER)" ]; then
                        docker stop $FRONTEND_CONTAINER || true
                        docker rm $FRONTEND_CONTAINER || true
                    fi
                    '''
                }
            }
        }

        stage('Run Docker Containers') {
            steps {
                script {
                    sh '''
                    docker images | grep "$(echo $BACKEND_IMAGE | cut -d ':' -f 1)" || { echo "Error: Backend image not found!"; exit 1; }
                    docker images | grep "$(echo $FRONTEND_IMAGE | cut -d ':' -f 1)" || { echo "Error: Frontend image not found!"; exit 1; }

                    docker run -d -p 5000:5000 --name $BACKEND_CONTAINER $BACKEND_IMAGE
                    docker run -d -p 5001:80 --name $FRONTEND_CONTAINER $FRONTEND_IMAGE
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Build, push, and container execution successful!"
        }
        failure {
            echo "❌ Build or container execution failed. Check logs for details."
        }
    }
}
