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
touch index.js
touch package.json
touch Dockerfile
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

1. **Start Minikube**

```bash
minikube start --driver=docker
```

* Verify nodes:

```bash
kubectl get nodes
```

2. **Install ArgoCD in the Cluster**

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

* Check ArgoCD pods:

```bash
kubectl get pods -n argocd
```

3. **Access ArgoCD UI**

* Port-forward ArgoCD server:

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

* Open in browser: [https://localhost:8080](https://localhost:8080)
* Get initial admin password:

```bash
kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d
```

4. **Set Up Helm Chart**

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
  type: NodePort
  port: 80
  nodePort: 30080
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
    app.kubernetes.io/name: {{ include "my-app.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
spec:
  type: {{ .Values.service.type }}
  selector:
    app.kubernetes.io/name: {{ include "my-app.name" . }}
    app.kubernetes.io/instance: {{ .Release.Name }}
  ports:
    - protocol: TCP
      port: {{ .Values.service.port }}
      targetPort: 80
      nodePort: {{ .Values.service.nodePort | default 30080 }}
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

5. **Create Minimal Node.js App**

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

6. **Create a Dockerfile**

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

7. **Build the image**

```bash
docker build -t holuphilix/my-k8s-app:latest .
```

**Screenshot:** Build the Image
![Build the image](./images/2.docker_build_t_push_orign.png)

8. **Push to Docker Hub**

```bash
docker login
docker push holuphilix/my-k8s-app:latest
```

9. **Commit Helm Chart to Git**

```bash
git init
git add .
git commit -m "Add Helm chart for my-app with Docker Hub image"
git remote add origin https://github.com/Holuphilix/argo-advanced-config.git
git push origin main
```

10. **Create ArgoCD Application**

* **Using CLI:**

```bash
# Log in to ArgoCD
argocd login localhost:8080 --username admin --password YVCg06P3V9EnjQ1m --insecure

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

**Screenshot:** Create ArgoCD Application
![Create ArgoCD Application](./images/1.application_my_app_localhost.png)

* **Using UI:**

  * Log in → Click **New App** → Fill repository URL and path → Set cluster & namespace → Enable automated sync.

11. **Sync and Verify**

* Sync application via CLI:

```bash
argocd app sync my-app
```

**Screenshot:** Argocd App Sync my-app via CLI
![argocd app sync my-app](./images/4.argocd_sync_app.png)

**Screenshot:** Argocd App Sync my-app via WEB
![argocd app sync my-app](./images/5.argocd_sync_app_UIC.png)

* Verify resources:

```bash
argocd app get my-app
kubectl get pods -n default
kubectl get svc -n default
```

**Screenshot:** Verify Resources
![Verify resources](./images/6.kubectl_get_pods_svc.png)

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
          image: holuphilix/my-k8s-app:latest  # base image
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "250m"
              memory: "64Mi"
            limits:
              cpu: "500m"
              memory: "128Mi"
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
  name: my-app-my-app-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        env: dev
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:dev
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
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
  name: my-app-my-app-staging
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        env: staging
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:staging
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "200m"
              memory: "256Mi"
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
  name: my-app-my-app-prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        env: prod
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:prod
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "200m"
              memory: "128Mi"
            limits:
              cpu: "300m"
              memory: "256Mi"
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

**Screenshot 1:** Sync Applications
![Sync Applications](./images/7.argocd_sync_app_dev.png)

**Screenshot 2:** Sync Applications
![Sync Applications](./images/8.argocd_sync_app_prod_staging.png)

### **Notes:**

* **Dev deployment**: **1 replica**, image tag `dev`, environment label `env: dev`, lightweight resource requests (`cpu: 100m`, `memory: 64Mi`).
* **Staging deployment**: **1 replica**, image tag `staging`, environment label `env: staging`, higher memory allocation (`cpu: 100m`, `memory: 128Mi`).
* **Prod deployment**: **1 replica**, image tag `prod`, environment label `env: prod`, strongest resource limits (`cpu: 200–300m`, `memory: 128–256Mi`).
* All deployments use `imagePullPolicy: Always` to ensure the latest images are pulled.
* Dev and Staging apps have **auto-sync, self-heal, and pruning** enabled in ArgoCD.
* Prod app uses **manual sync** for controlled deployment.
* `kubectl` commands confirm deployments and services applied correctly in each environment.
* ArgoCD dashboard shows **healthy synced applications** with automatic drift recovery for Dev & Staging.

## **Task 4: Secrets Management and Best Practices in ArgoCD**

### **Objective:**

Securely manage sensitive data such as API keys, passwords, and tokens in your Kubernetes applications, integrating them with ArgoCD for GitOps-based deployments.

### **Steps**

1. **Create a Sealed Secret in the Base**

Sealed Secrets allow safe GitOps workflows by storing encrypted secrets in Git.

**File:** `kustomize/base/sealed-secret.yaml`

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  name: my-app-secret
spec:
  encryptedData:
    API_KEY: c29tZS1zZWNyZXQta2V5  # generated with kubeseal
```

**Update base kustomization:** `kustomize/base/kustomization.yaml`

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml
  - sealed-secret.yaml

namePrefix: my-app-
```

2. **Update Dev Overlay Deployment**

Reference the SealedSecret in the dev deployment:

**File:** `kustomize/overlays/dev/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-my-app-dev
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        env: dev
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:dev
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "64Mi"
            limits:
              cpu: "200m"
              memory: "128Mi"
          env:
            - name: API_KEY
              valueFrom:
                secretKeyRef:
                  name: my-app-secret
                  key: API_KEY
```

3. **Update Staging Overlay Deployment**

**File:** `kustomize/overlays/staging/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-my-app-staging
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        env: staging
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:staging
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "200m"
              memory: "256Mi"
          env:
            - name: API_KEY
              valueFrom:
                secretKeyRef:
                  name: my-app-secret
                  key: API_KEY
```

4. **Update Prod Overlay Deployment**

**File:** `kustomize/overlays/prod/patch.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app-my-app-prod
spec:
  replicas: 1
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
        env: prod
    spec:
      containers:
        - name: my-app
          image: holuphilix/my-k8s-app:prod
          imagePullPolicy: Always
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "200m"
              memory: "128Mi"
            limits:
              cpu: "300m"
              memory: "256Mi"
          env:
            - name: API_KEY
              valueFrom:
                secretKeyRef:
                  name: my-app-secret
                  key: API_KEY
```

5. **Install Sealed Secrets Controller**

```bash
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.26.1/controller.yaml
```

Install `kubeseal` CLI to create SealedSecrets:

```bash
curl -OL "https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.30.0/kubeseal-0.30.0-linux-amd64.tar.gz"

tar -xvzf kubeseal-0.30.0-linux-amd64.tar.gz kubeseal

sudo install -m 755 kubeseal /usr/local/bin/kubeseal
```

Create a SealedSecret from an existing plain secret:

```bash
kubectl create secret generic my-app-secret --from-literal=API_KEY=some-secret-key -n default -o yaml --dry-run=client \
  | kubeseal --format yaml > kustomize/base/sealed-secret.yaml
```

> ✅ Note: Only commit SealedSecrets to Git. Plain Secrets should never be stored in Git.

6. **Apply Base and Overlay Configurations**

```bash
# Apply SealedSecret in base
kubectl apply -k kustomize/base

**Screenshot:** kubectl apply -k kustomize/base
![kubectl apply -k kustomize/base](./images/9.apply_k_sealedsecret.png)

# Apply environment overlays
kubectl apply -k kustomize/overlays/dev
kubectl apply -k kustomize/overlays/staging
kubectl apply -k kustomize/overlays/prod
```

7. **Verify Environment Variables in Pods**

```bash
kubectl get pods

# Dev
kubectl exec -it my-app-my-app-dev-7df5cdcf48-b5wq7 -- printenv | grep API_KEY

# Staging
kubectl exec -it my-app-my-app-staging-768484dcc-qb5c4 -- printenv | grep API_KEY

# Prod
kubectl exec -it my-app-my-app-prod-55bdc6b475-xjlrt -- printenv | grep API_KEY
```

Expected output for each pod:

```
API_KEY=some-secret-key
```

8. **Commit the Sealed Secret Updates to GitHub:**

```bash
git add .
git commit -m "Add SealedSecret for API_KEY"
git push origin main
```

9. **Sync Applications with ArgoCD**

```bash
argocd app sync my-app-dev
argocd app sync my-app-staging
argocd app sync my-app-prod
```

Verify in ArgoCD dashboard that **all apps are healthy** and secrets are applied correctly.

### **Notes**

* Dev, Staging, and Prod deployments **reference the SealedSecret** from the base → clean GitOps workflow.
* SealedSecrets are **encrypted and safe** to commit to Git.
* Each cluster has its **own SealedSecret key**, ensuring secrets are cluster-specific.
* ArgoCD automatically deploys SealedSecrets if the controller is installed.
* Use SealedSecrets for all sensitive data like API keys, database passwords, and OAuth tokens.
* For more advanced production-grade secret management, consider **Vault or AWS Secrets Manager**.

## **Task 5: Integrating External Secret Managers with ArgoCD (Local Kubernetes + AWS Secrets Manager)**

### **Objective:**

Enhance secret management by integrating **AWS Secrets Manager** with a **local Kubernetes cluster**. This ensures secrets are managed securely outside Git and injected into Kubernetes clusters during deployments, without incurring significant AWS costs.

### **Steps**

1. **Set Up a Local Kubernetes Cluster**

Use a free local Kubernetes environment like **Kind**, **Minikube**, or **k3d**:

```bash
# Example with Kind
kind create cluster --name my-local-cluster
kubectl cluster-info
```

2. **Install External Secrets Operator (ESO)**

ESO allows Kubernetes to pull secrets from external providers such as AWS Secrets Manager.

```bash
kubectl apply -f https://raw.githubusercontent.com/external-secrets/external-secrets/main/deploy/crds/bundle.yaml
kubectl apply -f https://raw.githubusercontent.com/external-secrets/external-secrets/main/deploy/operator/bundle.yaml
```

Verify installation:

```bash
kubectl get pods -n external-secrets
```

**Screenshot:** kubectl get pods -n external-secrets
[kubectl get pods -n external-secrets](./images/10.kubectl_get_pods_external_secret.png)

3. **🔑 Configure AWS CLI**

After installation, configure it with your AWS credentials:

```bash
aws configure
```

It will prompt for:

* **AWS Access Key ID**
* **AWS Secret Access Key**
* **Default region** (e.g. `us-east-1`)
* **Default output format** (e.g. `json`)

**Screenshot:** aws configure
![aws configure](./images/11.aws_access_ID_key.png)

4. **Store a Secret in AWS Secrets Manager**

```bash
aws secretsmanager create-secret --name my-app/api-key --secret-string "prod-super-secret-key"
```

**Screenshot:** Secret in AWS Secrets Manager
![Secret in AWS Secrets Manager](./images/12.aws_secret_prod_super_secret_key.png)

5. **Create a ClusterSecretStore**

**File:** `kustomize/base/clustersecretstore.yaml`

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ClusterSecretStore
metadata:
  name: aws-secrets-store
spec:
  provider:
    aws:
      service: SecretsManager
      region: us-east-1
      auth:
        secretRef:
          accessKeyIDSecretRef:
            name: aws-credentials
            key: access-key
          secretAccessKeySecretRef:
            name: aws-credentials
            key: secret-access-key
```

> **Note:** For demo purposes, we’ll use AWS IAM credentials as a Kubernetes secret. In production, use **IRSA** or a more secure method.

```bash
kubectl create secret generic aws-credentials \
  --from-literal=access-key-id=AKIAY6QVZHQO7S6Z36SE \
  --from-literal=secret-access-key=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 \
  -n external-secrets
```

6. **Create an ExternalSecret Resource**

**File:** `kustomize/base/externalsecret.yaml`

```yaml
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: my-app-external-secret
spec:
  refreshInterval: 1h
  secretStoreRef:
    name: aws-secrets-store
    kind: ClusterSecretStore
  target:
    name: my-app-secret
    creationPolicy: Owner
  data:
    - secretKey: API_KEY
      remoteRef:
        key: my-app/api-key
```

> This will create a **Kubernetes Secret (`my-app-secret`)** synced from AWS Secrets Manager.

7. **Reference the Secret in Deployment**

Update `kustomize/base/deployment.yaml`:

```yaml
env:
  - name: API_KEY
    valueFrom:
      secretKeyRef:
        name: my-app-secret
        key: API_KEY
```

8. **Deploy with ArgoCD**

Push your changes and sync the application:

```bash
git add .
git commit -m "Integrate External Secrets with AWS Secrets Manager"
git push origin main
```

Sync with ArgoCD:

```bash
argocd app sync my-app-dev
```

Validate:

```bash
kubectl get secrets | grep my-app-secret
kubectl exec -it <pod-name> -- printenv | grep API_KEY
```

Expected output:

```
API_KEY=prod-super-secret-key
```

9. **(Optional) Use ArgoCD Vault Plugin (AVP)**

Instead of ESO, AVP can fetch secrets directly during sync. Example annotation in `kustomization.yaml`:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
metadata:
  annotations:
    avp.kubernetes.io/path: "secret/data/my-app"
```

AVP supports **Vault, AWS Secrets Manager, Azure Key Vault, GCP Secret Manager**.

### **Notes**

* Using **local Kubernetes** (Kind/Minikube/k3d) keeps the project free.
* AWS Secrets Manager cost: **\~\$0.40/month** per secret + minimal API calls.
* ESO is ideal for **cluster-wide syncing**; AVP is better for **smaller setups**.
* In production, prefer **IRSA** or other secure auth methods over long-lived IAM keys.
* External secret managers provide **rotation, auditing, and access policies**, unlike SealedSecrets which store encrypted secrets in Git.

## **Task 6: Customizing Resource Management and Sync Policies in ArgoCD**

### **Objective**

Learn to **customize how ArgoCD handles resources** by ignoring certain differences, adding custom health checks, and setting sync policies (manual vs automated). This fine-tunes GitOps workflows for production.

### **Steps**

1. **Ignore Differences in Resources**
   Sometimes ArgoCD detects changes that don’t need syncing (e.g., autogenerated annotations in Ingress). You can tell ArgoCD to **ignore these fields**.

   Example: Add to `argocd-cm` ConfigMap:

   ```yaml
   resource.customizations.ignoreDifferences: |
     networking.k8s.io/Ingress:
       jsonPointers:
       - /metadata/annotations
   ```

   Apply:

   ```bash
   kubectl -n argocd edit configmap argocd-cm
   ```

   ✅ ArgoCD will now ignore annotation differences in Ingress resources.

2. **Custom Health Checks**
   For CRDs (Custom Resources), you may need **custom health checks** so ArgoCD knows when they’re “Healthy”.

   Example: Add Lua script for a custom resource:

   ```yaml
   resource.customizations: |
     custom.io/MyResource:
       health.lua: |
         hs = {}
         if obj.status ~= nil then
           if obj.status.condition == "Healthy" then
             hs.status = "Healthy"
             hs.message = obj.status.message
           else
             hs.status = "Degraded"
             hs.message = obj.status.message
           end
         end
         return hs
   ```

3. **Sync Policies**
   Decide how changes should sync between Git and cluster.

   **Automated Sync (self-heal + prune):**

   ```yaml
   spec:
     syncPolicy:
       automated:
         prune: true
         selfHeal: true
   ```

   **Manual Sync (safer for prod):**

   ```yaml
   spec:
     syncPolicy: {}
   ```

   ✅ Automated is good for dev/staging.
   ✅ Manual is safer for production, so you approve changes before rollout.

4. **Validate**

   * Check your application in the **ArgoCD UI**.
   * Apply a change in Git (like adding an annotation to an Ingress).
   * See if ArgoCD respects ignoreDifferences or applies custom health.
   * Verify sync behavior (auto vs manual).

### **Notes**

* **Balance automation vs control** → Use automated sync in dev, manual in prod.
* **Resource-specific policies** → Each resource can have its own rules.
* This ensures **reliable, GitOps-driven deployments** without false drifts.

## **Task 7: Git Final Push to GitHub**

In this task, we push our fully configured project (base manifests, overlays for dev/staging/prod, SealedSecrets, External Secrets, and ArgoCD configurations) to GitHub for GitOps automation.

### Steps

1. Add all project files.

   ```bash
   git add .
   ```

2. Commit the changes with a meaningful message.

   ```bash
   git commit -m "Kubernetes GitOps project with Kustomize, SealedSecrets, External Secrets, and ArgoCD"
   ```

3. Push to GitHub main branch.

   ```bash
   git push origin main
   ```

4. Confirm the repository is updated by checking on GitHub.

## **Task 8: Cleanup (Optional)**

After completing the project, it is recommended to clean up resources to avoid unwanted costs and maintain a tidy environment.

### **Steps**

1. **Local Kubernetes Cluster (Free)**
   If you used **Minikube** or **kind**, delete the cluster:

   ```bash
   minikube delete
   # or
   kind delete cluster
   ```

2. **AWS EKS (Paid)**
   If you used **AWS EKS** or created cloud resources:

   2.1 Delete the EKS cluster:

   ```bash
   eksctl delete cluster --name my-cluster --region us-east-1
   ```

   2.2 Delete IAM roles, External Secrets, and AWS Secrets Manager secrets (if no longer needed):

   ```bash
   aws secretsmanager delete-secret --secret-id my-app/api-key --force-delete-without-recovery
   ```

3. **Delete ArgoCD Applications**

   ```bash
   argocd app delete my-app-dev --cascade
   argocd app delete my-app-staging --cascade
   argocd app delete my-app-prod --cascade
   ```

4. **Delete Kubernetes Resources**

   ```bash
   kubectl delete -k kustomize/overlays/dev
   kubectl delete -k kustomize/overlays/staging
   kubectl delete -k kustomize/overlays/prod
   ```

   Delete namespaces (if created):

   ```bash
   kubectl delete namespace dev
   kubectl delete namespace staging
   kubectl delete namespace prod
   ```

5. **Delete SealedSecrets Controller**

   ```bash
   kubectl delete -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.26.0/controller.yaml
   ```

6. **Delete External Secrets Operator (ESO)**

   ```bash
   kubectl delete -k https://github.com/external-secrets/external-secrets//deploy/operator
   kubectl delete -k https://github.com/external-secrets/external-secrets//deploy/crds
   ```

✅ **Notes:**

* Local Kubernetes users only need Step 1.
* AWS users should perform Step 2 to avoid ongoing charges.
* Steps 3–6 are optional but recommended to fully clean the environment.

## **Conclusion**

This project demonstrated a **complete GitOps workflow** for managing Kubernetes applications across multiple environments using:

* **Kustomize** → modular base and overlay manifests for dev, staging, and prod.
* **SealedSecrets** → securely managing sensitive data in Git repositories.
* **External Secrets Operator (ESO)** + **AWS Secrets Manager** → pulling secrets dynamically from external managers.
* **ArgoCD** → automated and declarative GitOps-driven deployments.
* **AWS EKS (or local Minikube/kind)** → as the Kubernetes cluster to host the application.

By following this approach, you ensured:

* **Security** of secrets (never exposing plain Kubernetes secrets in Git).
* **Reusability** of manifests across multiple environments.
* **Automation & Reliability** with ArgoCD syncing directly from GitHub.

This setup is **production-ready** and scalable for real-world DevOps and GitOps workflows.

## **Author**

👨‍💻 **Philip Oluwaseyi Oludolamu**

* DevOps Engineer | Cloud & Kubernetes Enthusiast
* 📍 Based in Northern Cyprus
* 📧 Email: [oluphilix@gmail.com](mailto:oluphilix@gmail.com)
* 🔗 GitHub: [oluphilix](https://github.com/oluphilix)
* 🔗 LinkedIn: [linkedin.com/in/philipoludolamu](https://linkedin.com/in/philipoludolamu)
* 🌍 Dream: To work as a DevOps Engineer in the USA
