---
name: k8s-deploy
description: Kubernetes deployment workflow
---

# Kubernetes Deployment

## Deploy Steps
```bash
minikube start
minikube addons enable ingress
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/
```

## With Helm
```bash
helm install todo-app ./helm/todo-app -n todo-app --create-namespace
```

## Troubleshooting
```bash
kubectl logs -f deployment/todo-backend -n todo-app
kubectl describe pod <pod-name> -n todo-app
kubectl get events -n todo-app
```
