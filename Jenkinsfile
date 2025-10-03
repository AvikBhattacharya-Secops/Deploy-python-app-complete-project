pipeline {
    agent any

    environment {
        // Jenkins credentials
        DOCKERHUB_CREDENTIALS = credentials('docker-hub-creds')            // DockerHub credentials
        GIT_CREDENTIALS = credentials('GitAccess')                         // GitHub credentials
        ARGOCD_CREDENTIALS = credentials('argocd')                         // ArgoCD credentials

        // AWS and DockerHub details
        REGION = 'ap-south-1'
        IMAGE_TAG = "${BUILD_NUMBER}"
        AWS_ACCOUNT_ID = '439110395780'
        ECR_REPO = "${AWS_ACCOUNT_ID}.dkr.ecr.${REGION}.amazonaws.com/my-repo" // Ensure ECR repo exists
        DOCKERHUB_REPO = 'avikbhattacharya056/my-python-app-image'  // Ensure DockerHub repo exists
        ARGOCD_SERVER = '65.1.136.255:30976'  // Ensure ArgoCD server is reachable
        GITHUB_REPO = 'https://github.com/AvikBhattacharya-Secops/Deploy-python-app-complete-project.git' // GitHub repo for the new app
    }

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t $DOCKERHUB_REPO:$IMAGE_TAG ."
                }
            }
        }

        stage('Push Docker Image to DockerHub') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'docker-hub-creds', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
                        sh "docker login -u $DOCKER_USERNAME -p $DOCKER_PASSWORD"
                        sh "docker push $DOCKERHUB_REPO:$IMAGE_TAG"
                    }
                }
            }
        }

        stage('Push Docker Image to AWS ECR') {
            steps {
                script {
                    withCredentials([aws(credentialsId: 'aws-credentials-id')]) {
                        sh 'aws ecr get-login-password --region $REGION | docker login --username AWS --password-stdin $ECR_REPO'
                        sh "docker tag $DOCKERHUB_REPO:$IMAGE_TAG $ECR_REPO:$IMAGE_TAG"
                        sh "docker push $ECR_REPO:$IMAGE_TAG"
                    }
                }
            }
        }

        stage('Update Helm values.yaml') {
            steps {
                script {
                    // Update the Helm values.yaml with the new image tag and NodePort
                    sh """
                    sed -i 's/tag: .*/tag: $IMAGE_TAG/' path/to/helm/values.yaml
                    sed -i 's/nodePort: .*/nodePort: 30976/' path/to/helm/values.yaml  # Update NodePort to 30976
                    """
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: 'argocd', usernameVariable: 'ARGO_USERNAME', passwordVariable: 'ARGO_PASSWORD')]) {
                        sh """
                        helm upgrade --install my-python-app ./path/to/helm/chart --set image.tag=$IMAGE_TAG --set service.type=NodePort --set service.nodePort=30976
                        """
                    }
                }
            }
        }

        stage('Expose App via NodePort') {
            steps {
                script {
                    // Expose the app on NodePort (30976)
                    sh """
                    kubectl expose deployment my-python-app --type=NodePort --name=my-python-app-service --port=80 --target-port=80 --node-port=30976
                    """
                }
            }
        }
    }
}
