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
        stage('Checkout') {
            steps {
                script {
                    echo "Cloning from GitHub main branch..."
                    checkout scm
                }
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    echo "Building Docker image..."
                    sh "docker build -t ${DOCKERHUB_REPO}:${IMAGE_TAG} ."
                    sh "docker tag ${DOCKERHUB_REPO}:${IMAGE_TAG} ${ECR_REPO}:${IMAGE_TAG}"
                }
            }
        }

        stage('Push to DockerHub') {
            steps {
                script {
                    echo "Logging in and pushing to DockerHub..."
                    sh """
                        echo "${DOCKERHUB_CREDENTIALS_PSW}" | docker login -u "${DOCKERHUB_CREDENTIALS_USR}" --password-stdin
                        docker push ${DOCKERHUB_REPO}:${IMAGE_TAG}
                        docker logout
                    """
                }
            }
        }

        stage('Push to AWS ECR') {
            steps {
                script {
                    echo "Logging in and pushing to AWS ECR..."
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr-creds']]) {
                        sh """
                            aws ecr get-login-password --region ${REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                            docker push ${ECR_REPO}:${IMAGE_TAG}
                            docker logout
                        """
                    }
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                script {
                    echo "Updating Helm values.yaml..."
                    sh """
                        sed -i 's|repository:.*|repository: ${DOCKERHUB_REPO}|' helm/values.yaml
                        sed -i 's|tag:.*|tag: ${IMAGE_TAG}|' helm/values.yaml
                    """
                }
            }
        }

        stage('Push Helm Changes') {
            steps {
                script {
                    echo "Committing and pushing Helm changes to GitHub..."
                    withCredentials([usernamePassword(credentialsId: 'GitAccess', usernameVariable: 'GIT_USER', passwordVariable: 'GIT_PASSWORD')]) {
                        // URL encode the GitHub username to handle special characters like '@'
                        def encodedUsername = GIT_USER.replace('@', '%40')

                        sh """
                            git config user.email 'ci@jenkins.com'
                            git config user.name 'Jenkins CI'

                            # Ensure we are on the 'main' branch before committing and pushing
                            git checkout main || git checkout -b main  # Checkout main or create if doesn't exist

                            git add helm/values.yaml
                            git diff --cached --quiet || git commit -m 'Update image tag to ${IMAGE_TAG}'

                            # Push the changes using the GitHub credentials for authentication
                            git push https://${encodedUsername}:${GIT_PASSWORD}@github.com/AvikBhattacharya-Secops/Deploy-python-app-complete-project.git main
                        """
                    }
                }
            }
        }

        stage('Deploy to Kubernetes') {
            steps {
                script {
                    echo "Deploying to Kubernetes using Helm..."

                    // Use the argocd1 credentials to set the KUBECONFIG
                    withCredentials([file(credentialsId: 'argocd1', variable: 'KUBECONFIG')]) {
                        // Test the Kubernetes connection
                        sh """
                            echo "Checking Kubernetes connection..."
                            kubectl get nodes || exit 1  # Test the connection to Kubernetes

                            echo "Deploying with Helm..."
                            helm upgrade --install my-python-app ./path/to/helm/chart --set image.tag=${IMAGE_TAG} --set service.type=NodePort --set service.nodePort=30976
                        """
                    }
                }
            }
        }

        stage('Expose App via NodePort') {
            steps {
                script {
                    echo "Exposing app via NodePort..."
                    sh """
                        kubectl expose deployment my-python-app --type=NodePort --name=my-python-app-service --port=80 --target-port=80 --node-port=30976
                    """
                }
            }
        }
    }

    post {
        always {
            cleanWs()  // Clean workspace after every build
        }
    }
}
