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
                withCredentials([usernamePassword(credentialsId: 'github-jabastin', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/jegadeeppandiarajan/jenkins.git", branch: 'main'
                }
            }
        }

        stage('Build Backend Docker Image') {
            steps {
                // Use the backend directory as the build context
                sh 'docker build -t $BACKEND_IMAGE -f backend/dockerfile backend'
            }
        }

        stage('Build Frontend Docker Image') {
            steps {
                // Use the frontend directory as the build context
                sh 'docker build -t $FRONTEND_IMAGE -f frontend/dockerfile frontend'
            }
        }

        stage('Login to Docker Registry') {
            steps
                withCredentials([usernamePassword(credentialsId: 'jegadeep', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin'
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
                // Run backend container (exposing port 5000)
                sh 'docker run -d -p 5000:5000 --name $BACKEND_CONTAINER $BACKEND_IMAGE'
                // Run frontend container (exposing port 5001 mapped to container port 80)
                sh 'docker run -d -p 5001:80 --name $FRONTEND_CONTAINER $FRONTEND_IMAGE'
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
