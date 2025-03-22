pipeline {
    agent any

    environment {
        // If you need credentials (for example, for git or Docker registry),
        // ensure the correct credential IDs are used.
        GIT_TOKEN = credentials('your-git-token-id')
    }

    stages {
        stage('Checkout SCM') {
            steps {
                checkout scm
            }
        }
        stage('Verify Workspace') {
            steps {
                sh '''
                    echo "Current Workspace: $(pwd)"
                    echo "Listing files:"
                    ls -l
                '''
            }
        }
        stage('Build Docker Image') {
            steps {
                echo "Building Docker image from Dockerfile in the repository root..."
                // Build image using the Dockerfile from the root folder.
                sh 'docker build -t my-app:latest .'
            }
        }
        stage('Test Docker Image') {
            steps {
                echo "Running container for testing..."
                // This command runs the container and checks the Python version.
                // Adjust or remove this stage as per your requirements.
                sh 'docker run --rm my-app:latest python --version'
            }
        }
        // Additional stages (such as pushing to a registry or deployment) can be added here.
    }

    post {
        failure {
            echo "❌ Build or container execution failed. Check logs for details."
        }
    }
}
