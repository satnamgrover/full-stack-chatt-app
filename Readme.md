# 💬 Full Stack Real-Time Chat Application — Containerized, Orchestrated & Automated

![React](https://img.shields.io/badge/React-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Spring Boot](https://img.shields.io/badge/Spring_Boot-6DB33F?style=for-the-badge&logo=springboot&logoColor=white)
![WebSocket](https://img.shields.io/badge/WebSocket-010101?style=for-the-badge&logo=socketdotio&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)
![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)
![Helm](https://img.shields.io/badge/Helm-0F1689?style=for-the-badge&logo=helm&logoColor=white)
![Jenkins](https://img.shields.io/badge/Jenkins-D24939?style=for-the-badge&logo=jenkins&logoColor=white)

> ⚠️ **Fork Notice:** This project is forked from [Original Author's GitHub Repo Link].
> All credit for the original full-stack chat application (React + Spring Boot + WebSocket) goes to the original author.
> I have used this project to apply and demonstrate real-world DevOps practices including containerization, Kubernetes orchestration, Helm packaging, and CI/CD automation.

---

## 📌 What I Built On Top

| Layer | What I Added |
|---|---|
| 🐳 Docker | Containerized frontend, backend & database individually |
| 🐙 Docker Compose | Multi-container local development setup |
| ☸️ Kubernetes | Deployments, Services, StatefulSet, Namespace, PV/PVC |
| ⎈ Helm | Packaged full K8s setup as a reusable Helm Chart |
| ⚙️ Jenkins | End-to-end CI/CD pipeline for all services |

---

## 🏗️ Architecture Overview

```
Developer Push (GitHub)
        │
        ▼
  Jenkins CI/CD Pipeline
  ┌──────────────────────────────────────────┐
  │  1. Checkout Code                        │
  │  2. Build Docker Images                  │
  │     ├── Frontend  (React)                │
  │     ├── Backend   (Spring Boot)          │
  │     └── Database  (MongoDB/MySQL/etc.)   │
  │  3. Push Images to Registry              │
  │  4. Deploy via Helm Chart to Kubernetes  │
  └──────────────────────────────────────────┘
        │
        ▼
  Kubernetes Cluster
  ┌──────────────────────────────────────────┐
  │  Namespace: chat-app                     │
  │                                          │
  │  ┌─────────────┐  ┌─────────────┐        │
  │  │  Frontend   │  │  Backend    │        │
  │  │ Deployment  │  │ Deployment  │        │
  │  │  + Service  │  │  + Service  │        │
  │  └─────────────┘  └─────────────┘        │
  │                                          │
  │  ┌─────────────────────────┐             │
  │  │  Database StatefulSet   │             │
  │  │  + Service + PV + PVC   │             │
  │  └─────────────────────────┘             │
  └──────────────────────────────────────────┘
```

---

## 🛠️ Tech Stack

| Technology | Purpose |
|---|---|
| React | Frontend UI |
| Spring Boot (Java) | Backend REST API |
| WebSocket | Real-time bidirectional communication |
| Docker | Containerization of each service |
| Docker Compose | Local multi-service orchestration |
| Kubernetes | Container orchestration in cluster |
| Helm | Kubernetes package manager — templated deployments |
| Jenkins | CI/CD pipeline automation |

---

## 📁 Project Structure

```
├── frontend/
│   ├── src/                    # React source code
│   └── Dockerfile              # Frontend container
│
├── backend/
│   ├── src/                    # Spring Boot source code
│   └── Dockerfile              # Backend container
│
├── docker-compose.yml          # Local multi-container setup
│
├── k8s/
│   ├── namespace.yaml          # Isolated namespace
│   ├── frontend/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   ├── backend/
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── database/
│       ├── statefulset.yaml
│       ├── service.yaml
│       ├── pv.yaml
│       └── pvc.yaml
│
├── helm/
│   └── chat-app/
│       ├── Chart.yaml
│       ├── values.yaml         # Configurable values
│       └── templates/
│           ├── frontend-deployment.yaml
│           ├── frontend-service.yaml
│           ├── backend-deployment.yaml
│           ├── backend-service.yaml
│           ├── db-statefulset.yaml
│           ├── db-service.yaml
│           ├── pv.yaml
│           └── pvc.yaml
│
└── Jenkinsfile                 # CI/CD pipeline definition
```

---

## 🐳 Docker & Docker Compose

### Build individual images
```bash
# Frontend
docker build -t chat-frontend ./frontend

# Backend
docker build -t chat-backend ./backend
```

### Run locally with Docker Compose
```bash
docker-compose up --build
```

Access the app at `http://localhost:3000`

---

## ☸️ Kubernetes Setup

### Prerequisites
- A running Kubernetes cluster (local: Minikube / Kind, or cloud: EKS / GKE)
- `kubectl` configured and connected to your cluster

### Deploy with raw manifests

```bash
# Create namespace
kubectl apply -f k8s/namespace.yaml

# Deploy database (StatefulSet + PV + PVC)
kubectl apply -f k8s/database/

# Deploy backend
kubectl apply -f k8s/backend/

# Deploy frontend
kubectl apply -f k8s/frontend/

# Verify everything is running
kubectl get all -n chat-app
kubectl get pv,pvc -n chat-app
```

---

## ⎈ Helm Deployment

### Install the chart
```bash
helm install chat-app ./helm/chat-app \
  --namespace chat-app \
  --create-namespace
```

### Upgrade after changes
```bash
helm upgrade chat-app ./helm/chat-app --namespace chat-app
```

### Uninstall
```bash
helm uninstall chat-app --namespace chat-app
```

### Override values for different environments
```bash
# For production
helm install chat-app ./helm/chat-app \
  --namespace chat-app \
  -f helm/chat-app/values-prod.yaml
```

---

## ⚙️ Jenkins CI/CD Pipeline

The `Jenkinsfile` automates the following stages:

```
┌─────────────────────────────────────────────────┐
│  Stage 1 → Checkout Code from GitHub            │
│  Stage 2 → Build Docker Images (all services)   │
│  Stage 3 → Push Images to Registry              │
│  Stage 4 → Deploy using Helm Chart to K8s       │
└─────────────────────────────────────────────────┘
```

### Jenkinsfile snippet
```groovy
pipeline {
    agent any
    environment {
        IMAGE_FRONTEND = "your-registry/chat-frontend"
        IMAGE_BACKEND  = "your-registry/chat-backend"
        NAMESPACE      = "chat-app"
    }
    stages {
        stage('Checkout')         { steps { checkout scm } }
        stage('Build Images')     { steps { sh 'docker-compose build' } }
        stage('Push Images')      { steps { /* push to registry */ } }
        stage('Deploy with Helm') {
            steps {
                sh 'helm upgrade --install chat-app ./helm/chat-app --namespace ${NAMESPACE} --create-namespace'
            }
        }
    }
}
```

---

## 🗄️ Persistent Storage

The database uses **Persistent Volume (PV)** and **Persistent Volume Claim (PVC)** to ensure data is not lost when the database pod restarts.

```
PersistentVolume (PV)        ← Actual storage on the node/cloud
        │
PersistentVolumeClaim (PVC)  ← Request for storage by the pod
        │
StatefulSet (Database Pod)   ← Uses the PVC to read/write data
```

---

## 🌐 Namespace Isolation

All resources are deployed under a dedicated namespace:

```bash
kubectl get all -n chat-app
```

This keeps the chat app resources isolated from other workloads on the cluster.

---

## 🎯 Key DevOps Concepts Demonstrated

- ✅ Multi-service containerization with Docker
- ✅ Local orchestration with Docker Compose
- ✅ Kubernetes Deployments for stateless services
- ✅ Kubernetes StatefulSet for stateful database workloads
- ✅ Persistent Volumes for durable data storage
- ✅ Namespace-based environment isolation
- ✅ Helm Chart for templated, reusable K8s deployments
- ✅ Jenkins CI/CD for end-to-end automated delivery

---

## 📸 Screenshots

> *(Add screenshots here: Jenkins pipeline, Helm install output, kubectl get all -n chat-app, running app)*

---

## 👤 About Me

**Your Name**
- 🔗 LinkedIn: [linkedin.com/in/your-profile](https://linkedin.com/in/satnamgrover)
- 🐙 GitHub: [github.com/your-username](https://github.com/satnamgrover)
