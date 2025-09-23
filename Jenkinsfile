pipeline {
    agent any  // Ensure agent has Docker and kubectl

    environment {
        DOCKER_IMAGE = 'bha83/flask-app'  // Replace with your DockerHub (e.g., bha834/flask-app)
        DOCKER_TAG = "${BUILD_NUMBER}"  // Use build number as tag
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')  // Must match Jenkins credential ID
        GIT_REPO = 'https://github.com/Bha834/project-4.git'  // Your actual repo URL
        // For EKS: AWS_SHARED_CREDENTIALS_FILE = credentials('aws-creds')  // If needed
    }

    stages {
        stage('Pull Code from GitHub') {
            steps {
                // Since SCM checkout already happened, this is optional/re-pull
                git branch: 'main', url: "${GIT_REPO}", credentialsId: 'github-creds'  // Add creds if private
                echo 'Code pulled successfully from GitHub'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build Docker image
                    def image = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    // Login to DockerHub and push (auto logout after block)
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        image.push("${DOCKER_TAG}")
                        image.push('latest')  // Optional: push as latest
                    }
                    // Clean up local image (optional)
                    sh "docker rmi ${DOCKER_IMAGE}:${DOCKER_TAG} || true"
                }
                echo 'Docker image built and pushed to DockerHub'
            }
        }

        stage('Apply Kubernetes Manifests') {
            steps {
                script {
                    // Dynamically update image tag in deployment.yaml
                    sh """
                        sed -i 's/yourdockerhubusername\\/flask-app:v1/${DOCKER_IMAGE}:\${DOCKER_TAG}/g' deployment.yaml || true
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        kubectl rollout status deployment/flask-app --timeout=300s
                    """
                    // For EKS auth (if using): sh 'aws eks update-kubeconfig --name flask-cluster --region us-west-2'
                }
                echo 'Kubernetes manifests applied successfully'
            }
        }
    }

    post {
        success {
            echo 'CI/CD Pipeline succeeded! App is deployed. Check Kubernetes pods.'
        }
        failure {
            echo 'CI/CD Pipeline failed! Check logs for details.'
        }
        // Removed 'always' sh 'docker logout' - not needed, as withRegistry handles it
        // If you want cleanup, add a final stage instead
    }
}
