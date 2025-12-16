# End-to-End DevOps Platform: CI/CD, Kubernetes & Observability

This project demonstrates a complete application delivery platform — from source code to production-like Kubernetes deployment — with built-in CI/CD automation and full observability using Prometheus and Grafana.

The focus of this project is **connecting systems**: application code, containerization, infrastructure configuration, deployment automation, and monitoring — all working together as a cohesive DevOps workflow.

---

## 🚀 What This Project Delivers

- Automated CI pipeline that builds and publishes multi-architecture Docker images
- Kubernetes deployment managed using Helm
- Production-style health checks and metrics exposure
- Prometheus scraping via ServiceMonitor
- Grafana dashboards visualizing application and cluster behavior
- A live application UI showing runtime context

---

## 🏗 Architecture

![Architecture Diagram](assets/diagrams/architecture.png)

**High-level flow:**

1. Code changes are pushed to GitHub
2. CI pipeline builds and pushes container images to GHCR
3. Helm deploys the application to Kubernetes
4. Prometheus scrapes application and cluster metrics
5. Grafana visualizes system health and traffic
6. Application UI reflects runtime status

---

## 🧰 Technology Stack

| Area | Tooling |
|-----|--------|
Application | Flask (Python)
Containerization | Docker (multi-arch builds)
CI | GitHub Actions
Container Registry | GitHub Container Registry (GHCR)
Orchestration | Kubernetes (Minikube)
Deployment | Helm
Monitoring | Prometheus (kube-prometheus-stack)
Visualization | Grafana

---

## 📦 Application Capabilities

The Flask application exposes:

- `/` — Dashboard UI
- `/health` — Health probe for Kubernetes
- `/metrics` — Prometheus-compatible metrics

Custom metrics include:

- `flask_http_requests_total`
- Process and runtime metrics from `prometheus_client`

---

## 📸 Screenshots

| CI Pipeline | Grafana Dashboard | Application UI |
|------------|------------------|----------------|
| ![](assets/screenshots/ci.png) | ![](assets/screenshots/grafana.png) | ![](assets/screenshots/ui.png) |

---

## 📊 Observability Highlights

- Prometheus discovers the application using a `ServiceMonitor`
- Metrics are scraped automatically from `/metrics`
- Grafana dashboards visualize:
  - Request rate
  - Pod health
  - CPU & memory usage
  - Application uptime

---

## 🧠 Engineering Challenges Solved

- Multi-architecture Docker builds for Apple Silicon and amd64
- Kubernetes probe failures due to missing health endpoints
- ServiceMonitor discovery issues caused by port-name mismatches
- Prometheus scraping errors due to incorrect content types
- Helm ownership conflicts when migrating resources to chart-managed state

Each issue was diagnosed using logs, metrics, and Kubernetes introspection — mirroring real operational debugging workflows.

---

## 🔍 Why This Project Matters

This project demonstrates how modern DevOps systems are **designed, operated, and observed** — not just deployed.

It emphasizes:
- Automation over manual steps
- Declarative infrastructure
- Observable systems
- Production-style reliability patterns

---

## 📁 Related Repositories

- **Application & CI** → https://github.com/Ashukaushik3333/flask-app
- **Infrastructure & Helm** → https://github.com/Ashukaushik3333/infra-k8s

---

## 📌 Local Demo

All components run locally using Minikube and can be demonstrated live, end-to-end, without cloud dependencies.

---

## 👤 Author

**Ashutosh Kaushik**  
DevOps / SRE / Cloud Engineer  

🔗 GitHub: https://github.com/ashukaushik3333  
🔗 LinkedIn: https://www.linkedin.com/in/ashutoshkaushikk/  
