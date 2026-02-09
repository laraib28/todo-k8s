# Phase IV: Kubernetes Deployment Runbook

**Version**: 1.0.0
**Date**: 2026-01-08
**Status**: Production Ready
**Deployment Type**: Local Minikube

---

## Overview

This runbook provides complete step-by-step instructions for deploying the Todo application to a local Kubernetes cluster using either raw manifests or Helm charts.

**Infrastructure Generated**: 100% AI-generated (zero manual code)

---

## Table of Contents

1. [Prerequisites](#prerequisites)
2. [Quick Start (Automated)](#quick-start-automated)
3. [Manual Deployment](#manual-deployment)
4. [Helm Deployment](#helm-deployment)
5. [Verification](#verification)
6. [Troubleshooting](#troubleshooting)
7. [Cleanup](#cleanup)

---

## Prerequisites

### Required Tools

- ✅ **Docker**: For building images
- ✅ **Minikube**: Local Kubernetes cluster (v1.30+)
- ✅ **kubectl**: Kubernetes CLI (v1.28+)
- ✅ **Helm**: Package manager (v3.12+)

### Required Resources

- **CPU**: 4 cores minimum
- **Memory**: 8GB minimum
- **Disk**: 20GB free space

### Required Secrets

You will need:
1. **DATABASE_URL**: Neon PostgreSQL connection string
2. **OPENAI_API_KEY**: OpenAI API key for GPT-4o
3. **BETTER_AUTH_SECRET**: JWT secret (can be auto-generated)

---

## Quick Start (Automated)

### Option 1: Automated Deployment Script

```bash
# 1. Prepare secrets
./k8s/prepare-secrets.sh

# 2. Build images
docker build -t todo-frontend:latest ./frontend
docker build -t todo-backend:latest ./backend

# 3. Load images into Minikube
minikube image load todo-frontend:latest
minikube image load todo-backend:latest

# 4. Deploy everything
./k8s/deploy.sh

# 5. Validate deployment
./k8s/validate.sh
```

### Option 2: Helm Chart (Recommended for Production)

```bash
# 1. Update secrets in values.yaml
# Edit helm/todo-app/values.yaml and add base64-encoded secrets

# 2. Build and load images (same as above)
docker build -t todo-frontend:latest ./frontend
docker build -t todo-backend:latest ./backend
minikube image load todo-frontend:latest
minikube image load todo-backend:latest

# 3. Install with Helm
helm install todo-app ./helm/todo-app --namespace todo-app --create-namespace

# 4. Verify
helm status todo-app --namespace todo-app
kubectl get all -n todo-app
```

---

## Manual Deployment

### Step 1: Start Minikube Cluster

```bash
# Start Minikube with recommended resources
minikube start --cpus=4 --memory=8192 --driver=docker

# Enable required addons
minikube addons enable ingress
minikube addons enable metrics-server

# Verify cluster
minikube status
kubectl cluster-info
kubectl get nodes
```

**Expected Output**:
```
NAME       STATUS   ROLES           AGE   VERSION
minikube   Ready    control-plane   1m    v1.28.x
```

### Step 2: Build Docker Images

```bash
# Build frontend image
cd frontend
docker build -t todo-frontend:latest .

# Build backend image
cd ../backend
docker build -t todo-backend:latest .

# Verify images
docker images | grep todo
```

**Expected Output**:
```
todo-frontend   latest   abc123   2 minutes ago   120MB
todo-backend    latest   def456   1 minute ago    180MB
```

### Step 3: Load Images into Minikube

```bash
# Load both images
minikube image load todo-frontend:latest
minikube image load todo-backend:latest

# Verify in Minikube
minikube image ls | grep todo
```

**Expected Output**:
```
docker.io/library/todo-frontend:latest
docker.io/library/todo-backend:latest
```

### Step 4: Prepare Secrets

**Option A: Interactive Script**
```bash
./k8s/prepare-secrets.sh
# Follow prompts to enter secrets
```

**Option B: Manual Encoding**
```bash
# Encode DATABASE_URL
echo -n "postgresql://user:pass@host/db?sslmode=require" | base64
# Output: cG9zdGdyZXNxbDovL3VzZXI6cGFzc0Bob3N0L2RiP3NzbG1vZGU9cmVxdWlyZQ==

# Encode OPENAI_API_KEY
echo -n "sk-proj-your-key-here" | base64
# Output: c2stcHJvai15b3VyLWtleS1oZXJl

# Generate and encode JWT secret
echo -n "$(openssl rand -base64 32)" | base64
# Output: random-base64-string
```

**Update k8s/secret.yaml**:
```yaml
data:
  DATABASE_URL: cG9zdGdyZXNxbDovL3VzZXI6cGFzc0Bob3N0L2RiP3NzbG1vZGU9cmVxdWlyZQ==
  OPENAI_API_KEY: c2stcHJvai15b3VyLWtleS1oZXJl
  BETTER_AUTH_SECRET: random-base64-string
```

⚠️ **WARNING**: Never commit the updated secret.yaml to version control!

### Step 5: Deploy to Kubernetes

```bash
# Deploy in order
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment-frontend.yaml
kubectl apply -f k8s/deployment-backend.yaml
kubectl apply -f k8s/service-frontend.yaml
kubectl apply -f k8s/service-backend.yaml
kubectl apply -f k8s/ingress.yaml

# Wait for rollouts
kubectl rollout status deployment/todo-frontend -n todo-app
kubectl rollout status deployment/todo-backend -n todo-app
```

**Expected Output**:
```
deployment "todo-frontend" successfully rolled out
deployment "todo-backend" successfully rolled out
```

### Step 6: Configure Local DNS

```bash
# Get Minikube IP
minikube ip
# Example output: 192.168.49.2

# Add to /etc/hosts (replace with your Minikube IP)
sudo bash -c 'echo "192.168.49.2 todo.local api.todo.local" >> /etc/hosts'

# Verify
cat /etc/hosts | grep todo
```

### Step 7: Verify Deployment

```bash
# Check all resources
kubectl get all -n todo-app

# Check pods
kubectl get pods -n todo-app

# Check services
kubectl get svc -n todo-app

# Check ingress
kubectl get ingress -n todo-app

# Check logs
kubectl logs -l app=todo-frontend -n todo-app --tail=20
kubectl logs -l app=todo-backend -n todo-app --tail=20
```

### Step 8: Test Application

```bash
# Test frontend
curl -I http://todo.local
# Expected: HTTP/1.1 200 OK

# Test backend health
curl http://api.todo.local/health
# Expected: {"status":"ok"}

# Test backend API docs
curl -I http://api.todo.local/docs
# Expected: HTTP/1.1 200 OK

# Open in browser
xdg-open http://todo.local  # Linux
open http://todo.local      # macOS
```

---

## Helm Deployment

### Step 1: Prepare Values

Edit `helm/todo-app/values.yaml`:

```yaml
secrets:
  DATABASE_URL: "<your-base64-encoded-value>"
  OPENAI_API_KEY: "<your-base64-encoded-value>"
  BETTER_AUTH_SECRET: "<your-base64-encoded-value>"
```

### Step 2: Install Chart

```bash
# Install with default values
helm install todo-app ./helm/todo-app \
  --namespace todo-app \
  --create-namespace

# Or install with custom values
helm install todo-app ./helm/todo-app \
  --namespace todo-app \
  --create-namespace \
  --set frontend.replicaCount=3 \
  --set backend.replicaCount=3
```

### Step 3: Verify Installation

```bash
# Check Helm release
helm list --namespace todo-app

# Check resources
kubectl get all -n todo-app

# Get release notes
helm status todo-app --namespace todo-app
```

### Step 4: Upgrade Configuration

```bash
# Upgrade with new values
helm upgrade todo-app ./helm/todo-app \
  --namespace todo-app \
  --set frontend.replicaCount=4

# Rollback if needed
helm rollback todo-app --namespace todo-app
```

---

## Verification

### Automated Validation

```bash
./k8s/validate.sh
```

### Manual Verification Checklist

- [ ] All pods are running: `kubectl get pods -n todo-app`
- [ ] Services have endpoints: `kubectl get endpoints -n todo-app`
- [ ] Ingress has address: `kubectl get ingress -n todo-app`
- [ ] Frontend accessible: `curl -I http://todo.local`
- [ ] Backend health check: `curl http://api.todo.local/health`
- [ ] No errors in logs: `kubectl logs -l tier=frontend -n todo-app --tail=50`

### Application Feature Testing

1. **User Registration**:
   - Open http://todo.local
   - Click "Register"
   - Create account

2. **User Login**:
   - Login with credentials
   - Verify redirect to dashboard

3. **Task Management**:
   - Create task via form
   - Update task
   - Mark task complete
   - Delete task

4. **AI Chat** (Phase III):
   - Navigate to /chat
   - Send: "Add buy groceries"
   - Verify task created
   - Test: "Show my tasks"

### Performance Checks

```bash
# Check resource usage
kubectl top pods -n todo-app
kubectl top nodes

# Check pod restart counts
kubectl get pods -n todo-app -o custom-columns=NAME:.metadata.name,RESTARTS:.status.containerStatuses[*].restartCount
```

---

## Troubleshooting

### Pods Not Starting

```bash
# Check pod status
kubectl describe pod <pod-name> -n todo-app

# Common issues:
# 1. Image pull errors
kubectl get events -n todo-app | grep -i "pull"

# 2. Secret not found
kubectl get secret todo-secrets -n todo-app

# 3. Resource limits
kubectl top pods -n todo-app
```

### Frontend/Backend Connection Issues

```bash
# Check service endpoints
kubectl get endpoints -n todo-app

# Test service directly
kubectl port-forward svc/todo-backend 8000:80 -n todo-app
curl http://localhost:8000/health

# Check environment variables
kubectl exec -it <backend-pod> -n todo-app -- env | grep -E "DATABASE|OPENAI"
```

### Ingress Not Working

```bash
# Check ingress controller
kubectl get pods -n ingress-nginx

# Check ingress configuration
kubectl describe ingress todo-ingress -n todo-app

# Verify /etc/hosts
ping todo.local

# Test ingress directly
curl -H "Host: todo.local" http://$(minikube ip)
```

### Database Connection Errors

```bash
# Check secret is correct
kubectl get secret todo-secrets -n todo-app -o yaml

# Verify DATABASE_URL format
# Should be: postgresql://user:password@host/database?sslmode=require

# Test from backend pod
kubectl exec -it <backend-pod> -n todo-app -- sh
# Inside pod:
python -c "import os; print(os.getenv('DATABASE_URL'))"
```

### OpenAI API Errors

```bash
# Check logs for API errors
kubectl logs -l app=todo-backend -n todo-app | grep -i openai

# Verify API key
kubectl get secret todo-secrets -n todo-app -o jsonpath='{.data.OPENAI_API_KEY}' | base64 -d
# Should start with: sk-proj-
```

---

## Cleanup

### Remove Deployment (Keep Cluster)

```bash
# Using kubectl
kubectl delete namespace todo-app

# Using Helm
helm uninstall todo-app --namespace todo-app
kubectl delete namespace todo-app
```

### Remove Images from Minikube

```bash
minikube image rm todo-frontend:latest
minikube image rm todo-backend:latest
```

### Stop Minikube

```bash
minikube stop
```

### Delete Cluster

```bash
minikube delete
```

### Remove /etc/hosts Entry

```bash
sudo sed -i '/todo.local/d' /etc/hosts
```

---

## Production Considerations

### When Moving to Production

1. **Image Registry**:
   - Push images to registry (Docker Hub, GCR, ECR)
   - Update `imagePullPolicy` to `IfNotPresent`

2. **Secrets Management**:
   - Use external secrets (Vault, Sealed Secrets)
   - Enable secret encryption at rest

3. **TLS/HTTPS**:
   - Add TLS certificates to ingress
   - Update ingress annotations for HTTPS redirect

4. **Resource Limits**:
   - Tune based on actual usage
   - Set up HorizontalPodAutoscaler

5. **Monitoring**:
   - Enable Prometheus/Grafana
   - Set up alerts for pod failures

6. **Backup**:
   - Database backup strategy
   - Application state backup

7. **High Availability**:
   - Multi-node cluster
   - Pod anti-affinity rules
   - Database replicas

---

## Additional Resources

- **Kubernetes Manifests**: `/k8s/`
- **Helm Chart**: `/helm/todo-app/`
- **Dockerfiles**: `/frontend/Dockerfile`, `/backend/Dockerfile`
- **Automation Scripts**: `/k8s/*.sh`
- **Architecture Docs**: `/specs/004-phase4-kubernetes/`

---

## Support

For issues:
1. Check pod logs: `kubectl logs <pod> -n todo-app`
2. Run validation: `./k8s/validate.sh`
3. Review troubleshooting section above

**End of Runbook**
