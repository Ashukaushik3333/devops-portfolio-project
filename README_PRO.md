DevOps Portfolio Project — CI/CD → Kubernetes → Observability

Short summary
This is a professional case study of a complete end-to-end DevOps pipeline that I built to demonstrate SRE / DevOps skills:

CI: GitHub Actions (build multi-arch image, push to GHCR)

CD: GitHub Actions + Helm charts → Kubernetes (Minikube local)

Infra: Helm chart + k8s manifests

Observability: Prometheus (kube-prometheus-stack) + Grafana custom dashboard

App: Simple Flask app exposing /health and /metrics (Prometheus client)

Impact & outcomes

Implemented full CI/CD: GitHub Actions build → GHCR → Helm deploy

Multi-arch Docker build ensures compatibility with Linux amd64 + arm64 (M1)

Observability stack demonstrates production monitoring & alerting

Deployable locally with Minikube — zero cloud cost for development

Great interview talking points: pipeline design, multi-arch builds, observability

Key links

Flask app: https://github.com/Ashukaushik3333/flask-app

Infra repo: https://github.com/Ashukaushik3333/infra-k8s

Architecture diagram: assets/diagrams/architecture.png

How to run (one-liner summary)

Clone both repos, install Docker & Minikube.

Start minikube: minikube start --driver=docker --memory=4096 --cpus=2.

Deploy infra: run GitHub Actions (or locally run Helm chart in infra-k8s/helm/flask-app).

Confirm metrics & Grafana: kubectl port-forward svc/monitoring-grafana -n monitoring 3000:80.

Contact
Ashutosh Kaushik — GitHub: @Ashukaushik3333 — Email: ashukaushik3333@gmail.com
