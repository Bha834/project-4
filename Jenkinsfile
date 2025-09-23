pipeline {
    agent any  

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
                git branch: 'main', url: "${GIT_REPO}"
                echo 'Code pulled successfully from GitHub'
            }
        }

        stage('Build and Push Docker Image') {
            steps {
                script {
                    // Build Docker image
                    def image = docker.build("${DOCKER_IMAGE}:${DOCKER_TAG}")
                    // Login to DockerHub and push
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS) {
                        image.push()
                        image.push('latest')  // Optional: Also push as 'latest'
                    }
                }
                echo 'Docker image built and pushed to DockerHub'
            }
        }

        stage('Apply Kubernetes Manifests') {
            steps {
                script {
                    // Update image tag in deployment.yaml (optional: use sed to replace v1 with ${DOCKER_TAG})
                    sh """
                        sed -i 's/yourdockerhubusername\\/flask-app:v1/${DOCKER_IMAGE}:${DOCKER_TAG}/g' deployment.yaml
                        kubectl apply -f deployment.yaml
                        kubectl apply -f service.yaml
                        kubectl rollout status deployment/flask-app --timeout=300s
                    """
                }
                echo 'Kubernetes manifests applied successfully'
            }
        }
    }

    post {
        always {
            sh 'docker logout'  // Clean up Docker login
        }
        success {
            echo 'CI/CD Pipeline succeeded! App is deployed.'
        }
        failure {
            echo 'CI/CD Pipeline failed! Check logs.'
        }
    }
}