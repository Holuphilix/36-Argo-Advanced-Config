# **Advanced Configuration Management in ArgoCD**

## **Project Overview**

This project provides an in-depth exploration of advanced configuration management in ArgoCD, focusing on tools like **Helm** and **Kustomize**, as well as secrets management and custom resource synchronization policies. Learners will gain practical experience in deploying and managing Kubernetes applications using GitOps practices.

## **Why is this Project Relevant**

In modern DevOps workflows, managing application configurations, secrets, and deployment policies efficiently is critical. This project enables learners to understand best practices for secure and automated deployments in Kubernetes environments, which is highly valuable for DevOps engineers and cloud-native practitioners.

## **Project Goals and Objectives**

* Master **Helm** and **Kustomize** integration with ArgoCD for configuration management.
* Implement **secure secrets management** using Kubernetes secrets and external secret managers like HashiCorp Vault and AWS Secrets Manager.
* Customize **resource management** and synchronization policies in ArgoCD to optimize deployments.
* Understand **automated vs. manual sync** strategies and how to implement self-healing and pruning policies.

## **Prerequisites**

* Basic understanding of **Kubernetes** and **GitOps** principles.
* Familiarity with **ArgoCD**, **Helm**, and **Kustomize**.
* Access to a Kubernetes cluster and ArgoCD installation.
* Basic knowledge of secrets management tools (e.g., Vault, AWS Secrets Manager).

## **Project Deliverables**

* A fully structured project directory containing all configurations, scripts, and supporting files.
* Helm charts and Kustomize overlays for multiple environments (dev, staging, prod).
* Examples of Kubernetes secrets and integrations with external secret managers.
* Custom resource health checks and sync policy configurations in ArgoCD.
* README.md documenting all steps, procedures, and best practices.

## **Tools & Technologies Used**

* **Kubernetes**
* **ArgoCD**
* **Helm**
* **Kustomize**
* **HashiCorp Vault**
* **AWS Secrets Manager**
* **Git**
* **Kubectl**
* **Lua (for custom health checks)**

## **Project Components**

1. **Helm Chart Integration** – Manage application templates and deployment configurations.
2. **Kustomize Overlays** – Customize deployments for multiple environments.
3. **Secrets Management** – Secure storage and access of sensitive data.
4. **Resource Management & Sync Policies** – Customize resource handling, automated sync, pruning, and self-healing.


## Task 1: Project Structure Setup 

### **Objective:** 

Create the project directory with all necessary subdirectories, files, and supporting resources for the ArgoCD advanced configuration management project.

### **Steps:**

1. Create the root project directory:

```bash
mkdir argo-advanced-config
cd argo-advanced-config
```

2. Create the essential subdirectories:

```bash
mkdir -p helm-charts/my-app/templates
mkdir -p kustomize/base
mkdir -p kustomize/overlays/dev
mkdir -p kustomize/overlays/staging
mkdir -p kustomize/overlays/prod
mkdir -p secrets
mkdir -p images
```

3. Create placeholder files:

```bash
touch README.md
# Helm chart files
touch helm-charts/my-app/Chart.yaml
touch helm-charts/my-app/values.yaml
touch helm-charts/my-app/templates/deployment.yaml
touch helm-charts/my-app/templates/service.yaml
touch helm-charts/my-app/templates/ingress.yaml

# Kustomize base files
touch kustomize/base/kustomization.yaml
touch kustomize/base/deployment.yaml
touch kustomize/base/service.yaml

# Kustomize overlay files
touch kustomize/overlays/dev/kustomization.yaml
touch kustomize/overlays/dev/patch.yaml
touch kustomize/overlays/staging/kustomization.yaml
touch kustomize/overlays/staging/patch.yaml
touch kustomize/overlays/prod/kustomization.yaml
touch kustomize/overlays/prod/patch.yaml
```

### **Project Directory Structure:**

```
argo-advanced-config/
├── README.md
├── helm-charts/
│   └── my-app/
│       ├── Chart.yaml
│       ├── values.yaml
│       └── templates/
│           ├── deployment.yaml
│           ├── service.yaml
│           └── ingress.yaml
├── kustomize/
│   ├── base/
│   │   ├── kustomization.yaml
│   │   ├── deployment.yaml
│   │   └── service.yaml
│   └── overlays/
│       ├── dev/
│       │   ├── kustomization.yaml
│       │   └── patch.yaml
│       ├── staging/
│       │   ├── kustomization.yaml
│       │   └── patch.yaml
│       └── prod/
│           ├── kustomization.yaml
│           └── patch.yaml
├── secrets/
├── images/
├── index.js
├── package.json
└── Dockerfile
```

### **Notes:**

* `kustomize/base/` → Contains base Deployment and Service definitions.
* `kustomize/overlays/dev/` → Dev-specific configuration: 1 replica, `dev` image tag.
* `kustomize/overlays/staging/` → Staging-specific configuration: 2 replicas, `staging` image tag.
* `kustomize/overlays/prod/` → Prod-specific configuration: 3 replicas, `latest` image tag.
* `secrets/` → Reserved for Kubernetes secrets or external secret references (used in later tasks).
* `images/` → Optional folder for diagrams or screenshots.
* `helm-charts/` → Helm chart templates for your application.
* `index.js` / `package.json` / `Dockerfile` → Node.js application and Docker configuration.

## **Task 2: Helm Chart Creation and ArgoCD Integration**

### **Objective:**

Create a Helm chart for the application, start a local Kubernetes cluster (Minikube), install ArgoCD, and integrate it to enable GitOps-based deployments.

### **Requirements / Preparations**

Before starting Task 2, ensure the following are set up:

1. **GitHub Repository**

   * Repository Name: `argo-advanced-config`
   * URL: https://github.com/Holuphilix/argo-advanced-config.git
   * Contains the project structure created in Task 1.
   * Ensure Helm chart placeholders exist under `helm-charts/my-app/`.

2. **Docker Hub Repository**

   * Repository: `holuphilix/my-k8s-app`
   * Push your application Docker image to Docker Hub.

3. **Helm Installed**

   * Install Helm CLI locally: [Install Helm](https://helm.sh/docs/intro/install/)
   * Verify installation:

```bash
helm version
```

### **Steps**

#### **Step 1: Start Minikube**

```bash
minikube start --driver=docker
```

* Verify nodes:

```bash
kubectl get nodes
```

#### **Step 2: Install ArgoCD in the Cluster**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

* Check ArgoCD pods:

```bash
kubectl get pods -n argocd
```

#### **Step 3: Access ArgoCD UI**

* Port-forward ArgoCD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

* Open in browser: [https://localhost:8080](https://localhost:8080)
* Get initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

#### **Step 4: Set Up Helm Chart**

```bash
cd helm-charts/my-app
```

* Edit `values.yaml`:

```yaml
replicaCount: 2

image:
  repository: holuphilix/my-k8s-app
  tag: latest
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
```

* Edit templates in `templates/`:

```
deployment.yaml → Deployment configuration
service.yaml    → Service configuration
ingress.yaml    → Ingress rules (optional)
```

* Edit `deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    app.kubernetes.io/name: {{ include "my-app.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app.kubernetes.io/name: {{ include "my-app.name" . }}
      app.kubernetes.io/instance: {{ .Release.Name }}
  template:
    metadata:
      labels:
        app.kubernetes.io/name: {{ include "my-app.name" . }}
        app.kubernetes.io/instance: {{ .Release.Name }}
    spec:
      containers:
        - name: {{ .Chart.Name }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          imagePullPolicy: "{{ .Values.image.pullPolicy }}"
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
```

* Edit `service.yaml`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    app: {{ include "my-app.name" . }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app: {{ include "my-app.name" . }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 80
```

* Edit `ingress.yaml`:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "my-app.fullname" . }}
  labels:
    {{- include "my-app.labels" . | nindent 4 }}
  annotations:
    kubernetes.io/ingress.class: nginx
spec:
  rules:
    - host: my-app.local
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: {{ include "my-app.fullname" . }}
                port:
                  number: {{ .Values.service.port }}
```

### **Step 6: Create Minimal Node.js App**

Create `index.js`:

```javascript
const http = require('http');
const PORT = 80;
const server = http.createServer((req, res) => {
  res.writeHead(200, {'Content-Type': 'text/plain'});
  res.end('Hello from ArgoCD Helm Deployment!\n');
});
server.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

Create `package.json`:

```json
{
  "name": "my-k8s-app",
  "version": "1.0.0",
  "main": "index.js",
  "scripts": { "start": "node index.js" },
  "dependencies": {}
}
```

### **Step 6: Create a Dockerfile**

Create a file called `Dockerfile` in your project root:

```dockerfile
# Use Node.js LTS base image
FROM node:18-alpine

# Set working directory
WORKDIR /app

# Copy package files
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy app source
COPY . .

# Expose port
EXPOSE 80

# Start the application
CMD ["node", "index.js"]
```

### **Step 7: Build the image**

```bash
docker build -t holuphilix/my-k8s-app:latest .
```

### **Step 8: Push to Docker Hub**

```bash
docker login
docker push holuphilix/my-k8s-app:latest
```

### **Step 9: Commit Helm Chart to Git**

```bash
git init
git add .
git commit -m "Add Helm chart for my-app with Docker Hub image"
git remote add origin https://github.com/Holuphilix/argo-advanced-config.git
git push origin main
```

### **Step 10: Create ArgoCD Application**

* **Using CLI:**

```bash
# Log in to ArgoCD
argocd login localhost:8080 --username admin --password 4I7qZw4rJ4kFrqPQ --insecure

# Create the ArgoCD app with NodePort service
argocd app create my-app \
  --repo https://github.com/Holuphilix/argo-advanced-config.git \
  --path helm-charts/my-app \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set image.repository=holuphilix/my-k8s-app \
  --helm-set image.tag=latest \
  --helm-set service.type=NodePort \
  --helm-set service.port=80 \
  --helm-set service.nodePort=30080
```

* **Using UI:**

  * Log in → Click **New App** → Fill repository URL and path → Set cluster & namespace → Enable automated sync.

### **Step 11: Sync and Verify**

* Sync application via CLI:

```bash
argocd app sync my-app
```

* Verify resources:

```bash
argocd app get my-app
kubectl get pods -n default
kubectl get svc -n default
```
### **Expected Outcome**

* Minikube cluster running with ArgoCD installed.
* Functional Helm chart for the application.
* ArgoCD application tracking the GitHub repository.
* Application deployed automatically to Kubernetes, pulling images from Docker Hub.

### **Notes**

* Ensure Docker Hub image `holuphilix/my-k8s-app:latest` is accessible from the cluster.
* Use environment-specific `values.yaml` for dev/staging/prod.
* This task sets up the foundation for integrating **secrets** and **Kustomize overlays** in later tasks.
* Docker Hub image `holuphilix/my-k8s-app:latest` is referenced in `values.yaml`.
* Deployment, Service, and optional Ingress templates are ready.
* You can now push these templates to GitHub and sync via ArgoCD.

Ah! I see exactly what you mean now — you want **Task 3 fully rewritten to match the style, numbering, alignment, and formatting of your Task 1** you pasted earlier. That means:

* **Numbered “Task 3” header like Task 1**
* Steps numbered sequentially (1, 2, 3…)
* Proper “### **Step X: …**” headers
* Code blocks directly under each step
* Notes at the bottom using the same indentation/alignment style

## Task 3: Kustomize Overlays for Dev, Staging, Prod

### **Objective:**

Implement environment-specific Kustomize overlays for dev, staging, and prod deployments, enabling ArgoCD automated syncing, self-healing, and pruning.

### **Steps:**

1. **Create the Kustomize Base**

**File:** `kustomize/base/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml

namePrefix: my-app-
labels:
  app: my-app
```

**File:** `kustomize/base/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:latest
          ports:
            - containerPort: 80
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
```

**File:** `kustomize/base/service.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app
  labels:
    app: my-app
spec:
  type: ClusterIP
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
```

---

2. **Create Dev Overlay**

**File:** `kustomize/overlays/dev/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -dev

patches:
  - path: patch.yaml
    target:
      kind: Deployment
      name: my-app
```

**File:** `kustomize/overlays/dev/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:dev
```

3. **Create Staging Overlay**

**File:** `kustomize/overlays/staging/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -staging

patches:
  - path: patch.yaml
    target:
      kind: Deployment
      name: my-app
```

**File:** `kustomize/overlays/staging/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 2
  template:
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:staging
```

4. **Create Prod Overlay**

**File:** `kustomize/overlays/prod/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - ../../base

nameSuffix: -prod

patches:
  - path: patch.yaml
    target:
      kind: Deployment
      name: my-app
```

**File:** `kustomize/overlays/prod/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3
  template:
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:latest
```

5. **Build locally and tag for each environment**

```bash
# Build the base image
docker build -t holuphilix/my-k8s-app:latest .

# Tag images for each environment
docker tag holuphilix/my-k8s-app:latest holuphilix/my-k8s-app:dev
docker tag holuphilix/my-k8s-app:latest holuphilix/my-k8s-app:staging
docker tag holuphilix/my-k8s-app:latest holuphilix/my-k8s-app:prod
```

6. **Push to Docker Hub**

```bash
# Push all images to Docker Hub
docker push holuphilix/my-k8s-app:latest
docker push holuphilix/my-k8s-app:dev
docker push holuphilix/my-k8s-app:staging
docker push holuphilix/my-k8s-app:prod
```

7. **Test Kustomize Locally**

```bash
# Dev
kubectl apply -k kustomize/overlays/dev

# Staging
kubectl apply -k kustomize/overlays/staging

# Prod
kubectl apply -k kustomize/overlays/prod

# Verify
kubectl get deployments
kubectl get svc
```

8. **Commit and push** the changes to your GitHub repository:

```bash
git add .
git commit -m "Add kustomization files for base and overlays"
git push origin main
```

9. **Integrate Kustomize with ArgoCD**

```bash
# Dev - automated sync, self-heal & prune
argocd app create my-app-dev \
  --repo https://github.com/Holuphilix/argo-advanced-config.git \
  --path kustomize/overlays/dev \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# Staging - automated sync, self-heal & prune
# Staging - automated sync, self-heal & prune
argocd app create my-app-staging \
  --repo https://github.com/Holuphilix/argo-advanced-config.git \
  --path kustomize/overlays/staging \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy automated \
  --auto-prune \
  --self-heal

# Prod - manual sync
argocd app create my-app-prod \
  --repo https://github.com/Holuphilix/argo-advanced-config.git \
  --path kustomize/overlays/prod \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --sync-policy manual
```

10. **Sync Applications**

```bash
# Dev & Staging auto-sync
argocd app sync my-app-dev
argocd app sync my-app-staging

# Prod manual sync
argocd app sync my-app-prod

# Verify
argocd app get my-app-dev
argocd app get my-app-staging
argocd app get my-app-prod
```

### **Notes:**

* Dev deployment: **1 replica**, image tag `dev`
* Staging deployment: **2 replicas**, image tag `staging`
* Prod deployment: **3 replicas**, image tag `latest`
* Dev and Staging apps have **auto-sync, self-heal, and pruning** enabled
* Prod app is **manual sync** for controlled deployment
* `kubectl` commands confirm deployments and services applied correctly
* ArgoCD dashboard shows **healthy synced applications** and automatic drift recovery for dev & staging
