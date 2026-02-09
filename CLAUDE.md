# Todo K8s - Phase 4

Kubernetes deployment with Docker, Helm charts, NGINX Ingress, and Minikube setup.

## Quick Start

### Local Development
```bash
cd backend && pip install -r requirements.txt && uvicorn app.main:app --reload
cd frontend && npm install && npm run dev
```

### Kubernetes Deployment
```bash
minikube start
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.yaml
kubectl apply -f k8s/deployment-backend.yaml
kubectl apply -f k8s/deployment-frontend.yaml
kubectl apply -f k8s/service-backend.yaml
kubectl apply -f k8s/service-frontend.yaml
kubectl apply -f k8s/ingress.yaml
```

### With Helm
```bash
helm install todo-app ./helm/todo-app -n todo-app --create-namespace
```

## Architecture
- **Backend**: FastAPI + SQLAlchemy + PostgreSQL + AI chatbot
- **Frontend**: Next.js 16 + TypeScript + Tailwind CSS
- **K8s**: Deployments, Services, Ingress, ConfigMaps, Secrets
- **Helm**: Helm chart for templated deployment
- **Docker**: Multi-stage Dockerfiles
