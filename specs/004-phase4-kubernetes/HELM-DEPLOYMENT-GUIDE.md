# Helm Deployment Guide - AI-First Approach

**Generated**: 2026-01-08
**Approach**: Helm-based deployment with summarized results
**Target**: Minikube local cluster

---

## Overview

This guide provides a streamlined Helm-based deployment workflow with clean, summarized results instead of verbose command output. All deployment automation is AI-generated and focuses on result summaries rather than execution logs.

---

## Prerequisites

✅ **Assumed Ready**:
- Docker daemon running
- Minikube cluster running
- kubectl configured and connected
- Helm 3.x installed

---

## Deployment Workflow

### Phase 1: Image Preparation

**Build and Load Images**:
```bash
# Build images
docker build -t todo-frontend:latest ./frontend
docker build -t todo-backend:latest ./backend

# Load to Minikube
minikube image load todo-frontend:latest
minikube image load todo-backend:latest
```

**Expected Result Summary**:
```
✓ Frontend image built (size: ~120MB)
✓ Backend image built (size: ~185MB)
✓ Images loaded to Minikube
```

---

### Phase 2: Secrets Configuration

**Prepare Secrets** (Interactive):
```bash
./k8s/prepare-secrets.sh
```

**Or** manually encode and update `helm/todo-app/values.yaml`:
```yaml
secrets:
  databaseUrl: "<base64-encoded-value>"
  openaiApiKey: "<base64-encoded-value>"
  betterAuthSecret: "<base64-encoded-value>"
```

**Expected Result Summary**:
```
✓ DATABASE_URL encoded
✓ OPENAI_API_KEY encoded
✓ BETTER_AUTH_SECRET generated and encoded
```

---

### Phase 3: Helm Deployment

**Deploy with Helm** (Automated):
```bash
./k8s/helm-deploy.sh
```

This script performs:
1. Pre-deployment validation
2. Image loading verification
3. Helm install/upgrade
4. Deployment status monitoring
5. Health checks
6. Result summarization

**Expected Result Summary**:
```
═══════════════════════════════════════════════════════════
                  Deployment Summary
═══════════════════════════════════════════════════════════

Release Information:
  Name:              todo-app
  Namespace:         todo-app
  Status:            deployed
  Chart Version:     todo-app-1.0.0

Cluster Information:
  Context:           minikube
  Minikube IP:       192.168.49.2
  Pods Running:      4/4

Application Access:
  Frontend:          http://todo.local
  Backend API:       http://api.todo.local/health
  API Docs:          http://api.todo.local/docs

Validation Results:
  Steps Completed:   18/18
  Steps Failed:      0
  Warnings:          0

✅ Deployment successful!
```

---

### Phase 4: Validation

**Run Validation Summary**:
```bash
./k8s/validation-summary.sh
```

**Expected Result Summary**:
```
═══════════════════════════════════════════════════════════
           Validation Summary Report
═══════════════════════════════════════════════════════════

Summary:
  Total Checks:  25
  Passed:        25 (100.0%)
  Failed:        0
  Warnings:      0

Results by Category:
  cluster:        ✓  3/3 passed
  namespace:      ✓  3/3 passed
  deployments:    ✓  4/4 passed
  pods:           ✓  3/3 passed
  services:       ✓  4/4 passed
  ingress:        ✓  2/2 passed
  networking:     ✓  2/2 passed
  health:         ✓  3/3 passed

Deployment Status:
  Release:       todo-app
  Chart:         todo-app-1.0.0
  Pods Running:  4/4

✅ All validation checks passed!
```

---

## Deployment Status Monitoring

### Get Current Status

**JSON Format** (for automation):
```bash
./k8s/deployment-status.sh json
```

**Human-Readable Summary**:
```bash
./k8s/deployment-status.sh summary
```

**Expected Status Summary**:
```
═══════════════════════════════════════════════════════════
           Kubernetes Deployment Status Summary
═══════════════════════════════════════════════════════════

Timestamp: 2026-01-08T10:30:00Z

╔═══ Release ═══╗
  Name:      todo-app
  Namespace: todo-app
  Status:    deployed
  Chart:     todo-app-1.0.0
  Revision:  1

╔═══ Cluster ═══╗
  Context:      minikube
  Minikube IP:  192.168.49.2

╔═══ Deployments ═══╗
  Frontend:  ✓ 2/2 replicas ready
  Backend:   ✓ 2/2 replicas ready

╔═══ Pods ═══╗
  Total:    4
  Running:  4
  Pending:  0
  Failed:   0

╔═══ Services ═══╗
  Frontend:  ✓ 2 endpoint(s)
  Backend:   ✓ 2 endpoint(s)

╔═══ Ingress ═══╗
  Address:  192.168.49.2
  Hosts:    2 configured

╔═══ Application Health ═══╗
  Frontend:  ✓ Accessible (HTTP 200)
  Backend:   ✓ Accessible (HTTP 200)

Access URLs:
  Frontend:     http://todo.local
  Backend API:  http://api.todo.local/health
  API Docs:     http://api.todo.local/docs
```

---

## Helm Management Commands

### Check Release Status

```bash
helm status todo-app -n todo-app
```

**Summary Output**:
```
NAME: todo-app
NAMESPACE: todo-app
STATUS: deployed
REVISION: 1
```

### List Releases

```bash
helm list -n todo-app
```

**Summary Output**:
```
NAME      NAMESPACE  REVISION  STATUS    CHART
todo-app  todo-app   1         deployed  todo-app-1.0.0
```

### View Release Values

```bash
helm get values todo-app -n todo-app
```

### Upgrade Release

```bash
helm upgrade todo-app ./helm/todo-app \
  --namespace todo-app \
  --set frontend.replicaCount=3 \
  --set backend.replicaCount=3
```

**Expected Summary**:
```
Release "todo-app" has been upgraded.
REVISION: 2
```

### Rollback Release

```bash
helm rollback todo-app 1 -n todo-app
```

**Expected Summary**:
```
Rollback was a success! Happy Helming!
```

---

## Troubleshooting

### Quick Diagnostics

**Run Full Diagnostics**:
```bash
./k8s/validation-summary.sh
```

This provides categorized pass/fail results with actionable remediation.

### Check Specific Components

**Pods**:
```bash
kubectl get pods -n todo-app
```

**Services**:
```bash
kubectl get services -n todo-app
```

**Ingress**:
```bash
kubectl get ingress -n todo-app
```

### View Logs

**Frontend Logs** (last 50 lines):
```bash
kubectl logs -l app=todo-frontend -n todo-app --tail=50
```

**Backend Logs** (last 50 lines):
```bash
kubectl logs -l app=todo-backend -n todo-app --tail=50
```

---

## Scaling Operations

### Manual Scaling

**Scale Frontend**:
```bash
helm upgrade todo-app ./helm/todo-app \
  --namespace todo-app \
  --set frontend.replicaCount=3 \
  --reuse-values
```

**Scale Backend**:
```bash
helm upgrade todo-app ./helm/todo-app \
  --namespace todo-app \
  --set backend.replicaCount=3 \
  --reuse-values
```

**Expected Summary**:
```
Release "todo-app" has been upgraded.
Frontend: 2 → 3 replicas
Backend: 2 → 3 replicas
```

### Enable Auto-Scaling (HPA)

Edit `helm/todo-app/values.yaml`:
```yaml
autoscaling:
  frontend:
    enabled: true
    minReplicas: 2
    maxReplicas: 5
    targetCPUUtilizationPercentage: 80

  backend:
    enabled: true
    minReplicas: 2
    maxReplicas: 10
    targetCPUUtilizationPercentage: 80
```

Then upgrade:
```bash
helm upgrade todo-app ./helm/todo-app -n todo-app
```

---

## Cleanup

### Uninstall Release

```bash
helm uninstall todo-app -n todo-app
```

**Expected Summary**:
```
release "todo-app" uninstalled
```

### Delete Namespace

```bash
kubectl delete namespace todo-app
```

**Expected Summary**:
```
namespace "todo-app" deleted
```

### Remove DNS Entries

```bash
sudo sed -i '/todo.local/d' /etc/hosts
```

---

## Deployment Metrics

### Resource Usage

**Check Pod Resource Usage**:
```bash
kubectl top pods -n todo-app
```

**Expected Summary**:
```
NAME                            CPU(cores)   MEMORY(bytes)
todo-frontend-xxx-xxx           50m          120Mi
todo-frontend-xxx-xxx           45m          115Mi
todo-backend-xxx-xxx            80m          250Mi
todo-backend-xxx-xxx            75m          240Mi
```

### Deployment History

```bash
helm history todo-app -n todo-app
```

**Expected Summary**:
```
REVISION  STATUS      CHART           DESCRIPTION
1         deployed    todo-app-1.0.0  Install complete
```

---

## Validation Reports

### Generate Validation Report

```bash
./k8s/validation-summary.sh
```

**Report Location**: `./validation-reports/validation-<timestamp>.json`

**Report Contains**:
- Total checks: 25+
- Pass/fail breakdown by category
- Deployment metadata
- Pod status
- Service health
- Application accessibility

### View Historical Reports

```bash
ls -lt ./validation-reports/
cat ./validation-reports/validation-*.json | jq '.validation_report.summary'
```

---

## CI/CD Integration

### Automated Deployment Script

The `helm-deploy.sh` script is designed for CI/CD integration:

**Exit Codes**:
- `0` = Success (all checks passed)
- `1` = Failure (critical checks failed)

**Example GitLab CI**:
```yaml
deploy:
  stage: deploy
  script:
    - ./k8s/helm-deploy.sh
  only:
    - main
```

**Example GitHub Actions**:
```yaml
- name: Deploy to Minikube
  run: ./k8s/helm-deploy.sh
```

### Status Monitoring

**JSON Output** for monitoring systems:
```bash
./k8s/deployment-status.sh json | jq '.deployment.health'
```

---

## Best Practices

### Pre-Deployment Checklist

✅ All Docker images built and loaded
✅ Secrets properly encoded in values.yaml
✅ Helm chart linted (`helm lint ./helm/todo-app`)
✅ DNS entries configured in /etc/hosts
✅ Minikube ingress addon enabled

### Post-Deployment Validation

✅ Run validation summary (`./k8s/validation-summary.sh`)
✅ Check application accessibility
✅ Verify pod logs for errors
✅ Monitor resource usage
✅ Test core application features

### Monitoring and Maintenance

✅ Regular status checks (`./k8s/deployment-status.sh`)
✅ Log monitoring for errors
✅ Resource usage tracking
✅ Backup database regularly
✅ Keep Helm chart values in version control

---

## Summary

This guide provides a **Helm-first, summary-focused** deployment approach for the Todo application. All scripts generate clean, actionable summaries instead of verbose logs.

**Key Scripts**:
1. `helm-deploy.sh` - Automated Helm deployment with validation
2. `deployment-status.sh` - Current status summary (JSON/YAML/text)
3. `validation-summary.sh` - Comprehensive validation with reports

**Deployment Flow**:
1. Build and load images
2. Configure secrets
3. Run `helm-deploy.sh`
4. Validate with `validation-summary.sh`
5. Monitor with `deployment-status.sh`

**Result**: Clean, summarized deployment with full observability and easy troubleshooting.

---

**End of Helm Deployment Guide**
