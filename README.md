<div align="center">

# ☸️ Kubernetes: Basic to Advanced

### 🚀 A hands-on, practical roadmap — from your first Pod to event-driven autoscaling

[![Kubernetes](https://img.shields.io/badge/Kubernetes-326CE5?style=for-the-badge&logo=kubernetes&logoColor=white)](https://kubernetes.io/)
[![Docker](https://img.shields.io/badge/Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white)](https://www.docker.com/)
![Status](https://img.shields.io/badge/status-in%20progress-yellow?style=for-the-badge)
![GitHub last commit](https://img.shields.io/github/last-commit/ayushhkamble/Kubernetes-basic-to-advanced?style=for-the-badge)
![GitHub stars](https://img.shields.io/github/stars/ayushhkamble/Kubernetes-basic-to-advanced?style=for-the-badge)

</div>

---

## 📖 About This Repository

This repo is a **practical, example-driven collection of Kubernetes manifests** — built to take you from core fundamentals all the way to advanced cluster operations.

Every topic lives in its own folder with ready-to-apply **YAML files** and the exact `kubectl` commands you need to try it yourself. Perfect for:

- 🎓 Preparing for **CKA / CKAD** certifications
- 💼 Building a **DevOps / SRE** portfolio
- 🧪 Learning Kubernetes the **hands-on** way

---

## 📑 What's Inside

| No. | Topic | Description | Status |
|:---:|-------|-------------|:---:|
| 1 | [Pods](#1-pods) | 🏗️ Smallest deployable unit in Kubernetes | ✅ |
| 2 | [Namespaces](#2-namespaces) | 🗃️ Logical isolation of cluster resources | ⬜ |
| 3 | [ReplicaSets](#3-replicasets) | 🧬 Maintains a stable set of pod replicas | ⬜ |
| 4 | [Deployments](#4-deployments) | 🚀 Declarative updates, rollbacks & scaling | ⬜ |
| 5 | [Services](#5-services) | 🌐 Stable networking for Pods | ⬜ |
| 6 | [DaemonSets](#6-daemonsets) | 🛰️ Runs a pod on every node | ⬜ |
| 7 | [StatefulSets](#7-statefulsets) | 💾 Stateful apps with stable identity | ⬜ |
| 8 | [Ingress](#8-ingress) | 🚪 External HTTP/HTTPS routing | ⬜ |
| 9 | [PV & PVC](#9-pv-and-pvc) | 💽 Persistent storage for Pods | ⬜ |
| 10 | [StorageClass](#10-storageclass) | 🏷️ Dynamic storage provisioning | ⬜ |
| 11 | [RBAC](#11-rbac) | 🔐 Role-based access control | ⬜ |
| 12 | [HPA](#12-hpa) | 📈 Horizontal Pod Autoscaler | ⬜ |
| 13 | [KEDA](#13-keda) | ⚡ Event-driven (scale-to-zero) autoscaling | ⬜ |

---

## 🧰 Prerequisites

| Requirement | Why you need it |
|---|---|
| ☸️ **A Kubernetes cluster** | Minikube, Kind, or a managed cluster (EKS / GKE / AKS) |
| 🔧 **kubectl** | CLI to interact with the cluster — [install guide](https://kubernetes.io/docs/tasks/tools/) |
| 🐳 **Docker basics** | Helps in understanding container images & runtimes |
| 📝 **A good editor** | VS Code + Kubernetes extension is highly recommended |

---

## 🗂️ Topics Covered

### 1. Pods
The **smallest and simplest unit** in the Kubernetes object model. A Pod represents one or more tightly-coupled containers that share storage, network, and a single specification.

> **Key concepts:** single vs. multi-container pods, shared network namespace, ephemeral lifecycle

```bash
kubectl apply -f Pod/pod.yaml
kubectl get pods -o wide
kubectl logs <pod-name>
kubectl describe pod <pod-name>
```

---

### 2. Namespaces
**Virtual clusters** inside a physical cluster — used to divide resources between multiple teams, projects, or environments (`dev`, `staging`, `prod`).

> **Key concepts:** resource quotas & limits, scoped object names, default namespaces (`default`, `kube-system`, `kube-public`)

```bash
kubectl create namespace dev
kubectl apply -f Namespace/namespace.yaml
kubectl get all -n dev
```

---

### 3. ReplicaSets
Ensures that a **specified number of identical pod replicas** are running at all times — the self-healing layer that Deployments build on top of.

> **Key concepts:** label selectors, self-healing, rarely created directly

```bash
kubectl apply -f ReplicaSet/replicaset.yaml
kubectl get rs
kubectl scale rs <name> --replicas=3
```

---

### 4. Deployments
Provides **declarative updates** for Pods & ReplicaSets — the go-to way to run stateless apps with rolling updates, rollbacks, and easy scaling.

> **Key concepts:** rolling updates, `kubectl rollout`, revision history, zero-downtime deploys

```bash
kubectl apply -f Deployment/deployment.yaml
kubectl rollout status deployment/<name>
kubectl rollout undo deployment/<name>
kubectl scale deployment <name> --replicas=5
```

---

### 5. Services
An abstraction that exposes a set of Pods as a **single, stable network endpoint** — so other apps don't need to track changing Pod IPs.

> **Key concepts:**
> - `ClusterIP` – internal-only access (default)
> - `NodePort` – exposes via a static port on every node
> - `LoadBalancer` – provisions an external cloud load balancer
> - `ExternalName` – maps to an external DNS name

```bash
kubectl apply -f Service/service.yaml
kubectl get svc
kubectl expose deployment <name> --type=NodePort --port=80
```

---

### 6. DaemonSets
Ensures a copy of a Pod runs on **every (or selected) node** in the cluster — ideal for log collectors, monitoring agents, and networking plugins.

> **Key concepts:** node-level workloads, auto-adds pods to new nodes, used by tools like `kube-proxy`, Fluentd, etc.

```bash
kubectl apply -f DaemonSet/daemonset.yaml
kubectl get daemonsets -o wide
```

---

### 7. StatefulSets
Manages **stateful applications** by giving each Pod a **stable, unique identity** and persistent storage — used for databases like MySQL, MongoDB, Kafka, and Zookeeper.

> **Key concepts:** ordered deployment/scaling/deletion, stable network IDs (`pod-0`, `pod-1`...), works with headless Services

```bash
kubectl apply -f StatefulSet/statefulset.yaml
kubectl get statefulsets
kubectl get pods -l app=<app-name>
```

---

### 8. Ingress
Manages **external access** to Services — typically HTTP/HTTPS — providing host/path-based routing, SSL/TLS termination, and load balancing through an Ingress Controller (e.g., NGINX, Traefik).

> **Key concepts:** requires an Ingress Controller, virtual hosting, TLS secrets

```bash
kubectl apply -f Ingress/ingress.yaml
kubectl get ingress
kubectl describe ingress <name>
```

---

### 9. PV and PVC
- 💽 **Persistent Volume (PV):** a piece of storage provisioned in the cluster — by an admin, or dynamically via a StorageClass.
- 📌 **Persistent Volume Claim (PVC):** a user's *request* for storage, which Kubernetes binds to a matching PV.

> **Key concepts:** decouples storage from Pod lifecycle, access modes (`ReadWriteOnce`, `ReadOnlyMany`, `ReadWriteMany`), reclaim policies

```bash
kubectl apply -f PV-PVC/pv.yaml
kubectl apply -f PV-PVC/pvc.yaml
kubectl get pv,pvc
```

---

### 10. StorageClass
Defines **"classes" of storage** (e.g., `fast-ssd`, `standard-hdd`) so PVs can be **dynamically provisioned** on demand — no manual PV creation required.

> **Key concepts:** provisioner (e.g., `kubernetes.io/aws-ebs`), reclaim policy, volume binding mode

```bash
kubectl apply -f StorageClass/storageclass.yaml
kubectl get storageclass
```

---

### 11. RBAC
**Role-Based Access Control** governs *who* can perform *what action* on *which resources* — built using `Role` / `ClusterRole` and `RoleBinding` / `ClusterRoleBinding`.

> **Key concepts:** principle of least privilege, namespaced roles vs. cluster-wide roles, service accounts

```bash
kubectl apply -f RBAC/role.yaml
kubectl apply -f RBAC/rolebinding.yaml
kubectl auth can-i list pods --as=<user> -n <namespace>
```

---

### 12. HPA
The **Horizontal Pod Autoscaler** automatically scales the number of Pod replicas up or down based on observed **CPU/memory usage** or custom metrics.

> **Key concepts:** requires `metrics-server`, target utilization %, min/max replica bounds

```bash
kubectl apply -f HPA/hpa.yaml
kubectl get hpa
kubectl top pods
```

---

### 13. KEDA
**Kubernetes Event-Driven Autoscaling** extends (and complements) the HPA to scale workloads based on **events** from external sources — Kafka, RabbitMQ, AWS SQS, Cron, Prometheus, and 60+ scalers — with true **scale-to-zero**.

> **Key concepts:** `ScaledObject` / `ScaledJob` CRDs, event-driven scaling, scale-to-zero

```bash
kubectl apply -f KEDA/scaledobject.yaml
kubectl get scaledobjects
```

---

## 🗺️ Suggested Learning Path

```text
Pods → Namespaces → ReplicaSets → Deployments → Services
     → DaemonSets → StatefulSets → Ingress
     → PV & PVC → StorageClass
     → RBAC → HPA → KEDA
```

> 💡 Each concept builds on the previous one — following this order gives the smoothest learning curve.

---

## 📂 Repository Structure

```text
Kubernetes-basic-to-advanced/
├── Pod/
├── Namespace/
├── ReplicaSet/
├── Deployment/
├── Service/
├── DaemonSet/
├── StatefulSet/
├── Ingress/
├── PV-PVC/
├── StorageClass/
├── RBAC/
├── HPA/
├── KEDA/
└── README.md
```

---

## ⚡ Quick Start

```bash
# 1. Clone the repository
git clone https://github.com/ayushhkamble/Kubernetes-basic-to-advanced.git
cd Kubernetes-basic-to-advanced

# 2. Jump into any topic folder
cd Pod

# 3. Apply the manifest
kubectl apply -f pod.yaml

# 4. Verify it's running
kubectl get all
```

---

## 🤝 Contributing

Suggestions and improvements are always welcome!

1. Fork this repository
2. Create your branch (`git checkout -b feature/new-topic`)
3. Commit your changes (`git commit -m "Add XYZ example"`)
4. Push and open a Pull Request 🎉

---

## ⭐ Show Your Support

If this repository helps you on your Kubernetes journey, please consider giving it a **star** — it motivates further updates and helps others discover it too!

---

<div align="center">

### Maintained by [Ayush Kamble](https://github.com/ayushhkamble)

*Happy Learning! ☸️*

</div>
