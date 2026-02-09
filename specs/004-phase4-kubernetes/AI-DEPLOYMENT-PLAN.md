# AI-First Deployment Plan - Phase IV Kubernetes

**Generated**: 2026-01-08
**Status**: Ready for Execution
**Approach**: AI-Orchestrated Deployment

---

## Overview

This document outlines the AI-first deployment strategy for deploying the Todo application to Minikube. All steps are orchestrated through AI-generated manifests and automation.

---

## Deployment Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                  AI Deployment Orchestrator                 │
│  - Validates prerequisites                                  │
│  - Executes deployment sequence                             │
│  - Monitors rollout status                                  │
│  - Verifies application health                              │
└───────────────────┬─────────────────────────────────────────┘
                    │
                    ▼
┌─────────────────────────────────────────────────────────────┐
│                    Minikube Cluster                         │
│                                                             │
│  ┌─────────────────────────────────────────────────────┐  │
│  │  Namespace: todo-app                                │  │
│  │                                                       │  │
│  │  ┌──────────────┐         ┌──────────────┐         │  │
│  │  │  Frontend    │         │   Backend    │         │  │
│  │  │  Deployment  │         │  Deployment  │         │  │
│  │  │  2 replicas  │         │  2 replicas  │         │  │
│  │  └──────┬───────┘         └──────┬───────┘         │  │
│  │         │                        │                   │  │
│  │         ▼                        ▼                   │  │
│  │  ┌──────────────┐         ┌──────────────┐         │  │
│  │  │   Service    │         │   Service    │         │  │
│  │  │  ClusterIP   │         │  ClusterIP   │         │  │
│  │  └──────┬───────┘         └──────┬───────┘         │  │
│  │         │                        │                   │  │
│  │         └────────┬───────────────┘                   │  │
│  │                  ▼                                   │  │
│  │         ┌─────────────────┐                          │  │
│  │         │     Ingress     │                          │  │
│  │         │  NGINX routing  │                          │  │
│  │         └────────┬────────┘                          │  │
│  └──────────────────┼────────────────────────────────────┘  │
│                     │                                       │
└─────────────────────┼───────────────────────────────────────┘
                      │
                      ▼
              ┌───────────────┐
              │  /etc/hosts   │
              │  todo.local   │
              │  api.todo.local│
              └───────────────┘
```

---

## Deployment Sequence

### Phase 1: Pre-Deployment Validation

**Objective**: Verify all prerequisites are met

```yaml
validation_checks:
  cluster:
    - command: minikube status
      expected: "minikube: Running"

  kubectl:
    - command: kubectl cluster-info
      expected: "Kubernetes control plane is running"

  docker:
    - command: docker --version
      expected: "Docker version 20.x+"

  resources:
    - cpu: 4 cores available
    - memory: 8GB available
    - disk: 20GB free space
```

**Success Criteria**:
- ✅ Minikube cluster running
- ✅ kubectl configured and connected
- ✅ Docker daemon running
- ✅ Sufficient resources available

---

### Phase 2: Image Build & Preparation

**Objective**: Build optimized Docker images

```bash
# Frontend image build
docker build -t todo-frontend:latest ./frontend

# Expected output:
# [+] Building 120.5s (15/15) FINISHED
# => exporting to image
# => => naming to docker.io/library/todo-frontend:latest

# Backend image build
docker build -t todo-backend:latest ./backend

# Expected output:
# [+] Building 85.3s (12/12) FINISHED
# => exporting to image
# => => naming to docker.io/library/todo-backend:latest
```

**Validation**:
```bash
docker images | grep todo
# todo-frontend   latest   abc123   2 minutes ago   120MB
# todo-backend    latest   def456   1 minute ago    185MB
```

**Success Criteria**:
- ✅ Frontend image < 150MB
- ✅ Backend image < 200MB
- ✅ Both images built successfully
- ✅ No build errors in logs

---

### Phase 3: Image Loading to Minikube

**Objective**: Make images available to Minikube cluster

```bash
# Load frontend image
minikube image load todo-frontend:latest
# Expected: Image loaded to minikube

# Load backend image
minikube image load todo-backend:latest
# Expected: Image loaded to minikube
```

**Validation**:
```bash
minikube image ls | grep todo
# docker.io/library/todo-frontend:latest
# docker.io/library/todo-backend:latest
```

**Success Criteria**:
- ✅ Both images loaded to Minikube
- ✅ Images visible in Minikube image list
- ✅ No ImagePullBackOff errors possible

---

### Phase 4: Secrets Preparation

**Objective**: Encode and configure secrets

```bash
# Database URL encoding
echo -n "postgresql://user:pass@host/db?sslmode=require" | base64
# Output: cG9zdGdyZXNxbDovL3VzZXI6cGFzc0Bob3N0L2RiP3NzbG1vZGU9cmVxdWlyZQ==

# OpenAI API key encoding
echo -n "sk-proj-your-key-here" | base64
# Output: c2stcHJvai15b3VyLWtleS1oZXJl

# JWT secret generation and encoding
echo -n "$(openssl rand -base64 32)" | base64
# Output: (random base64 string)
```

**Update k8s/secret.yaml**:
```yaml
data:
  DATABASE_URL: <base64_encoded_value>
  OPENAI_API_KEY: <base64_encoded_value>
  BETTER_AUTH_SECRET: <base64_encoded_value>
```

**Success Criteria**:
- ✅ All secrets base64 encoded
- ✅ secret.yaml updated with real values
- ✅ No plaintext secrets in manifest
- ✅ secret.yaml NOT committed to version control

---

### Phase 5: Kubernetes Resource Deployment

**Objective**: Deploy all resources in correct order

#### Step 1: Namespace
```bash
kubectl apply -f k8s/namespace.yaml
# namespace/todo-app created
```

#### Step 2: ConfigMap
```bash
kubectl apply -f k8s/configmap.yaml
# configmap/todo-config created
```

#### Step 3: Secret
```bash
kubectl apply -f k8s/secret.yaml
# secret/todo-secrets created
```

#### Step 4: Frontend Deployment
```bash
kubectl apply -f k8s/deployment-frontend.yaml
# deployment.apps/todo-frontend created

kubectl apply -f k8s/service-frontend.yaml
# service/todo-frontend created
```

#### Step 5: Backend Deployment
```bash
kubectl apply -f k8s/deployment-backend.yaml
# deployment.apps/todo-backend created

kubectl apply -f k8s/service-backend.yaml
# service/todo-backend created
```

#### Step 6: Ingress
```bash
kubectl apply -f k8s/ingress.yaml
# ingress.networking.k8s.io/todo-ingress created
```

**Success Criteria**:
- ✅ All manifests applied successfully
- ✅ No YAML validation errors
- ✅ Resources created in todo-app namespace

---

### Phase 6: Rollout Verification

**Objective**: Wait for deployments to become ready

```bash
# Wait for frontend rollout
kubectl rollout status deployment/todo-frontend -n todo-app --timeout=120s
# deployment "todo-frontend" successfully rolled out

# Wait for backend rollout
kubectl rollout status deployment/todo-backend -n todo-app --timeout=120s
# deployment "todo-backend" successfully rolled out
```

**Monitor pods**:
```bash
kubectl get pods -n todo-app -w
# NAME                             READY   STATUS    RESTARTS   AGE
# todo-frontend-7d9b8c5f4d-abc12   1/1     Running   0          45s
# todo-frontend-7d9b8c5f4d-def34   1/1     Running   0          45s
# todo-backend-6c8a7b3e2f-ghi56    1/1     Running   0          40s
# todo-backend-6c8a7b3e2f-jkl78    1/1     Running   0          40s
```

**Success Criteria**:
- ✅ All 4 pods in Running state
- ✅ All pods ready (1/1)
- ✅ No CrashLoopBackOff errors
- ✅ No ImagePullBackOff errors
- ✅ Rollout completed within timeout

---

### Phase 7: Service & Ingress Verification

**Objective**: Verify networking configuration

```bash
# Check services
kubectl get services -n todo-app
# NAME            TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)   AGE
# todo-frontend   ClusterIP   10.96.1.100     <none>        80/TCP    2m
# todo-backend    ClusterIP   10.96.1.101     <none>        80/TCP    2m

# Check endpoints
kubectl get endpoints -n todo-app
# NAME            ENDPOINTS                     AGE
# todo-frontend   172.17.0.5:3000,172.17.0.6:3000   2m
# todo-backend    172.17.0.7:8000,172.17.0.8:8000   2m

# Check ingress
kubectl get ingress -n todo-app
# NAME           CLASS   HOSTS                          ADDRESS         PORTS   AGE
# todo-ingress   nginx   todo.local,api.todo.local      192.168.49.2    80      2m
```

**Success Criteria**:
- ✅ Both services have valid ClusterIP
- ✅ Both services have endpoints
- ✅ Ingress has ADDRESS assigned
- ✅ Ingress routes configured for both hosts

---

### Phase 8: Local DNS Configuration

**Objective**: Configure local DNS for ingress access

```bash
# Get Minikube IP
minikube ip
# 192.168.49.2

# Add to /etc/hosts
echo "192.168.49.2 todo.local api.todo.local" | sudo tee -a /etc/hosts

# Verify
cat /etc/hosts | grep todo
# 192.168.49.2 todo.local api.todo.local
```

**Test DNS resolution**:
```bash
ping -c 1 todo.local
# PING todo.local (192.168.49.2): 56 data bytes

ping -c 1 api.todo.local
# PING api.todo.local (192.168.49.2): 56 data bytes
```

**Success Criteria**:
- ✅ /etc/hosts updated
- ✅ DNS resolves to Minikube IP
- ✅ Both hosts accessible

---

### Phase 9: Application Validation

**Objective**: Verify application is accessible and functional

#### Frontend Tests
```bash
# HTTP status check
curl -I http://todo.local
# HTTP/1.1 200 OK
# Content-Type: text/html

# Full page load
curl -s http://todo.local | grep -i "<title>"
# <title>Todo App</title>
```

#### Backend Tests
```bash
# Health check
curl http://api.todo.local/health
# {"status":"ok"}

# API documentation
curl -I http://api.todo.local/docs
# HTTP/1.1 200 OK
# Content-Type: text/html

# API root
curl http://api.todo.local/
# {"message":"Todo API - See /docs for API documentation"}
```

**Success Criteria**:
- ✅ Frontend returns HTTP 200
- ✅ Backend health check passes
- ✅ API docs accessible
- ✅ No 502/503/504 errors

---

### Phase 10: Pod Health Verification

**Objective**: Verify pod internal health

```bash
# Check pod logs (frontend)
kubectl logs -l app=todo-frontend -n todo-app --tail=20
# Expected: No error messages
# - Server started on port 3000
# - Ready to accept connections

# Check pod logs (backend)
kubectl logs -l app=todo-backend -n todo-app --tail=20
# Expected: No error messages
# - Started server process
# - Uvicorn running on 0.0.0.0:8000
# - Application startup complete

# Check resource usage
kubectl top pods -n todo-app
# NAME                             CPU   MEMORY
# todo-frontend-xxx-xxx            50m   120Mi
# todo-frontend-xxx-xxx            45m   115Mi
# todo-backend-xxx-xxx             80m   250Mi
# todo-backend-xxx-xxx             75m   240Mi
```

**Success Criteria**:
- ✅ No error/warning messages in logs
- ✅ All pods within resource limits
- ✅ Health probes passing
- ✅ No pod restarts

---

## Expected Final State

### Resources Created

```yaml
namespace: todo-app
  deployments: 2
    - todo-frontend (2 replicas)
    - todo-backend (2 replicas)

  services: 2
    - todo-frontend (ClusterIP)
    - todo-backend (ClusterIP)

  ingress: 1
    - todo-ingress (NGINX)

  configmap: 1
    - todo-config

  secret: 1
    - todo-secrets

  pods: 4 total
    - 2 frontend pods (Running)
    - 2 backend pods (Running)
```

### Application Access

- **Frontend**: http://todo.local
- **Backend API**: http://api.todo.local/health
- **API Docs**: http://api.todo.local/docs

### Resource Usage

- **Total CPU**: ~250m (250 millicores)
- **Total Memory**: ~750Mi
- **Total Pods**: 4
- **Total Services**: 2

---

## Validation Checklist

- [ ] Cluster health verified
- [ ] Docker images built successfully
- [ ] Images loaded to Minikube
- [ ] Secrets configured (base64 encoded)
- [ ] All manifests applied
- [ ] Namespace created
- [ ] ConfigMap created
- [ ] Secret created
- [ ] Frontend deployment rolled out
- [ ] Backend deployment rolled out
- [ ] Services created with endpoints
- [ ] Ingress created with routes
- [ ] /etc/hosts configured
- [ ] Frontend accessible (HTTP 200)
- [ ] Backend health check passing
- [ ] API docs accessible
- [ ] No pod errors in logs
- [ ] All pods within resource limits
- [ ] No pod restarts/crashes

---

## Rollback Plan

If deployment fails, rollback with:

```bash
# Delete all resources
kubectl delete namespace todo-app

# Or with Helm (if used)
helm uninstall todo-app --namespace todo-app

# Remove DNS entries
sudo sed -i '/todo.local/d' /etc/hosts

# Remove images from Minikube
minikube image rm todo-frontend:latest
minikube image rm todo-backend:latest
```

---

## Next Steps After Deployment

1. **Test Application Features**:
   - User registration
   - User login
   - Task CRUD operations
   - AI chat functionality

2. **Monitor Performance**:
   ```bash
   kubectl top pods -n todo-app
   kubectl top nodes
   ```

3. **Scale if Needed**:
   ```bash
   kubectl scale deployment/todo-frontend -n todo-app --replicas=3
   kubectl scale deployment/todo-backend -n todo-app --replicas=3
   ```

4. **Enable Auto-scaling** (optional):
   ```bash
   kubectl autoscale deployment todo-frontend -n todo-app --min=2 --max=5 --cpu-percent=80
   kubectl autoscale deployment todo-backend -n todo-app --min=2 --max=10 --cpu-percent=80
   ```

---

## Deployment Completion Report

After successful deployment, generate report with:

```bash
# Deployment summary
kubectl get all -n todo-app

# Detailed pod status
kubectl describe pods -n todo-app

# Ingress configuration
kubectl describe ingress todo-ingress -n todo-app

# Recent events
kubectl get events -n todo-app --sort-by='.lastTimestamp'
```

---

**End of AI-First Deployment Plan**
