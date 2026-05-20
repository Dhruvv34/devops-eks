# DevOps CI/CD Infrastructure Assignment

Complete CI/CD infrastructure pipeline on AWS using Terraform, Kubernetes (EKS), and ArgoCD.

## Repository Structure

```
├── main.tf                  # EKS cluster and VPC configuration
├── provider.tf              # AWS provider configuration
├── variables.tf             # Input variables
├── outputs.tf               # Outputs
├── manifests/
│   ├── namespace.yaml       # Kubernetes namespace
│   ├── deployment.yaml      # NGINX deployment (2 replicas)
│   └── service.yaml         # LoadBalancer service
├── argocd/
│   ├── application.yaml     # ArgoCD application definition
│   └── README.md            # ArgoCD setup guide
└── README.md                # This file
```

## Prerequisites

- AWS CLI configured with credentials
- Terraform >= 1.0
- kubectl
- Git

---

## Step 1: Provision EKS Cluster with Terraform

### Initialize Terraform
```bash
terraform init
```

### Review the plan
```bash
terraform plan
```

### Apply configuration (creates VPC + EKS cluster)
```bash
terraform apply
```
**Time:** ~10-15 minutes

**Output:**
- Cluster name: `devops-eks`
- Region: `ap-south-1`
- Resources created: 54


---

## Step 2: Configure kubectl

```bash
aws eks update-kubeconfig --region ap-south-1 --name devops-eks
```

### Verify cluster access
```bash
kubectl get nodes
```
---

## Step 3: Deploy NGINX Application

### Create namespace
```bash
kubectl apply -f manifests/namespace.yaml
```

### Deploy NGINX
```bash
kubectl apply -f manifests/deployment.yaml
kubectl apply -f manifests/service.yaml
```

### Verify deployment
```bash
kubectl get all -n nginx-app
```

---

## Step 4: Install ArgoCD

### Create namespace and install
```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### Wait for ArgoCD to be ready
```bash
kubectl get pods -n argocd
```

### Get admin password
```bash
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

---

## Step 5: Access ArgoCD UI

### Port-forward ArgoCD server
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

### Login to ArgoCD
- **URL:** https://localhost:8080
- **Username:** `admin`
- **Password:** (from previous command)

---

## Step 6: Deploy Application via ArgoCD (GitOps)

### Apply ArgoCD application
```bash
kubectl apply -f argocd/application.yaml
```

### Verify sync status
```bash
kubectl get applications -n argocd
```

**Expected Output:**
```
NAME        SYNC STATUS   HEALTH STATUS
nginx-app   Synced        Healthy
```

---

## Step 7: Access NGINX Application

### Option A: Port-forward
```bash
kubectl port-forward svc/nginx-service -n nginx-app 8081:80
```
Access: http://localhost:8081


### Option B: LoadBalancer (Public URL)
```bash
kubectl get svc nginx-service -n nginx-app
```

Get `EXTERNAL-IP` and access via browser or:
```bash
curl http://<EXTERNAL-IP>
```

---

## Cleanup

```bash
# Delete ArgoCD application
kubectl delete -f argocd/application.yaml

# Delete namespaces
kubectl delete namespace nginx-app
kubectl delete namespace argocd

# Wait 30 seconds for LoadBalancer deletion
sleep 30

# Destroy infrastructure
terraform destroy
```

---

## Configuration Details

- **AWS Region:** ap-south-1 (Mumbai)
- **Cluster Name:** devops-eks
- **Kubernetes Version:** 1.31
- **Node Instance Type:** t3.medium
- **Node Count:** 1
- **NGINX Replicas:** 2

---

## Repository

**GitHub:** https://github.com/Dhruvv34/devops-eks.git

---

## Assignment Completion

✅ **Task 1:** EKS Cluster provisioned with Terraform  
✅ **Task 2:** NGINX deployed with Kubernetes manifests  
✅ **Task 3:** ArgoCD installed and configured  
✅ **Task 4:** NGINX accessible via port-forward and LoadBalancer  
✅ **Deliverables:** All files in GitHub with documentation
