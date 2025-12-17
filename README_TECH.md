# DevOps Portfolio — Technical Walkthrough

This document is a **deep technical walkthrough** of the DevOps project demonstrating a complete **CI → CD → Observability** pipeline for a Flask application deployed to Kubernetes using Helm.

This file is intended for:
- Technical reviewers
- Future self-reference
- Understanding exact implementation details.

---

## Table of Contents

1. Project Overview & Goals  
2. Repository Structure (Multi-Repo Design)  
3. Local Prerequisites  
4. End-to-End Setup (Exact Commands)  
5. CI Pipeline (GitHub Actions — flask-app repo)  
6. CD Pipeline (Helm + Kubernetes — infra-k8s repo)  
7. Observability Stack (Prometheus, Grafana, ServiceMonitor)  
8. System Flow (How Everything Connects)  
9. Troubleshooting & Lessons Learned  
10. Architecture Diagram  

---

## 1. Project Overview & Goals

Core objectives:
- Separate **application code** from **infrastructure**
- Implement automated **CI** (build & push container images)
- Implement automated **CD** (deploy via Helm)
- Expose application metrics and visualize them in Grafana
- Keep everything **local, reproducible, and zero-cost**

---

## 2. Repository Structure (Multi-Repo Design)

This project intentionally uses **multiple repositories**, similar to enterprise setups.

devops-portfolio-project/ <-- Documentation-only repo
├── README.md <-- GitHub-rendered overview
├── README_TECH.md <-- This technical guide
├── assets/
│ ├── diagrams/
│ │ ├── architecture.mmd
│ │ └── architecture.png
│ └── screenshots/
│ ├── ci-workflow-success.png
│ ├── cd-workflow-success.png
│ ├── grafana-custom-dashboard.png
│ ├── flask-dashboard-ui.png
│ └── k8s-pods-running.png

flask-app/ <-- Application repo
├── app/
│ └── app.py
├── templates/
│ └── index.html
├── Dockerfile
├── requirements.txt
└── .github/workflows/ci.yaml

infra-k8s/ <-- Infrastructure repo
├── helm/flask-app/
│ ├── Chart.yaml
│ ├── values.yaml
│ └── templates/
│ ├── deployment.yaml
│ ├── service.yaml
│ └── servicemonitor.yaml
└── .github/workflows/deploy.yaml

---

## 3. Local Prerequisites

Tested on **macOS (Apple Silicon)**.

Required tools:
- Docker Desktop (≥ 4 GB memory)
- Homebrew
- brew install kubectl
- minikube

brew install minikube

brew install helm

Start Kubernetes Cluster

minikube start --driver=docker --memory=4096 --cpus=2

Create Namespaces

kubectl create namespace flask-app || true
kubectl create namespace monitoring || true

Install Monitoring Stack

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm repo update

helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

Deploy Application via Helm

cd ~/infra-k8s
helm upgrade --install flask-app helm/flask-app -n flask-app

Verify Pods
kubectl get pods -n flask-app
kubectl get pods -n monitoring

Access Application

minikube service flask-app -n flask-app

Access Grafana

kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80

Get admin password:

kubectl get secret -n monitoring monitoring-grafana \
-o jsonpath="{.data.admin-password}" | base64 -d ; echo


5. CI Pipeline (GitHub Actions — flask-app)
Purpose
Build Docker image

Support multi-architecture (ARM + AMD)

Push image to GitHub Container Registry

Trigger
Every push to main

Key Workflow Steps
- uses: docker/setup-buildx-action@v2

- uses: docker/login-action@v2
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- uses: docker/build-push-action@v5
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ghcr.io/<username>/flask-app:latest
Why Multi-Arch Matters
GitHub runners = amd64

Local machine = arm64

Prevents runtime crashes due to architecture mismatch

6. CD Pipeline (Helm + Kubernetes — infra-k8s)
Purpose
Deploy and upgrade application

Abstract Kubernetes YAML via Helm templates

Helm Chart Components
deployment.yaml

Defines replicas, container image, probes

Health checks on /health

service.yaml

Exposes port 5000

Port name = web (important for Prometheus)

servicemonitor.yaml

Tells Prometheus how and where to scrape metrics

Matches service by label and port name

Deployment Command
helm upgrade --install flask-app helm/flask-app -n flask-app

7. Observability Stack
Metrics Source
Flask exposes metrics via:

python
from prometheus_client import Counter, generate_latest
Endpoint:

/metrics
ServiceMonitor (Critical Detail)
Service port name must match

ServiceMonitor port field

Example:

yaml
endpoints:
- port: web
  path: /metrics
If mismatched → Prometheus shows target DOWN

Verification
curl http://localhost:5000/metrics
Prometheus UI → Status → Targets → flask-app = UP

8. System Flow (How Everything Connects)
Developer pushes code to flask-app

CI workflow builds and pushes Docker image

Helm chart references :latest image

helm upgrade deploys updated pods

Kubernetes Service exposes app

Prometheus scrapes /metrics

Grafana visualizes metrics

9. Troubleshooting & Lessons Learned
Pod Restarting
Cause: health endpoint missing or incorrect

Fix: implement /health and align probes

Metrics Not Showing
Cause: service port name mismatch

Fix: service.yaml port name == ServiceMonitor.port

ImagePull Errors
Cause: architecture mismatch

Fix: multi-arch Docker build

Grafana Shows Huge Memory Values
Cause: incorrect PromQL unit conversion

Fix: divide by 1024^2 for MiB

10. Architecture Diagram
Located at:

assets/diagrams/architecture.png

assets/diagrams/architecture.mmd
To regenerate:

mmdc -i assets/diagrams/architecture.mmd -o assets/diagrams/architecture.png
