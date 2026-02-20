# MERN_with_AKS
# Deploying a MERN Microservices Application on AKS

This document provides a complete step-by-step guide to deploy the MERN microservices application on Azure Kubernetes Service (AKS).

# Repository:
https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices

Project Overview

This project demonstrates:

Deployment of a MERN (MongoDB, Express, React, Node.js) microservices application

Containerization using Docker

Image hosting in Azure Container Registry (ACR)

Orchestration using Azure Kubernetes Service (AKS)

Service exposure using Kubernetes LoadBalancer

🏗 Architecture Overview

Frontend (React) → Backend APIs (Node/Express microservices) → MongoDB

All services are deployed as Kubernetes Deployments and exposed via Services.

AKS manages container orchestration, scaling, and networking.

`

├── frontend/
│   ├── Dockerfile
│   └── src/
│
├── auth-service/
│   ├── Dockerfile
│   └── src/
│
├── product-service/
│   ├── Dockerfile
│   └── src/
│
├── k8s/
│   ├── base/
│   │   ├── namespace.yaml
│   │   ├── mongo/
│   │   │   ├── mongo-secret.yaml
│   │   │   ├── mongo-pvc.yaml
│   │   │   ├── mongo-deployment.yaml
│   │   │   └── mongo-service.yaml
│   │   │
│   │   ├── backend/
│   │   │   ├── auth-deployment.yaml
│   │   │   ├── auth-service.yaml
│   │   │   ├── product-deployment.yaml
│   │   │   └── product-service.yaml
│   │   │
│   │   └── frontend/
│   │       ├── frontend-deployment.yaml
│   │       ├── frontend-service.yaml
│   │       └── frontend-hpa.yaml
│   │
│   └── README.md

`


🔧 Prerequisites

Ensure the following tools are installed:

Azure CLI

kubectl

Docker

Git

Azure Subscription

#Step 1: Clone the Repository

`
git clone https://github.com/UnpredictablePrashant/SampleMERNwithMicroservices.git
cd SampleMERNwithMicroservices
`


Step 2: Create Azure Resource Group

`
az group create --name mern-aks-rg --location eastus
`


Step 3: Create Azure Container Registry (ACR)

`
az acr create --resource-group mern-aks-rg \
              --name mernacr12345 \
              --sku Basic

`

Enable admin access:

`
az acr update -n mernacr12345 --admin-enabled true
`


Step 4: Build and Push Docker Images

Login to ACR:

`
az acr login --name mernacr12345
Build Frontend Image
docker build -t mernacr12345.azurecr.io/frontend:latest ./frontend
docker push mernacr12345.azurecr.io/frontend:latest
Build Backend Services
`

Repeat for each microservice:

`
docker build -t mernacr12345.azurecr.io/service1:latest ./service1
docker push mernacr12345.azurecr.io/service1:latest
`


Step 5: Create AKS Cluster
`
az aks create \
  --resource-group mern-aks-rg \
  --name mern-aks-cluster \
  --node-count 2 \
  --enable-addons monitoring \
  --generate-ssh-keys \
  --attach-acr mernacr12345
`

Step 6: Connect to AKS

`
az aks get-credentials \
  --resource-group mern-aks-rg \
  --name mern-aks-cluster
`

Verify connection:

`
kubectl get nodes

`


Step 7: Create Kubernetes Manifests

Create the following YAML files:

mongo-deployment.yaml

mongo-service.yaml

backend-deployment.yaml

backend-service.yaml

frontend-deployment.yaml

frontend-service.yaml

`
Example: MongoDB Deployment
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mongo
spec:
  replicas: 1
  selector:
    matchLabels:
      app: mongo
  template:
    metadata:
      labels:
        app: mongo
    spec:
      containers:
      - name: mongo
        image: mongo:6
        ports:
        - containerPort: 27017
Example: MongoDB Service
apiVersion: v1
kind: Service
metadata:
  name: mongo-service
spec:
  selector:
    app: mongo
  ports:
    - port: 27017
  clusterIP: None
Example: Frontend Service (LoadBalancer)
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  type: LoadBalancer
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 3000
`
      
Step 8: Deploy to AKS

kubectl apply -f .

Verify deployments:

kubectl get pods
kubectl get services


Step 9: Access Application

Retrieve external IP:


kubectl get svc frontend-service

Open in browser:

http://<EXTERNAL-IP>

`
🚀 Deploy Everything
kubectl apply -f namespace.yaml
kubectl apply -f .

`



#Ingress (Recommended Instead of LoadBalancer)

install NGINX Ingress:

kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml


Scaling the Application

Scale frontend replicas:

kubectl scale deployment frontend --replicas=3

Verify:

kubectl get pods
🛠 Troubleshooting

Check pod logs:

kubectl logs <pod-name>

Describe pod:

kubectl describe pod <pod-name>

🧹 Cleanup Resources

az group delete --name mern-aks-rg --yes --no-wait

🎯 Outcome

Successfully deployed MERN microservices on AKS

Used ACR for container registry

Exposed frontend via LoadBalancer

Verified scaling and monitoring
