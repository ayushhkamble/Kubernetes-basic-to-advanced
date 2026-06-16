# Kubernetes ReplicaSet using YAML

## Project Overview

This project demonstrates the creation and management of a **Kubernetes ReplicaSet** using a YAML configuration file. The ReplicaSet ensures that a specified number of Pod replicas are always running, providing high availability, fault tolerance, and self-healing capabilities within a Kubernetes cluster.

---

## Objective

The primary objectives of this project are:

* Understand the working of Kubernetes ReplicaSets.
* Deploy multiple identical Pod replicas using YAML.
* Learn label selectors and Pod templates.
* Implement resource requests and limits.
* Observe ReplicaSet self-healing behavior.
* Perform scaling operations on running workloads.

---

## What is a ReplicaSet?

A ReplicaSet is a Kubernetes controller responsible for maintaining a stable set of identical Pods running at any given time.

It continuously monitors the cluster and ensures that the desired number of Pod replicas are available. If a Pod crashes or is deleted, the ReplicaSet automatically creates a new Pod to replace it.

### Key Features

* High Availability
* Self-Healing
* Scalability
* Desired State Management
* Fault Tolerance

---

## Project Structure

```text
replicaset-project/
│
├── replicaset.yaml
└── README.md
```

---

## ReplicaSet Configuration

The ReplicaSet is configured using the following specifications:

| Parameter       | Value      |
| --------------- | ---------- |
| API Version     | apps/v1    |
| Kind            | ReplicaSet |
| Replica Count   | 3          |
| Container Image | nginx:1.25 |
| Container Port  | 80         |
| CPU Request     | 100m       |
| CPU Limit       | 500m       |
| Memory Request  | 128Mi      |
| Memory Limit    | 256Mi      |

---

## YAML Configuration Used

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: nginx-rs
  labels:
    app: nginx
    environment: dev

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
        image: nginx:1.25

        ports:
        - containerPort: 80

        resources:
          requests:
            cpu: "100m"
            memory: "128Mi"

          limits:
            cpu: "500m"
            memory: "256Mi"
```

---

## Prerequisites

Before deploying the ReplicaSet, ensure the following tools are installed:

* Kubernetes Cluster (Minikube, Kind, EKS, AKS, GKE, etc.)
* kubectl CLI
* Docker (for local Kubernetes environments)

---

## Deployment Steps

### Clone the Repository

```bash
git clone <repository-url>
cd replicaset-project
```

### Deploy the ReplicaSet

```bash
kubectl apply -f replicaset.yaml
```

### Verify ReplicaSet Creation

```bash
kubectl get rs
```

Expected Output:

```text
NAME       DESIRED   CURRENT   READY
nginx-rs   3         3         3
```

### Verify Running Pods

```bash
kubectl get pods
```

Expected Output:

```text
NAME             READY   STATUS    RESTARTS
nginx-rs-xxxxx   1/1     Running   0
nginx-rs-yyyyy   1/1     Running   0
nginx-rs-zzzzz   1/1     Running   0
```

---

## Describe ReplicaSet

To view detailed information about the ReplicaSet:

```bash
kubectl describe rs nginx-rs
```

This command displays:

* Replica count
* Pod template information
* Labels and selectors
* Events
* Resource configuration

---

## Self-Healing Demonstration

ReplicaSets automatically replace failed or deleted Pods.

### Delete a Pod

```bash
kubectl delete pod <pod-name>
```

### Verify Recreation

```bash
kubectl get pods
```

A new Pod will be created automatically to maintain the desired replica count of 3.

---

## Scaling the ReplicaSet

### Scale Up

```bash
kubectl scale rs nginx-rs --replicas=5
```

### Verify Scaling

```bash
kubectl get rs
kubectl get pods
```

### Scale Down

```bash
kubectl scale rs nginx-rs --replicas=2
```

---

## Resource Management

The ReplicaSet defines resource requests and limits to ensure efficient cluster utilization.

### Resource Requests

```yaml
requests:
  cpu: "100m"
  memory: "128Mi"
```

These values represent the minimum resources guaranteed to the container.

### Resource Limits

```yaml
limits:
  cpu: "500m"
  memory: "256Mi"
```

These values represent the maximum resources the container can consume.

---

## Labels and Selectors

### Labels

```yaml
labels:
  app: nginx
```

Labels are key-value pairs attached to Kubernetes objects.

### Selector

```yaml
selector:
  matchLabels:
    app: nginx
```

The selector identifies which Pods are managed by the ReplicaSet.

---

## Useful Commands

### View ReplicaSets

```bash
kubectl get rs
```

### View Pods

```bash
kubectl get pods
```

### View Pods with Detailed Information

```bash
kubectl get pods -o wide
```

### Describe Pod

```bash
kubectl describe pod <pod-name>
```

### View Logs

```bash
kubectl logs <pod-name>
```

### Delete ReplicaSet

```bash
kubectl delete -f replicaset.yaml
```

---

## Learning Outcomes

After completing this project, you will understand:

* Kubernetes ReplicaSet architecture
* Pod replication and management
* Label selectors
* Resource requests and limits
* Scaling operations
* Self-healing capabilities
* YAML-based Kubernetes deployments

---

## Conclusion

This project demonstrates how Kubernetes ReplicaSets maintain application availability by ensuring a fixed number of Pod replicas are always running. It highlights essential Kubernetes concepts such as declarative configuration, self-healing, scaling, and resource management, making it a foundational project for anyone learning Kubernetes workload controllers.

---

### Author

**Ayush Kamble**
DevOps | Cloud | Kubernetes | Terraform | AWS Enthusiast 🚀
