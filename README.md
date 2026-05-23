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
├── screenshots/             # Project screenshots
├── Dockerfile               # Nginx serving dist/ on port 3000
├── nginx.conf               # Nginx config (SPA routing + gzip)
├── buildspec.yml            # AWS CodeBuild pipeline spec
├── .gitignore
└── README.md
```

---

## 🐳 Phase 1 — Docker

### What we did
- Created `Dockerfile` using `nginx:alpine` base image
- Created `nginx.conf` to serve the React app on port 3000 with SPA routing
- Built and tested the Docker image locally

### Build & Run Locally

```bash
# Build the Docker image
docker build -t brain-tasks-app:latest .

# Verify image was created
docker images | grep brain-tasks-app

# Run the container on port 3000
docker run -d -p 3000:3000 --name brain-tasks-test brain-tasks-app:latest

# Check container is running
docker ps

# View container logs
docker logs brain-tasks-test

# Stop and remove container
docker stop brain-tasks-test
docker rm brain-tasks-test
```

### ✅ Output — Docker Image Built

![Docker Image](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/docker-image.png)

### ✅ Output — Container Running

![Docker Running](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/docker-running.png)

### ✅ Output — App Running at localhost:3000

![App Preview](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/app-preview.png)

---

## 🗂️ Phase 2 — AWS ECR (Container Registry)

### What we did
- Created a private ECR repository to store Docker images
- Tagged and pushed the Docker image to ECR

```bash
# Create ECR repository
aws ecr create-repository \
  --repository-name brain-tasks-app \
  --region ap-south-1

# Authenticate Docker to ECR
aws ecr get-login-password --region ap-south-1 | \
  docker login --username AWS --password-stdin \
  675253838749.dkr.ecr.ap-south-1.amazonaws.com

# Tag image
docker tag brain-tasks-app:latest \
  675253838749.dkr.ecr.ap-south-1.amazonaws.com/brain-tasks-app:latest

# Push to ECR
docker push \
  675253838749.dkr.ecr.ap-south-1.amazonaws.com/brain-tasks-app:latest

# Verify image in ECR
aws ecr list-images --repository-name brain-tasks-app --region ap-south-1
```

### ✅ Output — ECR Login Successful

![ECR Login](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28641%29.png)

### ✅ Output — ECR Repository Created

![ECR Repository](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28642%29.png)

### ✅ Output — Image Pushed to ECR

![ECR Push](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28645%29.png)

### ✅ Output — Image Listed in ECR

![ECR Image](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28644%29.png)

---

## ☸️ Phase 3 — AWS EKS (Kubernetes)

### What we did
- Created an EKS cluster with 2 t3.micro worker nodes
- Wrote `deployment.yaml` (2 replicas, health checks, resource limits)
- Wrote `service.yaml` (LoadBalancer exposing port 80 → 3000)
- Deployed the app using `kubectl`

```bash
# Create EKS cluster
eksctl create cluster \
  --name brain-tasks-cluster \
  --region ap-south-1 \
  --nodegroup-name brain-tasks-nodes \
  --node-type t3.micro \
  --nodes 2 \
  --managed

# Configure kubectl
aws eks update-kubeconfig --region ap-south-1 --name brain-tasks-cluster

# Verify nodes are ready
kubectl get nodes

# Deploy to EKS
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml

# Check pods
kubectl get pods

# Get public URL
kubectl get svc brain-tasks-app-service
```

### ✅ Output — EKS Nodes Running

![EKS Nodes](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28646%29.png)

### ✅ Output — Pods Running

![Pods Running](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28647%29.png)

### ✅ Output — Service & LoadBalancer

![Service](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28648%29.png)

### ✅ Output — App Running via EKS LoadBalancer

![App on EKS](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28649%29.png)

---

## 🔧 Phase 4 — AWS CodeBuild

### What we did
- Created `buildspec.yml` with 4 phases: install, pre_build, build, post_build
- CodeBuild installs kubectl, logs into ECR, builds Docker image, pushes to ECR, deploys to EKS
- Logs streamed to CloudWatch

```bash
# Store AWS Account ID in SSM
aws ssm put-parameter \
  --name "/codebuild/aws-account-id" \
  --value "<YOUR_ACCOUNT_ID>" \
  --type "String"

# Trigger a manual build
aws codebuild start-build \
  --project-name brain-tasks-app-build \
  --region ap-south-1
```

### ✅ Output — CodeBuild

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28650%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28651%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28652%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28656%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28657%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28658%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28659%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28661%29.png)

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28662%29.png)

---

## 📦 Version Control — GitHub

### ✅ Output — GitHub Repository

![](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/Screenshot%20%28663%29.png)

---

## 🚀 Phase 5 — AWS CodePipeline

### What we did
- Created a 3-stage pipeline: Source → Build → Deploy
- Pipeline triggers automatically on every `git push` to `main`

| Stage | Tool | Action |
|-------|------|--------|
| Source | GitHub | Detects push to `main` branch |
| Build | AWS CodeBuild | Builds Docker image, pushes to ECR |
| Deploy | kubectl (in CodeBuild) | Applies K8s manifests to EKS |

### ✅ Output — Pipeline Succeeded

![CodePipeline](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/codepipeline.png)

---

## 📊 Phase 6 — Monitoring (CloudWatch)

### What we did
- Enabled CloudWatch Logs in CodeBuild project
- Log group: `/codebuild/brain-tasks-app`
- Monitored pod logs via `kubectl`

```bash
# Stream CodeBuild logs live
aws logs tail /codebuild/brain-tasks-app --follow

# View logs for a specific pod
kubectl logs -f <POD_NAME>

# View logs for all pods
kubectl logs -l app=brain-tasks-app --all-containers
```

### ✅ Output — CloudWatch Logs

![CloudWatch Logs](https://raw.githubusercontent.com/subashree06/brain-tasks-app/main/screenshots/cloudwatch.png)

---

## ⚙️ Environment Variables

| Variable | Value |
|----------|-------|
| `AWS_REGION` | `ap-south-1` |
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
| CloudWatch Logs | 5 GB free |
| EKS Control Plane | ❌ $0.10/hr — delete when not in use |
| EC2 t3.micro nodes | 750 hrs/month free |

```bash
# Delete EKS cluster to stop charges
eksctl delete cluster --name brain-tasks-cluster --region ap-south-1
```

---

## 👤 Author

**Subashree** — Deployed as part of a DevOps learning project covering Docker, ECR, EKS, CodeBuild, CodePipeline, and CloudWatch.
