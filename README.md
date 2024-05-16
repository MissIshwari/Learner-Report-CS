Create container with docker files for both frontend and backend 
docker build -t misscoder21/learner-report-backend:v1 .
docker build -t misscoder21/learner-report-frontend:v1 .

Create kubernetes deployment and service files

Start minikube locally
minikube start

Apply deployment and services
kubectl apply -f deploymentfile
kubectl apply -f servicefile

Creation of helm charts

CICD pipeline
Docker build
Push to ECR
Kubernetes deployment with Helm on EKS
Jenkins pipeline
