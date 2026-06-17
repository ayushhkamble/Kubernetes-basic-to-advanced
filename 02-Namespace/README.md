# Kubernetes Practical: Namespace, Pod, and Service

## Aim

To understand and create Kubernetes resources such as Namespace, Pod, and Service for organizing, running, and exposing applications within a Kubernetes cluster.

---

# 1. Namespace

## Definition

A Namespace is a logical partition within a Kubernetes cluster that helps organize and isolate resources. It allows multiple teams, projects, or environments to share the same cluster without resource name conflicts.

## Purpose

* Provides resource isolation.
* Organizes cluster resources.
* Enables environment separation (Development, Testing, Production).
* Supports resource quotas and access control.

## Real-World Use Case

A company may create separate namespaces such as:

* `dev` for development
* `test` for testing
* `prod` for production

This ensures resources from one environment do not interfere with another.

## Benefits

* Better resource management.
* Easier administration.
* Improved security through isolation.
* Simplified monitoring and troubleshooting.

---

# 2. Pod

## Definition

A Pod is the smallest deployable unit in Kubernetes. It contains one or more containers that share the same network and storage resources.

## Purpose

* Runs application containers.
* Provides a shared execution environment.
* Enables communication between containers within the same pod.

## Key Characteristics

* Each Pod receives its own IP address.
* Containers inside a Pod can communicate using `localhost`.
* Pods are temporary and can be recreated by Kubernetes if managed by higher-level controllers.

## Real-World Use Case

Running an Nginx web server container inside a Pod to serve web pages.

## Benefits

* Easy application deployment.
* Efficient resource utilization.
* Simplified container management.

---

# 3. Service

## Definition

A Service is a Kubernetes object that provides a stable network endpoint for accessing Pods. Since Pod IP addresses can change when Pods restart, a Service offers a consistent way to connect to applications.

## Purpose

* Exposes applications running in Pods.
* Performs load balancing across multiple Pods.
* Provides stable networking within the cluster.

## How It Works

A Service identifies Pods using labels and selectors. Traffic sent to the Service is automatically forwarded to the matching Pods.

## Real-World Use Case

A Service can expose an Nginx Pod so other applications or users can access it without needing to know the Pod's IP address.

## Benefits

* Stable network access.
* Automatic load balancing.
* Simplified application communication.

---

# Relationship Between Namespace, Pod, and Service

```text
Namespace
    │
    ├── Pod
    │      │
    │      └── Runs Application Container
    │
    └── Service
           │
           └── Provides Access to Pod
```

### Workflow

1. A Namespace is created to logically separate resources.
2. A Pod is deployed inside the Namespace to run the application.
3. A Service is created in the same Namespace.
4. The Service discovers the Pod using labels and provides a stable endpoint for accessing it.

---

# Conclusion

In this practical, Kubernetes resources such as Namespace, Pod, and Service are used together to deploy and expose an application. The Namespace provides isolation, the Pod runs the application container, and the Service ensures reliable network access to the application. Together, these resources form the foundation of application deployment and communication in Kubernetes.
