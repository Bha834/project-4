pipeline {
    agent any  // Ensure agent has Docker and kubectl

    environment {
        DOCKER_IMAGE = 'bha83/flask-app'  // Replace with your DockerHub repo name
        DOCKER_TAG = "${BUILD_NUMBER}"  // Use Jenkins build number as tag (or set to 'latest')
        DOCKER_CREDENTIALS = credentials('dockerhub-credentials')  // Jenkins credential ID for DockerHub login
        GIT_REPO = 'https://github.com/Bha834/project-4.git'  // Replace with your GitHub repo URL
        // For AWS EKS: Add AWS credentials if needed (e.g., AWS_ACCESS_KEY_ID = credentials('aws-creds'))
        // KUBE_CONFIG = credentials('kubeconfig-id')  // Optional: If using a secret for kubeconfig
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
                        sed -i 's/bha83\\/flask-app:v1/${DOCKER_IMAGE}:${DOCKER_TAG}/g' deployment.yaml
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
