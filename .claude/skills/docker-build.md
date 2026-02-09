---
name: docker-build
description: Docker image build and push workflow
---

# Docker Build

## Build Images
```bash
docker build -t todo-backend:latest ./backend
docker build -t todo-frontend:latest ./frontend
```

## For Minikube
```bash
eval $(minikube docker-env)
docker build -t todo-backend:latest ./backend
docker build -t todo-frontend:latest ./frontend
```

## Test Locally
```bash
docker run -p 8000:8000 --env-file backend/.env todo-backend:latest
docker run -p 3000:3000 todo-frontend:latest
```
