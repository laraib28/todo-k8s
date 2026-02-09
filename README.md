# Todo K8s - Phase 4

Kubernetes deployment of the AI-powered Todo application with Docker, Helm charts, NGINX Ingress, and Minikube.

## Features
- Everything from Phase 3 (web app, auth, AI chatbot, MCP)
- Dockerized backend and frontend
- Kubernetes deployment manifests
- Helm chart for templated deployment
- NGINX Ingress controller
- ConfigMaps and Secrets management

## Tech Stack
- **App**: FastAPI + Next.js 16 + PostgreSQL + OpenAI
- **Containers**: Docker multi-stage builds
- **Orchestration**: Kubernetes (Minikube)
- **Package Manager**: Helm 3
- **Ingress**: NGINX Ingress Controller

## Quick Start

### With Minikube
```bash
minikube start && minikube addons enable ingress
eval $(minikube docker-env)
docker build -t todo-backend:latest ./backend
docker build -t todo-frontend:latest ./frontend
kubectl apply -f k8s/namespace.yaml
kubectl apply -f k8s/
```

## Project Structure
```
backend/           # FastAPI backend
frontend/          # Next.js frontend
k8s/               # Kubernetes manifests
helm/todo-app/     # Helm chart
```

## Built with Claude Code + SpecKit
