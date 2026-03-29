Built a full GitOps CI/CD pipeline using Node.js, Docker, Kubernetes, and Argo CD. The application is containerized, stored in Docker Hub, and automatically deployed to a Kubernetes cluster via declarative Git-based workflows.


# gitops-k8s-manifests

STEP 1: Create Node.js App 
# Initialize project
mkdir gitops-app
cd gitops-app
npm init -y

#Install Express
npm install express

# Create app.js
const express = require("express");

const app = express();

app.get("/", (req, res) => {
  res.send("🚀 GitOps App Running on Kubernetes!");
});

const PORT = 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});

#package.json


{
  "name": "gitops-app",
  "version": "1.0.0",
  "description": "Node.js app for GitOps Kubernetes deployment",
  "main": "app.js",
  "scripts": {
    "start": "node app.js"
  },
  "dependencies": {
    "express": "^4.18.2"
  }
}

# STEP 2: Dockerize Application
  # Create Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 3000

CMD ["npm", "start"]

  # Build Docker Image
docker build -t aderixxe/gitops-app:v1 .
  # Login to Docker Hub
docker login
  # Push Image to Docker Hub
docker push aderixxe/gitops-app:v1

# STEP 3: Start Kubernetes (Minikube)
  # Start cluster
minikube start --driver=docker --cpus=2 --memory=1800
  # Verify cluster
kubectl get nodes


# STEP 4: Kubernetes Manifests
  # deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gitops-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: gitops-app
  template:
    metadata:
      labels:
        app: gitops-app
    spec:
      containers:
        - name: gitops-app
          image: your-dockerhub-username/gitops-app:v1
          ports:
            - containerPort: 3000
  # service.yaml
apiVersion: v1
kind: Service
metadata:
  name: gitops-service
spec:
  type: NodePort
  selector:
    app: gitops-app
  ports:
    - port: 3000
      targetPort: 3000
      nodePort: 30001
  # Deploy to Kubernetes
kubectl apply -f deployment.yaml
kubectl apply -f service.yaml
  # Access Application
minikube service gitops-service

# STEP 5  GitOps with Argo CD
  # Install Argo CD
kubectl create namespace argocd
kubectl apply -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
  
  # Access UI
kubectl port-forward svc/argocd-server -n argocd 8080:443

  # Open:
https://localhost:8080
  # Get password
kubectl -n argocd get secret argocd-initial-admin-secret \
-o jsonpath="{.data.password}" | base64 -d
  # Connect GitHub Repo
  # Your repo should contain:

gitops-k8s-manifests/
├── deployment.yaml
├── service.yaml


# Argo CD Setup & Application Deployment

This is a one-time setup step (already completed if Argo CD is installed)

1. Create Application in Argo CD

In the Argo CD UI:

Click NEW APP
Set Application Name: gitops-app
Project: default

2. Source Configuration
Repository URL: https://github.com/aderixxe/gitops-k8s-manifests
Revision: main
Path: .                                                                                                         
3. Destination Configuration
Cluster: https://kubernetes.default.svc
Namespace: default                                                                                          
4. Sync Application

Click:

CREATE → SYNC → SYNCHRONIZE

After Setup (Important)

Once the application is created:

Kubernetes resources are automatically managed by Argo CD

FINAL FLOW
Developer writes code
        ↓
Docker builds image
        ↓
Push to Docker Hub
        ↓
Update Kubernetes YAML in GitHub
        ↓
Argo CD detects change
        ↓
Auto deploy to Kubernetes cluster





