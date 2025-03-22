pipeline {
    agent any

    environment {
        BACKEND_IMAGE = "jegadeep/backend-app:latest"
        FRONTEND_IMAGE = "jegadeep/frontend-app:latest"
        CONTAINER_BACKEND = "backend-container"
        CONTAINER_FRONTEND = "frontend-container"
        REGISTRY_CREDENTIALS = "jega1"  // Jenkins credentials ID for Docker login
        GIT_CREDENTIALS = "jega"  // Jenkins credentials ID for GitHub
    }

    stages {
        stage('Checkout Code') {
            steps {
                withCredentials([usernamePassword(credentialsId: GIT_CREDENTIALS, usernameVariable: 'GIT_USER', passwordVariable: 'GIT_TOKEN')]) {
                    git url: "https://$GIT_USER:$GIT_TOKEN@github.com/jegadeeppandiarajan/k8s.git", branch: 'main'
                }
            }
        }

        // ================= BACKEND BUILD & DEPLOY =================
        stage('Build Backend Image') {
            steps {
                script {
                    sh '''
                    cd backend
                    docker build -t $BACKEND_IMAGE .
                    '''
                }
            }
        }

        stage('Push Backend Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: REGISTRY_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $BACKEND_IMAGE
                        '''
                    }
                }
            }
        }

        // ================= FRONTEND BUILD & DEPLOY =================
        stage('Build Frontend Image') {
            steps {
                script {
                    sh '''
                    cd frontend
                    docker build -t $FRONTEND_IMAGE .
                    '''
                }
            }
        }

        stage('Push Frontend Image') {
            steps {
                withCredentials([usernamePassword(credentialsId: REGISTRY_CREDENTIALS, usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    script {
                        sh '''
                        echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                        docker push $FRONTEND_IMAGE
                        '''
                    }
                }
            }
        }

        // ================= FIX: AUTHENTICATE KUBERNETES =================
        stage('Authenticate Kubernetes') {
            steps {
                withCredentials([string(credentialsId: 'K8S_TOKEN', variable: 'KUBE_TOKEN')]) {
                    script {
                        sh '''
                        kubectl config set-credentials jenkins --token=$KUBE_TOKEN
                        kubectl config set-context --current --user=jenkins
                        '''
                    }
                }
            }
        }

        // ================= DEPLOY TO KUBERNETES =================
        stage('Deploy to Kubernetes') {
            steps {
                script {
                    sh '''
                    kubectl apply -f k8s/backend-deployment.yaml --validate=false
                    kubectl apply -f k8s/frontend-deployment.yaml --validate=false
                    kubectl apply -f k8s/service.yaml --validate=false
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "Frontend & Backend successfully built, pushed, and deployed to Kubernetes!"
        }
        failure {
            echo "Pipeline failed. Check logs for details."
        }
    }
}
