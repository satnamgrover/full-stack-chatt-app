# FULL STACK CHAT APP - CHATTINGO

A full-stack web application (React frontend + Spring Boot backend + database) forked and used as a hands-on playground for practicing DevOps workflows — containerization, orchestration, and CI/CD — in a setup that mirrors a real production/development environment.

> This repository is a fork of ([<ORIGINAL_REPO_URL>](iemafzalhassan/chattingo)). The application logic itself is unchanged; the focus of this fork is the DevOps tooling layered on top of it.

---

## Tech Stack

| Layer        | Technology                          |
|--------------|--------------------------------------|
| Frontend     | React                                |
| Backend      | Spring Boot (Java)                  |
| Database     | MySQL                               |
| Containers   | Docker, Docker Compose              |
| Orchestration| Kubernetes (Minikube/Kind)          |
| CI/CD        | Jenkins                             |
| Registry     | Docker Hub                          |

---

## Project Structure

```
.
├── frontend/                 # React application
│   └── Dockerfile
├── backend/                  # Spring Boot application
│   └── Dockerfile
├── docker-compose.yml         # Multi-container local setup
├── k8s/
│   ├── namespace.yaml
│   ├── frontend.yaml          # Deployment + Service
│   ├── backend.yaml           # Deployment + Service
│   └── database.yaml          # StatefulSet + Service + PV + PVC
├── Jenkinsfile
└── README.md
```

> Adjust paths above to match your actual repo layout if it differs.

---

## 1. Containerization with Docker

Each service (frontend, backend, database) has been containerized independently with its own `Dockerfile`.

- **Frontend** – runs the React app on its default development port (3000).
- **Backend** – uses a Maven base image to build and run the Spring Boot application.
- **Database** – uses the official `MySQL` image with a mounted volume for data persistence.

### Build images individually

```bash
# Frontend
docker build -t <DOCKERHUB_USERNAME>/<PROJECT_NAME>-frontend:latest ./frontend

# Backend
docker build -t <DOCKERHUB_USERNAME>/<PROJECT_NAME>-backend:latest ./backend
```

### Run the full stack with Docker Compose

```bash
docker-compose up --build
```

This spins up the frontend, backend, and database as separate containers, networked together, simulating a local development environment.

```bash
docker-compose down -v   # tear down, including volumes
```

---

## 2. Kubernetes Deployment

The Docker Compose setup was migrated to Kubernetes to simulate a production-grade orchestration environment.

### Namespace

A dedicated namespace isolates all resources for this project from the rest of the cluster.

```bash
kubectl apply -f k8s/namespace.yaml
```

### Frontend & Backend — Deployment + Service

The frontend and backend are stateless, so they're managed via standard `Deployment` + `Service` objects, allowing easy scaling and rolling updates.

```bash
kubectl apply -f k8s/frontend.yaml

kubectl apply -f k8s/backend.yaml
```

### Database — StatefulSet + Service + PV/PVC

The database is stateful, so it's managed via a `StatefulSet` with stable network identity, paired with a headless `Service`. Persistent storage is provisioned through `PersistentVolume` (PV) and `PersistentVolumeClaim` (PVC) so data survives pod restarts/rescheduling.

```bash
kubectl apply -f k8s/database.yaml
```

### Verify the deployment

```bash
kubectl get all -n <NAMESPACE_NAME>
kubectl get pv,pvc -n <NAMESPACE_NAME>
```

### Local cluster access (Minikube/Kind)

Since this is running on a local cluster, services are exposed via `NodePort` / `minikube service` (adjust based on your actual Service type):

```bash
minikube service <FRONTEND_SERVICE_NAME> -n <NAMESPACE_NAME>
```

---

## 3. CI/CD with Jenkins

A Jenkins pipeline automates building, pushing, and deploying the application on every change, removing the need for manual Docker builds and `kubectl apply` commands.

### Pipeline stages

1. **Checkout** – pulls the latest code from the repository.
2. **Build** – builds Docker images for frontend and backend.
3. **Push** – pushes the built images to Docker Hub.
4. **Deploy** – applies the updated Kubernetes manifests to the cluster (`kubectl apply`), triggering a rolling update of the Deployments.

### Prerequisites for the Jenkins pipeline

- Jenkins with Docker installed/accessible on the agent.
- `kubectl` configured on the Jenkins agent with access to the Minikube/Kind cluster.
- Docker Hub credentials stored in Jenkins as a credential (`dockerhub-creds` in the example above).

---

## Local Setup — Quick Start

```bash
# 1. Clone the repo
git clone <REPO_URL>
cd <PROJECT_NAME>

# 2. Run with Docker Compose (simplest local dev setup)
docker-compose up --build

# OR

# 3. Deploy to local Kubernetes (Minikube/Kind)
minikube start
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/ -n <NAMESPACE_NAME>

# 4. Trigger Jenkins pipeline (manually or via webhook) for full CI/CD flow
```

---

## What This Project Demonstrates

- Containerizing a multi-service application (frontend, backend, database) with Docker.
- Local multi-container orchestration with Docker Compose.
- Kubernetes fundamentals: Namespaces, Deployments, Services, StatefulSets, PV/PVC.
- Separating stateless workloads (frontend/backend) from stateful workloads (database) and choosing the right Kubernetes objects for each.
- Building a CI/CD pipeline with Jenkins to automate image builds, registry pushes, and cluster deployments.

---

## Future Improvements (optional section — fill in as you go)

- [ ] Add Helm charts to templatize the Kubernetes manifests.
- [ ] Add Ingress + TLS instead of NodePort access.
- [ ] Add HPA (Horizontal Pod Autoscaler) for frontend/backend.
- [ ] Add monitoring (Prometheus + Grafana).
- [ ] Move from local Minikube/Kind to a managed cloud Kubernetes cluster (EKS/GKE/AKS).
- [ ] Add automated tests as a pipeline stage before build/deploy.

---

## License

This project is based on a forked open-source repository. Refer to the original repository's license for terms governing the application code. DevOps configuration files (Docker, Kubernetes manifests, Jenkinsfile) added in this fork are free to use/reference.
