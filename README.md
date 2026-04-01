# 🚀 Automated CI/CD & GitOps Pipeline: Docker, GitHub Actions, Kubernetes & ArgoCD

## 📖 Project Overview
This is a complete **End-to-End CI/CD and GitOps** pipeline project. The primary objective is to automate the software delivery process—whenever a developer pushes new code to GitHub, an automated pipeline builds a Docker image, pushes it to Docker Hub, automatically updates the Kubernetes manifest with the new image tag, and commits the changes. Finally, **ArgoCD** detects these changes and seamlessly deploys the new version to the Kubernetes cluster.

## 🛠️ Technologies Used
* **Containerization:** Docker, Nginx (Alpine)
* **CI/CD Pipeline:** GitHub Actions
* **Container Orchestration:** Kubernetes (Deployment & Service)
* **GitOps Tool:** ArgoCD
* **Version Control:** Git & GitHub

## 🔄 CI/CD Workflow Architecture
1. **Developer Push:** Code is pushed to the `main` branch.
2. **Continuous Integration (CI):** GitHub Actions is triggered automatically.
3. **Build & Push:** A new Docker image is built from the source code and pushed to Docker Hub using the Git SHA as the image tag.
4. **Manifest Update (GitOps):** GitHub Actions automatically updates the `k8s/deployment.yaml` file with the new image tag and commits it back to the repository using a bot account.
5. **Continuous Deployment (CD):** ArgoCD monitors the GitHub repository for changes and deploys the updated application to the Kubernetes cluster.

## 📂 Project Structure
```text
.
├── .github/
│   └── workflows/
│       └── docker-build.yml    # CI/CD Pipeline configuration
├── app/
│   ├── Dockerfile              # Docker image configuration
│   └── index.html              # Simple static web application
└── k8s/
    ├── deployment.yaml         # Kubernetes Deployment manifest
    └── service.yaml            # Kubernetes Service manifest (NodePort)
```

## ⚙️ Step-by-Step Implementation Details

### Step 1: The Application & Dockerization (`/app`)
A simple HTML webpage was created and containerized using a lightweight `nginx:alpine` base image for fast and efficient deployments.

**`app/Dockerfile`**
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
```

### Step 2: Kubernetes Manifests (`/k8s`)
A Kubernetes Deployment and Service were created to run the application within the cluster.

**`k8s/deployment.yaml`**
The `image` tag here is dynamically updated by the CI pipeline.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: app
  template:
    metadata:
      labels:
        app: app
    spec:
      containers:
        - name: app
          image: shafiur22/app:45869fa  # Auto-updated by CI
          ports:
            - containerPort: 80
```

**`k8s/service.yaml`**
A NodePort service is used to expose the application to the external network.
```yaml
apiVersion: v1
kind: Service
metadata:
  name: app-service
spec:
  selector:
    app: app
  ports:
    - port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```

### Step 3: CI/CD Pipeline via GitHub Actions
This is the core automation engine of the project. It handles everything from building the code to updating the deployment manifests.

**Key Steps in `.github/workflows/docker-build.yml`:**
* **Set Image Tag:** Generates a unique image tag using the first 7 characters of the Git commit hash (Git SHA).
* **Docker Login & Push:** Authenticates with Docker Hub using repository secrets (`DOCKER_USERNAME`, `DOCKER_TOKEN`) and pushes the newly built image.
* **Update Manifest:** Uses the `sed` command to replace the old image tag with the newly generated tag in the `deployment.yaml` file.
* **Commit to GitHub:** Automatically commits and pushes the updated manifest file back to the repository using the `github-actions[bot]`.

## 🚀 How to Test/Run
1. Fork or clone this repository.
2. Add your `DOCKER_USERNAME` and `DOCKER_TOKEN` in the GitHub Repository Secrets.
3. Make a change to the `app/index.html` file and push it to the `main` branch.
4. Navigate to the **Actions** tab in your GitHub repository to watch the pipeline automatically build, push, and update the manifest!
