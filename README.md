# Kubernetes ECR Web Application Deployment  

[![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.27+-326CE5?logo=kubernetes)](https://kubernetes.io/)
[![AWS ECR](https://img.shields.io/badge/AWS_ECR-FF9900?logo=amazon-aws)](https://aws.amazon.com/ecr/)
[![Docker](https://img.shields.io/badge/Docker-v24.0+-2496ED?logo=docker)](https://www.docker.com/)

This project demonstrates secure deployment of a containerized Node.js web application to a Kubernetes cluster using **AWS Elastic Container Registry (ECR)** as a private Docker registry. It highlights industry best practices for credential management via Kubernetes Secrets and declarative deployment configurations.

## 🔥 Key Features  
- **Secure Private Registry Access**: Kubernetes Secrets for authenticating with AWS ECR.  
- **Infrastructure-as-Code**: Declarative Kubernetes manifests for reproducible deployments.  
- **Production-Grade Configs**: Optimized Dockerfile and resource-constrained deployments.  
- **Minimalist Web App**: Demo Node.js app with health checks and static assets.  

## 🛠️ Technologies Used  
| Component       | Technology             |
|-----------------|------------------------|
| Orchestration   | Kubernetes 1.27+      |
| Container Registry | AWS ECR               |
| Containerization| Docker 24.0+          |
| Web Framework   | Node.js (Express.js)  |

---

## 📂 Project Structure  
```bash
js-app/
├── app/                      # Application source
│   ├── images/               # Static assets
│   ├── index.html            # Frontend entrypoint
│   ├── server.js             # Express.js backend
│   ├── package.json          # Node.js dependencies
├── kube/                     # Kubernetes manifests
│   ├── docker-secret.yml     # ECR credential secret
│   ├── myapp-deployment.yml  # Deployment + Service
│   └── myapp-service.yml     # NodePort service
├── Dockerfile                # Production Docker image
└── docker-compose.yaml       # Local development
```

## 🚀 Deployment Workflow
### Prerequisites
- AWS CLI configured with IAM ECR access
- kubectl configured for target cluster
- Docker Engine 24.0+

### Step 1: Build & Push to AWS ECR
```bash
# Login to ECR
aws ecr get-login-password | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com

# Build image
docker build -t webapp:v1 .

# Tag and push
docker tag webapp:v1 <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/webapp:v1
docker push <AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com/webapp:v1
```
### Step 2: Create Kubernetes Secret for ECR
```bash
kubectl create secret docker-registry ecr-cred \
  --docker-server=<AWS_ACCOUNT_ID>.dkr.ecr.<REGION>.amazonaws.com \
  --docker-username=AWS \
  --docker-password=$(aws ecr get-login-password) \
  --namespace=default
```
### Step 3: Deploy to Kubernetes
```bash
# Apply manifests (from kube/ directory)
kubectl apply -f docker-secret.yml
kubectl apply -f myapp-deployment.yml
kubectl apply -f myapp-service.yml

# Verify
kubectl get pods -l app=webapp
kubectl get svc webapp-service
```

## 🌐 Access the Application
After deployment, use the NodePort service to access the app:
```bash
# Get Node IP and Port
NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
NODE_PORT=$(kubectl get svc webapp-service -o jsonpath='{.spec.ports[0].nodePort}')

# Access in browser
echo http://${NODE_IP}:${NODE_PORT}
```
## 🔒 Security Notes
- Secrets Management: Never commit actual credentials. Use kubectl create secret with environment variables.
- IAM Roles: For production, assign ECR permissions via IAM roles to worker nodes.
- Image Scanning: Enable ECR image vulnerability scanning in AWS.

## 📜 License




