# ArgoCD Installation and Setup Guide

This guide covers installing ArgoCD on your EKS cluster and deploying the NGINX application.

## Prerequisites
- EKS cluster is running
- kubectl configured to access the cluster
- Git repository with manifests pushed

## Installation Steps

### 1. Install ArgoCD

```bash
# Create argocd namespace
kubectl create namespace argocd

# Install ArgoCD
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

# Wait for ArgoCD pods to be ready
kubectl wait --for=condition=ready pod -l app.kubernetes.io/name=argocd-server -n argocd --timeout=300s
```

### 2. Access ArgoCD UI

#### Option A: Port Forward (Recommended for Testing)
```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```
Access at: https://localhost:8080

#### Option B: LoadBalancer (For External Access)
```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Get the LoadBalancer URL
kubectl get svc argocd-server -n argocd
```

### 3. Get ArgoCD Admin Password

```bash
# Get initial admin password
kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d && echo
```

Login credentials:
- Username: `admin`
- Password: (use command above)

### 4. Install ArgoCD CLI (Optional)

```bash
# For Linux
curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64
```

### 5. Deploy NGINX Application

**Before deploying, update the Git repository URL in `application.yaml`:**

Edit `argocd/application.yaml` and replace:
```yaml
repoURL: https://github.com/YOUR_USERNAME/YOUR_REPO.git
```

Then apply:
```bash
kubectl apply -f argocd/application.yaml
```

### 6. Verify Deployment

```bash
# Check ArgoCD application status
kubectl get applications -n argocd

# Check NGINX pods
kubectl get pods -n nginx-app

# Check NGINX service
kubectl get svc -n nginx-app
```

### 7. Access NGINX Application

#### Via Port Forward:
```bash
kubectl port-forward svc/nginx-service -n nginx-app 8081:80
```
Access at: http://localhost:8081

#### Via LoadBalancer:
```bash
# Get LoadBalancer URL
kubectl get svc nginx-service -n nginx-app
```

## Troubleshooting

### ArgoCD Application Not Syncing
```bash
# Check application details
kubectl describe application nginx-app -n argocd

# Check ArgoCD logs
kubectl logs -n argocd -l app.kubernetes.io/name=argocd-application-controller
```

### NGINX Pods Not Running
```bash
# Check pod status
kubectl describe pod -n nginx-app -l app=nginx

# Check events
kubectl get events -n nginx-app --sort-by='.lastTimestamp'
```

## Clean Up ArgoCD

```bash
# Delete NGINX application
kubectl delete -f argocd/application.yaml

# Delete ArgoCD
kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
kubectl delete namespace argocd
```
