# Deploy-python-app-complete-project

Jenkins builds Docker image 

Pushes to DockerHub and AWS ECR

Updates Helm values.yaml with new image tag

Commits and pushes change to GitHub

ArgoCD detects Git change and deploys via Helm

K8s exposes the service via LoadBalancer

Optionally scales based on traffic

Deploy-python-app-complete-project/



