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
└── images/
```

**Notes:**

* `helm-charts/` → for Helm templates including `deployment.yaml`, `service.yaml`, and optional `ingress.yaml`.
* `kustomize/` → for Kustomize bases and overlays including `service.yaml`.
* `secrets/` → for Kubernetes secrets or external secret references.
* `images/` → optional, for diagrams or screenshots.
* `README.md` → main project documentation.

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

#### **Step 5: Commit Helm Chart to Git**

```bash
git init
git add .
git commit -m "Add Helm chart for my-app with Docker Hub image"
git remote add origin https://github.com/Holuphilix/argo-advanced-config.git
git push origin main
```

#### **Step 6: Create ArgoCD Application**

* **Using CLI:**

```bash
argocd login localhost:8080 --username admin --password WCAo3rXF1S3BHsJ3 --insecure

argocd app create my-app \
  --repo https://github.com/Holuphilix/argo-advanced-config.git \
  --path helm-charts/my-app \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default \
  --helm-set image.repository=holuphilix/my-k8s-app \
  --helm-set image.tag=latest
```

* **Using UI:**

  * Log in → Click **New App** → Fill repository URL and path → Set cluster & namespace → Enable automated sync.

#### **Step 7: Sync and Verify**

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

---

### **Expected Outcome**

* Minikube cluster running with ArgoCD installed.
* Functional Helm chart for the application.
* ArgoCD application tracking the GitHub repository.
* Application deployed automatically to Kubernetes, pulling images from Docker Hub.

---

### **Notes**

* Ensure Docker Hub image `holuphilix/my-k8s-app:latest` is accessible from the cluster.
* Use environment-specific `values.yaml` for dev/staging/prod.
* This task sets up the foundation for integrating **secrets** and **Kustomize overlays** in later tasks.
