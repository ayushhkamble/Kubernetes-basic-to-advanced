# Kubernetes Deployment Strategies — Hands-on Practical

A hands-on comparison of how to roll out a new application version in Kubernetes — from the two strategies built directly into the Deployment object, to the two patterns (Blue-Green and Canary) teams build on top of Deployments and Services for safer, more controlled releases.

## Table of Contents
- [Concepts](#concepts)
  - [What is a Deployment Strategy?](#what-is-a-deployment-strategy)
  - [Recreate](#recreate)
  - [RollingUpdate](#rollingupdate-the-default)
  - [Blue-Green](#blue-green-pattern-not-a-native-strategy)
  - [Canary](#canary-pattern-not-a-native-strategy)
- [Project Structure](#project-structure)
- [Project Workflow](#project-workflow)
- [Verification](#verification)
- [Cleanup](#cleanup)
- [Quick Comparison](#quick-comparison)
- [Beyond This Project](#beyond-this-project)

---

## Concepts

### What is a Deployment Strategy?
When a Deployment's Pod template changes — most commonly a container image bump — Kubernetes has to decide how to move from the old Pods to the new ones. That transition behavior is the **deployment strategy**. Only two are built directly into the Deployment object via `.spec.strategy.type`: `Recreate` and `RollingUpdate`. Everything else — Blue-Green, Canary, A/B testing — is a *pattern* assembled from ordinary Deployments and Services, not a distinct field in the API.

### Recreate
Terminates **all** existing Pods first, waits for them to fully shut down, and only then creates the new ones. Simple and predictable, but causes a brief full outage during the switch. Appropriate when old and new versions genuinely can't run side by side — for example, a breaking database schema migration, or a singleton workload that can't have two differing replicas active at once.

### RollingUpdate (the default)
Replaces Pods incrementally, keeping the application available throughout. Two fields control the pace:
- **`maxUnavailable`** — how many Pods (count or %) can be down at once during the update.
- **`maxSurge`** — how many extra Pods (count or %) can be created above the desired replica count while the update is in progress.

With `replicas: 4`, `maxUnavailable: 1`, `maxSurge: 1`, Kubernetes never drops below 3 available Pods and never exceeds 5 total while cycling through the update — a smooth, zero-downtime rollout for most stateless workloads.

### Blue-Green (pattern, not a native strategy)
Run two complete, independent environments — **blue** (the current live version) and **green** (the new version) — at the same time, each as its own Deployment. A single Service points at blue. Once green has been tested and verified healthy, you switch the Service's selector to green and traffic moves over **instantly and atomically**. Rolling back is just as fast: flip the selector back to blue. The trade-off is running double the capacity while both versions exist.

### Canary (pattern, not a native strategy)
Run the new version alongside the old one, but give it a deliberately small slice of traffic — for example, 9 "stable" Pods and 1 "canary" Pod behind a single Service that selects both. Because a Service load-balances evenly across every Pod IP it matches, the canary organically receives roughly 1-in-10 requests. If it behaves well, gradually shift the ratio toward canary (eventually replacing stable entirely); if it doesn't, scale canary back to zero with minimal blast radius.

---

## Project Structure
```
.
├── 01-recreate/
│   └── recreate-deployment.yaml
├── 02-rolling-update/
│   └── rolling-update-deployment.yaml
├── 03-blue-green/
│   ├── blue-deployment.yaml
│   ├── green-deployment.yaml
│   └── service.yaml
├── 04-canary/
│   ├── stable-deployment.yaml
│   ├── canary-deployment.yaml
│   └── service.yaml
└── README.md
```

---

## Project Workflow

### Strategy 1 — Recreate

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-recreate
  labels:
    app: nginx-recreate
spec:
  replicas: 3
  strategy:
    type: Recreate
  selector:
    matchLabels:
      app: nginx-recreate
  template:
    metadata:
      labels:
        app: nginx-recreate
    spec:
      containers:
        - name: nginx
          image: nginx:1.24
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f 01-recreate/recreate-deployment.yaml
kubectl get pods -l app=nginx-recreate -w
```

Trigger the strategy with an image update and watch all 3 Pods terminate together before any replacement appears:
```bash
kubectl set image deployment/nginx-recreate nginx=nginx:1.25
kubectl get pods -l app=nginx-recreate -w
```

### Strategy 2 — RollingUpdate

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-rolling
  labels:
    app: nginx-rolling
spec:
  replicas: 4
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
  selector:
    matchLabels:
      app: nginx-rolling
  template:
    metadata:
      labels:
        app: nginx-rolling
    spec:
      containers:
        - name: nginx
          image: nginx:1.24
          ports:
            - containerPort: 80
```

```bash
kubectl apply -f 02-rolling-update/rolling-update-deployment.yaml
kubectl rollout status deployment/nginx-rolling
```

Trigger the rollout and watch Pods get replaced one at a time, never dropping below 3 available or exceeding 5 total:
```bash
kubectl set image deployment/nginx-rolling nginx=nginx:1.25
kubectl rollout status deployment/nginx-rolling
kubectl get pods -l app=nginx-rolling -w
```

Roll back if needed:
```bash
kubectl rollout history deployment/nginx-rolling
kubectl rollout undo deployment/nginx-rolling
```

### Strategy 3 — Blue-Green

```bash
kubectl apply -f 03-blue-green/blue-deployment.yaml
kubectl apply -f 03-blue-green/service.yaml
kubectl apply -f 03-blue-green/green-deployment.yaml   # deployed, but the Service still sends traffic to blue
```

Once green's Pods are healthy and verified (e.g. by port-forwarding directly into a green Pod), cut traffic over instantly by patching the Service's selector:
```bash
kubectl patch service my-app-service -p '{"spec":{"selector":{"version":"green"}}}'
```

If anything looks wrong, roll back just as instantly by patching the selector back to `blue`. Once you're confident, remove the old `app-blue` Deployment.

### Strategy 4 — Canary

```bash
kubectl apply -f 04-canary/stable-deployment.yaml
kubectl apply -f 04-canary/canary-deployment.yaml
kubectl apply -f 04-canary/service.yaml
```

With 9 stable + 1 canary Pod behind the same Service, the canary receives roughly 10% of requests. Ramp it up gradually:
```bash
kubectl scale deployment/app-canary --replicas=3
kubectl scale deployment/app-stable --replicas=7
```
Keep shifting the ratio until canary fully replaces stable — or scale canary back to 0 to abort with minimal impact.

---

## Verification

```bash
kubectl get deploy -o wide
kubectl rollout status deployment/<name>
kubectl describe deployment/<name>
```

## Cleanup

```bash
kubectl delete -f 01-recreate/
kubectl delete -f 02-rolling-update/
kubectl delete -f 03-blue-green/
kubectl delete -f 04-canary/
```

---

## Quick Comparison

| Strategy       | Native to Kubernetes?              | Downtime          | Rollback Speed                  | Best For                                                        |
|------------------|--------------------------------------|---------------------|-------------------------------------|-----------------------------------------------------------------|
| Recreate         | Yes                                  | Brief, full         | Fast (recreate again)               | Apps that can't run two versions at once (schema migrations, singletons) |
| RollingUpdate    | Yes (default)                        | None (tunable)       | Moderate (re-roll previous revision) | Most stateless web apps and APIs                                  |
| Blue-Green       | No — 2 Deployments + 1 Service       | None                 | Instant (flip selector back)         | High-confidence releases that need an instant rollback path        |
| Canary           | No — 2 Deployments + 1 Service       | None                 | Fast (scale canary to 0)             | Validating a risky change against real traffic before a full rollout |

## Beyond This Project
Hand-rolled Blue-Green and Canary work well for learning and small setups, but at scale teams usually automate them with tools like **Argo Rollouts** or **Flagger**, often paired with a service mesh (Istio, Linkerd) for fine-grained traffic splitting, automated metric-based promotion or rollback, and true A/B testing based on request headers rather than plain replica counts.

---

*Part of a hands-on Kubernetes practicals series — Deployment Strategies.*