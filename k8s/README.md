# My Company Website

A simple static frontend for a 3-tier app. Built with HTML/CSS/JS and served with NGINX.

## Features
- Static webpage with a button
- Containerized with Docker
- Ready for deployment to Kubernetes (EKS)

## Build Docker Image

```bash
docker build -t yourdockerhubusername/frontend:latest -f dockerfiles/Dockerfile.frontend .

