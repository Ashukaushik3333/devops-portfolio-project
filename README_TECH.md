DevOps Portfolio — Technical Walkthrough

This document is a full technical guide and troubleshooting playbook for the project that demonstrates CI → CD → Observability for a Flask app, deployed with Helm to Kubernetes.

Table of contents

Overview & goals

Repo layout (multi-repo explained)

Local prerequisites (macOS M1)

Setup — exact commands (copy/paste)

CI (GitHub Actions) — workflows explained

CD (Helm + infra repo) — how it works

Observability (Prometheus + Grafana + ServiceMonitor)

Troubleshooting & common fixes

Architecture (Mermaid) — how to render

1) Overview & goals

Goal: Build an enterprise-style multi-repo pipeline:

flask-app — code, Dockerfile, GitHub Actions workflow to build & push multi-arch images to GHCR.

infra-k8s — Helm chart, ServiceMonitor, GitHub Actions workflow that deploys to Kubernetes (Minikube for local demo).

2) Repo layout (how I structured it)
devops-portfolio-project/        <-- this repo (docs + diagrams)
  README_PRO.md
  README_TECH.md
  diagrams/
    architecture.mmd
    architecture.png
  screenshots/
flask-app/                       <-- app repo (separate)
  app/
    app.py
  Dockerfile
  .github/workflows/ci.yaml
infra-k8s/                       <-- infra repo (separate)
  helm/flask-app/
    Chart.yaml
    values.yaml
    templates/
      deployment.yaml
      service.yaml
      servicemonitor.yaml
  .github/workflows/deploy.yaml
3) Local prerequisites (macOS Sequoia on M1)

Docker Desktop (Apple Silicon) — ensure memory >= 4GB available

Homebrew

kubectl (brew install kubectl)

minikube (brew install minikube)

helm (brew install helm)

Node + npm if you want to run mermaid CLI (npm i -g @mermaid-js/mermaid-cli)

4) Setup — commands
# 1. Start Docker Desktop, ensure it's running.

# 2. Start minikube (adjust memory/cpus per your machine)
minikube start --driver=docker --memory=4096 --cpus=2

# 3. Create namespaces
kubectl create namespace flask-app || true
kubectl create namespace monitoring || true

# 4. Deploy kube-prometheus-stack
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring

# 5. Deploy your helm chart (infra-k8s)
cd ~/infra-k8s
helm upgrade --install flask-app helm/flask-app -n flask-app

# 6. Check pods
kubectl get pods -n flask-app
kubectl get pods -n monitoring

# 7. Port-forward Grafana if you want:
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
# Visit http://localhost:3000
# Admin password:
kubectl get secret -n monitoring monitoring-grafana -o jsonpath="{.data.admin-password}" | base64 -d ; echo

# 8. Access the app:
minikube service flask-app -n flask-app
# or (nodeport)
minikube ip    # e.g., 192.168.49.2
# if service NodePort=30005 then: http://<minikube_ip>:30005
5) CI (GitHub Actions) — essential parts

Key points

Use docker/build-push-action with platforms: linux/amd64,linux/arm64 so you produce multi-arch images.

Push images to GHCR in Actions using a PAT stored as GHCR_PAT or use GITHUB_TOKEN with packages: write permissions.

Minimal build step (ci.yaml)

- name: Set up Docker Buildx
  uses: docker/setup-buildx-action@v2

- name: Login to GHCR
  uses: docker/login-action@v2
  with:
    registry: ghcr.io
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}

- name: Build and push
  uses: docker/build-push-action@v5
  with:
    context: .
    platforms: linux/amd64,linux/arm64
    push: true
    tags: ghcr.io/${{ github.repository_owner }}/flask-app:latest
6) CD (Helm + infra-k8s) — how it works

The infra repo contains a Helm chart helm/flask-app with values.yaml controlling image tag/registry.

GitHub Actions in infra repo is triggered on pushes to main or when CI completes. The CD job:

Pulls the correct image tag from GHCR.

Runs helm upgrade --install against target cluster (for local demo we use kubectl + Minikube).

Optionally runs smoke tests (curl /health, /metrics).

Note: For production you’d switch to a cloud kubecontext (GKE/EKS) and use proper secrets/infra.

7) Observability (ServiceMonitor + Prometheus + Grafana)

Flask app exposes /metrics using prometheus_client.

Service must expose a port named the same as ServiceMonitor expects — I used web in service.yaml and port: web in ServiceMonitor. If they differ, Prometheus won’t scrape.

Ensure the ServiceMonitor is in monitoring namespace and its namespaceSelector includes flask-app.

Key checks

kubectl get servicemonitor -n monitoring
kubectl get endpoints -n flask-app
curl http://localhost:5000/metrics | head
8) Troubleshooting & common fixes

ImagePullBackOff: check GHCR auth and image platform. Use platforms: linux/amd64,linux/arm64 in build.

ServiceMonitor not scraping: verify service port name matches ServiceMonitor port field and that namespaceSelector.matchNames includes flask-app. Confirm Prometheus logs and target page in Prometheus UI.

Grafana not reachable: kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80 then open http://localhost:3000. Admin password from secret (see above).

Minikube memory errors: set minikube start --memory=3072mb or increase Docker Desktop memory.

9) Architecture (Mermaid) — diagrams/architecture.mmd or directly into README
flowchart LR
  subgraph Developer
    A[git push (flask-app)] --> CI[GitHub Actions - CI]
  end

  CI -->|build multi-arch image| Build[Docker buildx]
  Build -->|push| GHCR[ghcr.io/<you>/flask-app:tag]

  subgraph InfraRepo
    B[git push (infra-k8s)] --> CD[GitHub Actions - CD]
  end

  CD -->|helm upgrade| K8s[Kubernetes (Minikube / GKE)]
  K8s --> App[flask-app Pod]
  App -->|exposes /metrics| Prometheus
  Prometheus --> Grafana
  Grafana --> User[Browser]

To generate PNG:

npm i -g @mermaid-js/mermaid-cli
mmdc -i diagrams/architecture.mmd -o diagrams/architecture.png
