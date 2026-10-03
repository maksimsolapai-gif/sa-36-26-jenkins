# Homework Assignment 1: Transform Jenkins Deployment to Helm

This repository contains the solution for **Homework Assignment 1**. The goal of this assignment is to transform a static 5-manifest Kubernetes deployment of Jenkins into a production-ready, fully parameterized Helm chart, package it, and prepare it for repository publication.

## Project Structure

The static Kubernetes manifests (Namespace, RBAC, Storage, ConfigMaps/Secrets, and Istio Ingress) have been refactored into the following Helm chart structure:

```text
.
├── 01-namespace-rbac.yaml
├── 02-storage.yaml
├── 03-config.yaml
├── 04-jenkins.yaml
├── Dockerfile
├── Jenkins
│   ├── Chart.yaml
│   ├── templates
│   │   ├── config-secret.yaml
│   │   ├── deployment-service.yaml
│   │   ├── _helpers.tpl
│   │   ├── istio.yaml
│   │   ├── rbac.yaml
│   │   └── volume.yaml
│   └── values.yaml
├── jenkins-0.1.0.tgz
├── jenkins-istio.yaml
└── README.md
```

## Task Breakdown & Implementation

### 1. Variables Centralization (`values.yaml`)
To fulfill the requirement that **all variables must be inside the variable file**, hardcoded values (such as image tags, NFS paths, replica counts, and Istio hosts) were replaced with Go template placeholders. 

The `values.yaml` handles environment configurations globally:
* **Image Management**: Configurable repository and image tags for seamless CI/CD rollouts.
* **Storage Parameters**: Hardcoded NFS server parameters (`192.168.37.105` and paths) are moved to the storage block.
* **Security & Credentials**: Initial Jenkins administrative credentials and tokens are exposed safely.
* **Istio Routing**: Routing hosts (e.g., `jenkins.k8s-7.sa`) are dynamically injected into the Istio Gateway and VirtualService templates.

### 2. Validation & Quality Assurance
The chart was rigorously tested against formatting and validation rules using Helm's native linting engine:

```bash
helm lint ./Jenkins/
```
*Output: `1 chart(s) linted, 0 chart(s) failed` — indicating clean syntax and successful variable mapping.*

---

## Guide: How to Deploy, Package, and Publish

### Prerequisites
* Kubernetes Cluster with **Istio Service Mesh** installed.
* Helm v3 CLI installed.
* Configured NFS server matching the specifications in `values.yaml`.

### Step 1: Finish Application Deployment (Local Installation)
To deploy the Jenkins application from your local chart folder into the target `ci-cd` namespace (creating it if it does not exist), execute:

```bash
helm install my-jenkins ./Jenkins/ --create-namespace -n ci-cd
```

To verify the running pods and infrastructure components:
```bash
kubectl get all -n ci-cd
```

### Step 2: Create the Helm Package
To bundle the verified Jenkins application into a reusable, compressed distribution archive (`.tgz`), run:

```bash
helm package ./Jenkins/
```
*This command generates a production package archive, such as `jenkins-0.1.0.tgz`.*

### Step 3: Publish Helm on Your Repository
To turn a directory into a hosted Helm repository, generate the necessary registry repository index referencing your remote URL:

```bash
# Generate the index.yaml tracking file
helm repo index . --url https://<your-repository-domain-or-github-pages>/
```

#### Example via GitHub Pages:
1. Push the generated `jenkins-0.1.0.tgz` and `index.yaml` to a public repository (e.g., `helm-charts`).
2. Enable **GitHub Pages** under repository settings.
3. Access or share your published chart globally:
   ```bash
   helm repo add my-jenkins-repo https://<your-username>.github.io/helm-charts/
   helm repo update
   helm install my-jenkins my-jenkins-repo/jenkins -n ci-cd
   ```
