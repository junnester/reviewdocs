# Kubernetes Basic Tutorial

## Table of Contents
1. [Introduction](#introduction)
2. [What is Kubernetes?](#what-is-kubernetes)
3. [Core Concepts](#core-concepts)
4. [Architecture](#architecture)
5. [Getting Started](#getting-started)
6. [Basic Commands](#basic-commands)
7. [Deploying Applications](#deploying-applications)
8. [Services and Networking](#services-and-networking)
9. [ConfigMaps and Secrets](#configmaps-and-secrets)
10. [Persistence](#persistence)
11. [Best Practices](#best-practices)

---

## Introduction

Kubernetes (K8s) is an open-source container orchestration platform that automates many of the manual processes involved in deploying, managing, and scaling containerized applications. This tutorial will guide you through the fundamental concepts and practical usage of Kubernetes.

---

## What is Kubernetes?

Kubernetes is a container orchestration system that:
- **Automates deployment** of containerized applications across clusters
- **Manages scaling** based on demand
- **Handles networking** and storage
- **Provides self-healing** capabilities
- **Offers declarative configuration** through YAML manifests

### Why Use Kubernetes?

- **High Availability**: Automatic failover and recovery
- **Scalability**: Easily scale applications up or down
- **Resource Efficiency**: Optimal use of hardware resources
- **Portable**: Run on any infrastructure (cloud, on-premise, hybrid)
- **Community**: Large ecosystem and community support

---

## Core Concepts

### Pods

A Pod is the smallest deployable unit in Kubernetes.

- Represents one or more containers (usually one)
- Containers in a Pod share network namespace (same IP address)
- Ephemeral by nature (not directly created in production)

Example Pod YAML:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
spec:
  containers:
  - name: nginx
    image: nginx:1.21
    ports:
    - containerPort: 80
```

### Deployments

A Deployment manages a set of identical Pods and ensures the desired number of replicas are running.

- Provides declarative updates
- Supports rolling updates and rollbacks
- Ensures high availability

Example Deployment YAML:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      containers:
      - name: nginx
        image: nginx:1.21
        ports:
        - containerPort: 80
```

### Services

A Service exposes Pods to network traffic, either internally or externally.

Types:
- **ClusterIP**: Internal communication only (default)
- **NodePort**: Exposes on a port on each node
- **LoadBalancer**: External load balancer
- **ExternalName**: Maps to external DNS name

Example Service YAML:
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: web
  type: LoadBalancer
  ports:
  - protocol: TCP
    port: 80
    targetPort: 80
```

### Namespaces

Logical divisions of the cluster, used for multi-tenancy and organization.

- Default namespace: `default`
- System namespace: `kube-system`

### Labels and Selectors

Labels are key-value pairs for identifying and organizing Kubernetes objects.

Selectors are queries to find objects by labels.

```yaml
metadata:
  labels:
    app: web
    environment: production
    version: v1
```

---

## Architecture

### Control Plane (Master)

Manages the Kubernetes cluster:
- **API Server**: REST API for cluster management
- **etcd**: Distributed key-value store for cluster data
- **Scheduler**: Assigns Pods to nodes
- **Controller Manager**: Runs controller processes

### Worker Nodes

Run containerized applications:
- **kubelet**: Agent ensuring containers run in Pods
- **Container Runtime**: Docker, containerd, or other container runtime
- **kube-proxy**: Network proxy maintaining network rules

```
┌─────────────────────────────────────────┐
│         Control Plane (Master)          │
│  ┌──────────────┐  ┌──────────────┐    │
│  │  API Server  │  │     etcd     │    │
│  └──────────────┘  └──────────────┘    │
│  ┌──────────────┐  ┌──────────────┐    │
│  │  Scheduler   │  │ Controller   │    │
│  │              │  │  Manager     │    │
│  └──────────────┘  └──────────────┘    │
└─────────────────────────────────────────┘
         │
    ┌────┴────┬─────────┬─────────┐
    │          │         │         │
┌───▼──┐  ┌───▼──┐  ┌───▼──┐  ┌──▼───┐
│Node 1│  │Node 2│  │Node 3│  │Node N│
│┌────┐│  │┌────┐│  │┌────┐│  │┌────┐│
││Pod ││  ││Pod ││  ││Pod ││  ││Pod ││
│└────┘│  │└────┘│  │└────┘│  │└────┘│
└──────┘  └──────┘  └──────┘  └──────┘
```

---

## Getting Started

### Prerequisites

1. **Install kubectl**: Command-line tool for Kubernetes
2. **Install a Kubernetes cluster**:
   - Minikube (local single-node cluster)
   - Docker Desktop (includes Kubernetes)
   - Cloud provider (GKE, EKS, AKS)

### Install kubectl

**macOS (Homebrew):**
```bash
brew install kubectl
```

**Linux:**
```bash
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

**Windows (Chocolatey):**
```bash
choco install kubernetes-cli
```

### Install Minikube

**macOS:**
```bash
brew install minikube
minikube start
```

**Linux/Windows:**
Visit [minikube documentation](https://minikube.sigs.k8s.io/)

### Verify Installation

```bash
kubectl version --client
kubectl cluster-info
kubectl get nodes
```

---

## Basic Commands

### Cluster Information

```bash
# Get cluster info
kubectl cluster-info

# Get nodes
kubectl get nodes

# Describe a node
kubectl describe node <node-name>
```

### Working with Pods

```bash
# List pods in default namespace
kubectl get pods

# List pods in all namespaces
kubectl get pods --all-namespaces

# Get detailed info about pods
kubectl get pods -o wide

# Describe a pod
kubectl describe pod <pod-name>

# Create a pod from YAML
kubectl apply -f pod.yaml

# Delete a pod
kubectl delete pod <pod-name>

# View pod logs
kubectl logs <pod-name>

# Stream logs
kubectl logs -f <pod-name>

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/bash
```

### Working with Deployments

```bash
# List deployments
kubectl get deployments

# Create a deployment
kubectl apply -f deployment.yaml

# Update a deployment
kubectl set image deployment/<deployment-name> <container-name>=<new-image>

# Scale a deployment
kubectl scale deployment <deployment-name> --replicas=5

# View rollout history
kubectl rollout history deployment/<deployment-name>

# Rollback to previous version
kubectl rollout undo deployment/<deployment-name>

# Delete a deployment
kubectl delete deployment <deployment-name>
```

### Working with Services

```bash
# List services
kubectl get services

# Create a service
kubectl apply -f service.yaml

# Describe a service
kubectl describe service <service-name>

# Port forward
kubectl port-forward service/<service-name> 8080:80

# Delete a service
kubectl delete service <service-name>
```

### Namespaces

```bash
# List namespaces
kubectl get namespaces

# Create a namespace
kubectl create namespace <namespace-name>

# Set default namespace
kubectl config set-context --current --namespace=<namespace-name>

# View resources in a namespace
kubectl get pods -n <namespace-name>
```

---

## Deploying Applications

### Step 1: Create a Deployment YAML

```yaml
# app-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
  labels:
    app: hello
spec:
  replicas: 3
  selector:
    matchLabels:
      app: hello
  template:
    metadata:
      labels:
        app: hello
    spec:
      containers:
      - name: hello-container
        image: gcr.io/google-samples/hello-app:1.0
        ports:
        - containerPort: 8080
```

### Step 2: Create a Service YAML

```yaml
# app-service.yaml
apiVersion: v1
kind: Service
metadata:
  name: hello-service
spec:
  selector:
    app: hello
  type: NodePort
  ports:
  - protocol: TCP
    port: 80
    targetPort: 8080
    nodePort: 30080
```

### Step 3: Deploy

```bash
# Apply deployment
kubectl apply -f app-deployment.yaml

# Apply service
kubectl apply -f app-service.yaml

# Verify deployment
kubectl get deployments
kubectl get pods
kubectl get services
```

### Step 4: Access the Application

```bash
# For Minikube
minikube service hello-service

# For port-forward
kubectl port-forward service/hello-service 8080:80
# Then access: http://localhost:8080
```

---

## Services and Networking

### Service Types

#### ClusterIP (Default)
Internal service accessible only within the cluster.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: internal-service
spec:
  type: ClusterIP
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

#### NodePort
Exposes service on a port of each node.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nodeport-service
spec:
  type: NodePort
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
    nodePort: 30080  # Port on nodes (30000-32767)
```

#### LoadBalancer
Exposes service externally with a load balancer.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: loadbalancer-service
spec:
  type: LoadBalancer
  selector:
    app: myapp
  ports:
  - port: 80
    targetPort: 8080
```

#### ExternalName
Maps service to external DNS name.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: external-service
spec:
  type: ExternalName
  externalName: example.com
```

### Ingress

For HTTP(S) routing to services:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: app-ingress
spec:
  rules:
  - host: example.com
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: app-service
            port:
              number: 80
```

---

## ConfigMaps and Secrets

### ConfigMaps

Store non-sensitive configuration data.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  database_host: "db.example.com"
  database_port: "5432"
  log_level: "INFO"
```

Using ConfigMap in Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-config
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        envFrom:
        - configMapRef:
            name: app-config
```

### Secrets

Store sensitive data (base64 encoded).

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: dXNlcm5hbWU=     # base64 encoded
  password: cGFzc3dvcmQ=     # base64 encoded
```

Using Secret in Deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-secrets
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        env:
        - name: DB_USER
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: username
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: password
```

---

## Persistence

### PersistentVolume (PV)

Cluster-level storage resource.

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-data
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  hostPath:
    path: "/mnt/data"
```

### PersistentVolumeClaim (PVC)

Request for storage by a Pod.

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: pvc-data
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
```

### Using PVC in Deployment

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-storage
spec:
  template:
    spec:
      containers:
      - name: app
        image: myapp:1.0
        volumeMounts:
        - name: data
          mountPath: /data
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: pvc-data
```

---

## Best Practices

### 1. Resource Requests and Limits

Always specify resource requirements:

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "500m"
```

### 2. Health Checks

Implement liveness and readiness probes:

```yaml
spec:
  containers:
  - name: app
    image: myapp:1.0
    livenessProbe:
      httpGet:
        path: /health
        port: 8080
      initialDelaySeconds: 10
      periodSeconds: 5
    readinessProbe:
      httpGet:
        path: /ready
        port: 8080
      initialDelaySeconds: 5
      periodSeconds: 5
```

### 3. Security

- Use specific image tags (not `latest`)
- Run containers as non-root users
- Use NetworkPolicies to restrict traffic
- Scan images for vulnerabilities
- Use RBAC for access control

```yaml
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: app
    image: myapp:v1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        drop:
        - ALL
```

### 4. Logging and Monitoring

- Use centralized logging (ELK, Loki, Stackdriver)
- Monitor with Prometheus and Grafana
- Set up alerts

### 5. Documentation

- Document your deployments
- Use meaningful names and labels
- Maintain version control for manifests

### 6. Update Strategy

Use rolling updates for zero-downtime deployments:

```yaml
spec:
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 1
      maxUnavailable: 0
```

### 7. Network Policies

Restrict traffic between pods:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080
```

---

## Conclusion

This tutorial covers the fundamentals of Kubernetes. Key takeaways:

1. **Pods** are the basic building blocks
2. **Deployments** manage application replicas
3. **Services** expose pods to network traffic
4. **ConfigMaps and Secrets** manage configuration
5. **PersistentVolumes** handle storage
6. Always follow best practices for production deployments

### Next Steps

- Explore StatefulSets for stateful applications
- Learn about DaemonSets and Jobs
- Dive into advanced networking with Istio or Linkerd
- Study cluster security and RBAC
- Practice with real-world deployments

### Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes GitHub Repository](https://github.com/kubernetes/kubernetes)
- [Kubernetes Community Slack](https://kubernetes.slack.com/)
- [CNCF Projects](https://www.cncf.io/projects/)

---

**Last Updated**: 2026-04-28

**Happy Kubernetes Learning!** 🚀
