# Kubernetes Deployment & Service — Hands-on Practical Guide

A hands-on walkthrough of two foundational Kubernetes objects — **Deployments** and **Services** — built around a single NGINX workload exposed through every major Service type. The goal is to understand each concept by actually running it, not just reading about it.

## Table of Contents
- [Concepts](#concepts)
  - [What is a Deployment?](#what-is-a-deployment)
  - [What is a Service?](#what-is-a-service)
  - [Service Types Explained](#service-types-explained)
- [Project Structure](#project-structure)
- [Project Workflow](#project-workflow)
- [Verification](#verification)
- [Cleanup](#cleanup)
- [Quick Comparison](#quick-comparison)

---

## Concepts

### What is a Deployment?

A **Deployment** is a controller that manages a set of identical Pods on your behalf. Instead of creating and babysitting Pods manually, you describe a *desired state* — container image, replica count, update strategy — and the Deployment controller continuously reconciles the cluster to match it.

A Deployment doesn't manage Pods directly; it manages a **ReplicaSet**, which in turn manages the Pods. You'll rarely touch the ReplicaSet yourself — the Deployment is the layer you interact with.

What it gives you:
- **Self-healing** — a crashed Pod or a failed node gets a replacement automatically.
- **Scaling** — `kubectl scale deployment/nginx-deployment --replicas=5` adjusts capacity instantly.
- **Rolling updates** — image or config changes replace Pods gradually, with zero downtime by default.
- **Rollbacks** — `kubectl rollout undo` reverts to the last working revision if a rollout misbehaves.

In this project, `nginx-deployment.yaml` runs **3 replicas** of `nginx:alpine`, each requesting `100m` CPU / `128Mi` memory and capped at `300m` CPU / `256Mi` memory.

### What is a Service?

Pods are disposable — every reschedule gives a Pod a brand-new IP, so addressing Pods directly is unreliable. A **Service** solves this by giving a group of Pods one stable virtual IP and DNS name that other things can depend on, no matter how many times the underlying Pods are replaced.

A Service finds its Pods the same way a Deployment does: via **label selectors**. Every Service in this project selects Pods using `app: nginx`, the same label carried by the Pods the Deployment creates.

> **In short:** a Deployment keeps the right Pods running; a Service gives everything else one reliable way to reach them.

### Service Types Explained

#### 1. ClusterIP — the default
Allocates a virtual IP reachable **only from inside the cluster**. The right choice when a workload only needs to be called by other workloads in the same cluster — the most common pattern for internal service-to-service traffic.

#### 2. NodePort
Everything ClusterIP does, plus a static port (default range `30000–32767`) opened on **every node**. Anything that can reach a node's IP can reach the Service at `<NodeIP>:<NodePort>`. Useful for quick external access during development; rarely the actual entry point in production.

#### 3. LoadBalancer
Everything NodePort does, plus a request to the cloud provider (AWS, GCP, Azure, etc.) to provision an external load balancer with a public IP, routed into the Service. This is the standard way to expose a workload to the internet on a managed cloud cluster. On local clusters like Minikube there's no cloud provider to fulfil that request, so `EXTERNAL-IP` stays `<pending>` unless you run `minikube tunnel` or install something like MetalLB.

#### 4. ExternalName
Doesn't select Pods or proxy any traffic — it's a pure DNS-level alias. A lookup of the Service name returns a `CNAME` pointing at an external domain (here, `www.google.com`, just to demonstrate the mechanism). Useful for giving Pods a clean internal name to call out to a third-party API or an externally hosted database, without hardcoding the real address everywhere.

#### 5. Headless Service
Created by explicitly setting `clusterIP: None`. Instead of one load-balanced virtual IP, DNS returns the **individual IP of every matching Pod**. This powers per-Pod discovery for stateful workloads (commonly paired with StatefulSets), where a client needs to reach a *specific* Pod rather than whichever one a load balancer happens to pick. This manifest also sets `publishNotReadyAddresses: true`, so Pods get a DNS record even before passing their readiness probe — useful for clustered apps (Kafka- or Cassandra-style peer discovery) where Pods need to find each other before any one of them is fully "ready."

---

## Project Structure

```
.
├── deployment/
│   └── nginx-deployment.yaml
├── services/
│   ├── 01-clusterip.yaml
│   ├── 02-nodeport.yaml
│   ├── 03-loadbalancer.yaml
│   ├── 04-externalname.yaml
│   └── 05-headless.yaml
└── README.md
```

---

## Project Workflow

### Prerequisites
A running cluster (Minikube, kind, Docker Desktop, EKS/GKE/AKS...) with `kubectl` already pointing at it.

```bash
minikube start          # only if using Minikube
```

### Step 1 — Create the Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:alpine
          ports:
            - containerPort: 80
          resources:
            requests:
              cpu: "100m"
              memory: "128Mi"
            limits:
              cpu: "300m"
              memory: "256Mi"
```

```bash
kubectl apply -f deployment/nginx-deployment.yaml
kubectl get deploy,pods -l app=nginx
```
Three `nginx:alpine` Pods come up under the Deployment's control.

### Step 2 — Expose internally (ClusterIP)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-clusterip
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

```bash
kubectl apply -f services/01-clusterip.yaml
kubectl get svc nginx-clusterip
```
Any Pod in the cluster can now reach NGINX at `nginx-clusterip:80`.

### Step 3 — Expose per node (NodePort)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-nodeport
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 30080
  type: NodePort
```

```bash
kubectl apply -f services/02-nodeport.yaml
minikube service nginx-nodeport   # convenience command on Minikube
```
NGINX is now reachable from outside the cluster at `<NodeIP>:30080`.

### Step 4 — Expose externally (LoadBalancer)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-loadbalancer
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

```bash
kubectl apply -f services/03-loadbalancer.yaml
kubectl get svc nginx-loadbalancer --watch
```
Watch the `EXTERNAL-IP` column populate once a cloud provider finishes provisioning the load balancer.

### Step 5 — Map to an external domain (ExternalName)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-externalname
spec:
  type: ExternalName
  externalName: www.google.com
```

```bash
kubectl apply -f services/04-externalname.yaml
```
A DNS query for `nginx-externalname` inside the cluster now resolves to `www.google.com`.

### Step 6 — Enable per-Pod DNS (Headless)

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-headless
spec:
  selector:
    app: nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  clusterIP: None
  publishNotReadyAddresses: true
```

```bash
kubectl apply -f services/05-headless.yaml
```
An `nslookup nginx-headless` from inside a Pod now returns all 3 NGINX Pod IPs individually instead of one virtual IP.

---

## Verification

```bash
kubectl get deploy,svc,pods -l app=nginx -o wide
kubectl describe svc nginx-clusterip
kubectl describe svc nginx-headless
```

## Cleanup

```bash
kubectl delete -f services/
kubectl delete -f deployment/
```

---

## Quick Comparison

| Service Type  | Reachable From               | Typical Use Case                                            |
|----------------|-------------------------------|---------------------------------------------------------------|
| ClusterIP      | Inside the cluster only       | Internal service-to-service communication                     |
| NodePort       | Outside, via `<NodeIP>:port`  | Quick external access for dev/testing                         |
| LoadBalancer   | Public internet               | Production-grade external access on a managed cloud cluster   |
| ExternalName   | Internal DNS alias only       | Referring to an external service by a clean internal name     |
| Headless       | Inside the cluster, per-Pod   | Stateful apps that need to address individual Pods directly   |

**Note:** this project deliberately applies all five Service types to the same Deployment for learning purposes. A real application would typically use only one or two together — e.g., ClusterIP internally, with a LoadBalancer or Ingress at the edge.

---

*Part of a hands-on Kubernetes practicals series — Deployments & Services.*