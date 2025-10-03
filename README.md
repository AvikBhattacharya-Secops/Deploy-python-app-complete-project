# Deploy-python-app-complete-project

Jenkins builds Docker image 

Pushes to DockerHub and AWS ECR

Updates Helm values.yaml with new image tag

Commits and pushes change to GitHub

ArgoCD detects Git change and deploys via Helm

K8s exposes the service via LoadBalancer

Optionally scales based on traffic

Deploy-python-app-complete-project/


├── helm/
│   ├── Chart.yaml               # Helm chart metadata
│   ├── values.yaml              # Configuration for Helm deployment
│   ├── templates/
│   │   ├── deployment.yaml      # Kubernetes deployment template
│   │   ├── service.yaml         # Kubernetes service template
│   │   ├── hpa.yaml             # Horizontal Pod Autoscaler
│   │   ├── ingress.yaml         # Optional if using ingress
│   └── values-production.yaml   # Optional, production-specific config
├── Jenkinsfile                  # Jenkins pipeline script
├── Dockerfile                   # Dockerfile to build the image
├── README.md                    # Project documentation
└── requirements.txt             # Python dependencies for Docker image
