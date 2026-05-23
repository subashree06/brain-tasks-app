# 🧠 Brain Tasks App — DevOps CI/CD Pipeline

A production-ready deployment of the Brain Tasks React application using Docker, AWS ECR, EKS, CodeBuild, and CodePipeline.

---

## 📌 Project Overview

- **Application:** Brain Tasks — React-based task management app
- **Goal:** Deploy application in a production-ready environment using DevOps tools
- **App Port:** 3000
- **Deployment Platform:** AWS EKS (Kubernetes)

---

## 🏗️ Architecture

```
Developer (VS Code)
        │
        ▼
   GitHub Repo
        │
        ▼
 AWS CodePipeline
        │
        ├──▶ AWS CodeBuild
        │         │
        │         ├── Docker Build
        │         └── Push → AWS ECR
        │
        └──▶ EKS Deploy (kubectl)
                  │
                  ▼
          EKS Cluster (2 pods)
                  │
                  ▼
        LoadBalancer Service (port 80)
                  │
                  ▼
              Users 🌐
```

---

## 📁 Repository Structure

```
Brain-Tasks-App/
├── dist/                    # Pre-built React application
│   ├── index.html
│   └── assets/
│       ├── index.js
│       └── index.css
├── k8s/
│   ├── deployment.yaml      # Kubernetes Deployment (2 replicas)
│   └── service.yaml         # LoadBalancer Service
├── Dockerfile               # Nginx serving dist/ on port 3000
├── nginx.conf               # Nginx config (SPA routing + gzip)
├── buildspec.yml            # AWS CodeBuild pipeline spec
├── .gitignore
└── README.md
```

---

## 🐳 Docker Setup

### Dockerfile
Uses `nginx:alpine` to serve the pre-built React app on port 3000.

### Build & Run Locally

```bash
# Build the Docker image
docker build -t brain-tasks-app:latest .

# Run the container
docker run -d -p 3000:3000 --name brain-tasks-test brain-tasks-app:latest

# Open in browser
http://localhost:3000

# Stop the container
docker stop brain-tasks-test
docker rm brain-tasks-test
```

### Verify image
```bash
docker images | grep brain-tasks-app
```

---

## 🗂️ AWS ECR — Container Registry

```bash
# Create ECR repository
aws ecr create-repository --repository-name brain-tasks-app --region us-east-1

# Authenticate Docker to ECR
aws ecr get-login-password --region us-east-1 | \
  docker login --username AWS --password-stdin <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com

# Tag and push image
docker tag brain-tasks-app:latest <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/brain-tasks-app:latest
docker push <ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/brain-tasks-app:latest
```

---

## ☸️ Kubernetes — AWS EKS

### Create EKS Cluster
```bash
eksctl create cluster \
  --name brain-tasks-cluster \
  --region us-east-1 \
  --nodegroup-name brain-tasks-nodes \
  --node-type t3.micro \
  --nodes 2 \
  --managed
```

### Deploy to EKS
```bash
# Configure kubectl
aws eks update-kubeconfig --region us-east-1 --name brain-tasks-cluster

# Apply manifests
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Check pods
kubectl get pods

# Get app URL
kubectl get svc brain-tasks-app-service
```

### Kubernetes Files

| File | Purpose |
|------|---------|
| `k8s/deployment.yaml` | 2 replicas, rolling updates, health checks |
| `k8s/service.yaml` | AWS LoadBalancer, port 80 → 3000 |

---

## 🔧 AWS CodeBuild

The `buildspec.yml` automates:
1. Install `kubectl`
2. Login to ECR
3. Build & tag Docker image
4. Push image to ECR
5. Deploy to EKS via `kubectl`
6. Stream logs to CloudWatch

### Create SSM Parameter
```bash
aws ssm put-parameter \
  --name "/codebuild/aws-account-id" \
  --value "<YOUR_ACCOUNT_ID>" \
  --type "String"
```

---

## 🚀 AWS CodePipeline

Fully automated pipeline triggered on every `git push`:

| Stage | Tool | Action |
|-------|------|--------|
| Source | GitHub | Detects push to `main` branch |
| Build | AWS CodeBuild | Builds Docker image, pushes to ECR |
| Deploy | kubectl (in CodeBuild) | Applies K8s manifests to EKS |

---

## 📊 Monitoring — CloudWatch

```bash
# Stream CodeBuild logs
aws logs tail /codebuild/brain-tasks-app --follow

# View pod logs
kubectl logs -f <POD_NAME>

# View all pod logs
kubectl logs -l app=brain-tasks-app --all-containers
```

---

## ⚙️ Environment Variables (buildspec.yml)

| Variable | Value |
|----------|-------|
| `AWS_REGION` | `us-east-1` |
| `ECR_REPO_NAME` | `brain-tasks-app` |
| `CLUSTER_NAME` | `brain-tasks-cluster` |
| `K8S_NAMESPACE` | `default` |

---

## 💡 Free Tier Notes

| Service | Free Tier |
|---------|-----------|
| ECR | 500 MB/month free |
| CodeBuild | 100 min/month free |
| CodePipeline | 1 pipeline free |
| EKS | ❌ $0.10/hr — delete when not in use |
| EC2 t3.micro | 750 hrs/month free |

```bash
# Delete EKS cluster to avoid charges
eksctl delete cluster --name brain-tasks-cluster --region us-east-1
```

---

## 🖥️ App Preview

> Brain Tasks running at `localhost:3000` via Docker + Nginx

- ✅ Task management with search
- ✅ Filter by All / Pending / Completed / Priority
- ✅ Create Task button
- ✅ Served by Nginx on port 3000 inside Docker

---

## 👤 Author

Deployed as part of a DevOps learning project covering Docker, ECR, EKS, CodeBuild, CodePipeline, and CloudWatch.
