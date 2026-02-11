# 🚀 DevOps End-to-End Project

⚙️ **CI/CD • Docker • Blue-Green Deployment • Kubernetes • Monitoring • Alerts**

---

## 📌 Overview

This repository demonstrates a production-style DevOps pipeline — starting from GitHub code commits to automated deployments on Kubernetes with monitoring and alerting.

### The project showcases:

- ✅ Automated CI/CD using Jenkins
- ✅ Docker image build & push to AWS ECR
- ✅ Blue-Green deployment strategy (Docker + Nginx)
- ✅ Kubernetes orchestration using k3s
- ✅ Observability with Prometheus, Grafana & Alertmanager

---

## 🧰 Tech Stack

| Layer | Tools |
|-------|-------|
| Cloud | AWS EC2, AWS ECR |
| CI/CD | Jenkins Pipeline |
| Containers | Docker |
| Deployment | Blue-Green Strategy |
| Orchestration | Kubernetes (k3s) |
| Monitoring | Prometheus |
| Visualization | Grafana |
| Alerts | Alertmanager |
| Language | Python Flask |
| SCM | GitHub |

---

## 🏗️ Architecture

```
Developer → GitHub → Jenkins Pipeline
                     │
                     ▼
                Docker Build
                     │
                     ▼
                   AWS ECR
                     │
                     ▼
            Blue-Green Deployment (Docker)
                     │
                     ▼
               Kubernetes (k3s)
                     │
                     ▼
      Prometheus → Alertmanager → Grafana
```

---

## 🔄 Workflow

| Step | Description |
|------|-------------|
| 1 | Developer pushes code to GitHub |
| 2 | Jenkins triggers pipeline |
| 3 | Docker image builds automatically |
| 4 | Image pushed to AWS ECR |
| 5 | Blue-Green deployment updates container |
| 6 | Kubernetes manages pods |
| 7 | Prometheus collects metrics |
| 8 | Grafana visualizes dashboards |
| 9 | Alerts fire when pods fail |

---

## 📁 Repository Structure

```
devops-end-to-end-project/
│
├── app/                  # Flask application
├── k8s/
│   ├── deployment.yaml   # Kubernetes deployment
│   └── service.yaml      # Kubernetes service
├── helm/
├── terraform/
├── ansible/
├── Jenkinsfile
└── Dockerfile
```

---

## ⚙️ Setup Guide

### 1️⃣ Connect to EC2

```bash
ssh ec2-user@<EC2_PUBLIC_IP>
```

Install dependencies:

```bash
sudo yum update -y
sudo yum install docker git -y
```

### 2️⃣ Install k3s Kubernetes

```bash
curl -sfL https://get.k3s.io | sh -
```

Verify:

```bash
kubectl get nodes
```

### 3️⃣ Jenkins Pipeline

**Pipeline stages:**

- Checkout Source Code
- Build Docker Image
- Login to AWS ECR
- Push Image
- Blue-Green Deployment
- Kubernetes Update

---

## 🐳 Blue-Green Deployment

Zero-downtime deployment strategy:

| Version | Container | Port |
|---------|-----------|------|
| Blue | flask-blue | 5001 |
| Green | flask-green | 5002 |

**Flow:**

```
Old container runs
    ↓
New container launches
    ↓
Nginx switches traffic
    ↓
Old container stops
```

---

## ☸️ Kubernetes Deployment (k3s)

Apply manifests:

```bash
kubectl apply -f k8s/deployment.yaml
kubectl apply -f k8s/service.yaml
```

Check pods:

```bash
kubectl get pods
```

Access app:

```
http://<EC2_PUBLIC_IP>:30007
```

---

## 📊 Monitoring Stack

Installed using Helm:

```bash
helm install monitoring prometheus-community/kube-prometheus-stack -n monitoring
```

**Components:**

- Prometheus
- Grafana
- Alertmanager
- Node Exporter

---

## 🔔 Custom Alert Rule

```yaml
alert: FlaskPodDown
expr: kube_pod_status_ready{namespace="default",pod=~"flask.*",condition="true"} == 0
for: 1m
```

**Alert lifecycle:**

```
Inactive → Pending → Firing
```

---

## 📸 Verification

### Prometheus UI

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-prometheus 9090:9090
```

Open: `http://localhost:9090`

### Grafana Dashboards

```bash
kubectl port-forward -n monitoring svc/monitoring-grafana 3000:80
```

Check:
- Kubernetes / Pods
- Kubernetes / Nodes

### Alertmanager

```bash
kubectl port-forward -n monitoring svc/monitoring-kube-prometheus-alertmanager 9093:9093
```

---

## 🧠 What This Project Demonstrates

Instead of manual deployments, this pipeline provides:

- ✨ Automated builds
- ⚡ Zero-downtime releases
- 🎯 Kubernetes orchestration
- 📊 Real-time monitoring
- 🔔 Intelligent alerting

This mirrors real enterprise DevOps workflows.

---

## 📈 Future Enhancements

- HTTPS Ingress with TLS
- ArgoCD GitOps deployment
- Horizontal Pod Autoscaling
- Slack Alert Integration
- Multi-node Kubernetes cluster

---

## 👨‍💻 Author

**Animish Nishakar**  
DevOps Engineer • Cloud • Automation • Kubernetes
