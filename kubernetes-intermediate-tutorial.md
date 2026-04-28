# Kubernetes Intermediate Tutorial

## Table of Contents
1. [Introduction](#introduction)
2. [Advanced Workload Management](#advanced-workload-management)
3. [StatefulSets and DaemonSets](#statefulsets-and-daemonsets)
4. [Jobs and CronJobs](#jobs-and-cronjobs)
5. [Advanced Networking](#advanced-networking)
6. [Storage Solutions](#storage-solutions)
7. [RBAC and Security](#rbac-and-security)
8. [Monitoring and Logging](#monitoring-and-logging)
9. [Helm Package Manager](#helm-package-manager)
10. [Custom Resource Definitions (CRDs)](#custom-resource-definitions-crds)
11. [Operators and Controllers](#operators-and-controllers)
12. [Best Practices](#best-practices)

---

## Introduction

This intermediate-level tutorial builds upon Kubernetes fundamentals and covers advanced concepts for production deployments. We'll explore stateful applications, complex networking, security implementations, monitoring strategies, and operational tools.

### Prerequisites

- Solid understanding of Pods, Deployments, and Services
- Familiarity with kubectl commands
- Knowledge of YAML syntax
- Experience deploying basic applications

---

## Advanced Workload Management

### Replica Set

Lower-level construct than Deployments; rarely used directly.

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: advanced-rs
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
      version: v2
  template:
    metadata:
      labels:
        app: myapp
        version: v2
    spec:
      containers:
      - name: myapp
        image: myapp:v2.0
        resources:
          requests:
            memory: "128Mi"
            cpu: "100m"
          limits:
            memory: "256Mi"
            cpu: "500m"
```

### Pod Disruption Budgets (PDB)

Ensures minimum availability during voluntary disruptions (maintenance, scaling).

```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  minAvailable: 2
  selector:
    matchLabels:
      app: myapp
```

Alternative using `maxUnavailable`:
```yaml
apiVersion: policy/v1
kind: PodDisruptionBudget
metadata:
  name: myapp-pdb
spec:
  maxUnavailable: 1
  selector:
    matchLabels:
      app: myapp
```

### Affinity Rules

Control Pod placement on nodes.

#### Node Affinity

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: node-affinity-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: disktype
                operator: In
                values:
                - ssd
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: node-type
                operator: In
                values:
                - compute-optimized
      containers:
      - name: myapp
        image: myapp:v1.0
```

#### Pod Affinity

Schedule pods relative to other pods.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: pod-affinity-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: web
  template:
    metadata:
      labels:
        app: web
    spec:
      affinity:
        podAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - database
            topologyKey: kubernetes.io/hostname
      containers:
      - name: web
        image: web:v1.0
```

#### Pod Anti-Affinity

Spread pods across nodes for high availability.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: anti-affinity-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      affinity:
        podAntiAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            podAffinityTerm:
              labelSelector:
                matchExpressions:
                - key: app
                  operator: In
                  values:
                  - myapp
              topologyKey: kubernetes.io/hostname
      containers:
      - name: myapp
        image: myapp:v1.0
```

### Taints and Tolerations

Control which pods can run on specific nodes.

**Add taint to node:**
```bash
kubectl taint nodes node1 key=value:NoSchedule
```

**Deployment with toleration:**
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: specialized-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: specialized
  template:
    metadata:
      labels:
        app: specialized
    spec:
      tolerations:
      - key: key
        operator: Equal
        value: value
        effect: NoSchedule
      containers:
      - name: app
        image: specialized-app:v1.0
```

---

## StatefulSets and DaemonSets

### StatefulSets

For applications requiring stable identity and persistent storage (databases, message queues).

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: mysql-statefulset
spec:
  serviceName: mysql
  replicas: 3
  selector:
    matchLabels:
      app: mysql
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - name: mysql
        image: mysql:8.0
        ports:
        - containerPort: 3306
          name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysql-secret
              key: root-password
        volumeMounts:
        - name: data
          mountPath: /var/lib/mysql
        resources:
          requests:
            memory: "512Mi"
            cpu: "250m"
          limits:
            memory: "1Gi"
            cpu: "1000m"
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 20Gi
---
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  clusterIP: None
  selector:
    app: mysql
  ports:
  - port: 3306
    targetPort: 3306
```

### DaemonSets

Runs one pod per node (monitoring, logging, networking).

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
spec:
  selector:
    matchLabels:
      app: node-exporter
  template:
    metadata:
      labels:
        app: node-exporter
    spec:
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule
      containers:
      - name: node-exporter
        image: prom/node-exporter:latest
        ports:
        - containerPort: 9100
          name: metrics
        volumeMounts:
        - name: proc
          mountPath: /host/proc
          readOnly: true
        - name: sys
          mountPath: /host/sys
          readOnly: true
      volumes:
      - name: proc
        hostPath:
          path: /proc
      - name: sys
        hostPath:
          path: /sys
```

---

## Jobs and CronJobs

### Jobs

Run one or more pods to completion.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: data-backup-job
spec:
  backoffLimit: 3
  activeDeadlineSeconds: 3600
  completions: 1
  parallelism: 1
  template:
    metadata:
      labels:
        app: backup
    spec:
      restartPolicy: Never
      containers:
      - name: backup
        image: backup-tool:v1.0
        env:
        - name: DATABASE_URL
          valueFrom:
            secretKeyRef:
              name: db-credentials
              key: url
        - name: BACKUP_DESTINATION
          value: "s3://backups/daily"
        resources:
          requests:
            memory: "256Mi"
            cpu: "500m"
          limits:
            memory: "512Mi"
            cpu: "1000m"
```

### CronJobs

Run Jobs on a schedule.

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: daily-backup
spec:
  schedule: "0 2 * * *"  # 2 AM daily
  concurrencyPolicy: Forbid
  successfulJobsHistoryLimit: 3
  failedJobsHistoryLimit: 1
  jobTemplate:
    spec:
      backoffLimit: 2
      template:
        metadata:
          labels:
            app: backup
        spec:
          restartPolicy: OnFailure
          containers:
          - name: backup
            image: backup-tool:v1.0
            env:
            - name: BACKUP_TYPE
              value: "daily"
            resources:
              requests:
                memory: "512Mi"
                cpu: "1000m"
              limits:
                memory: "1Gi"
                cpu: "2000m"
```

---

## Advanced Networking

### Network Policies

Fine-grained traffic control between pods and namespaces.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: frontend-backend-policy
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    - namespaceSelector:
        matchLabels:
          name: ingress
    ports:
    - protocol: TCP
      port: 8080
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: TCP
      port: 53
    - protocol: UDP
      port: 53
```

Default deny all traffic:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### Ingress with Advanced Routing

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: advanced-ingress
  annotations:
    cert-manager.io/cluster-issuer: letsencrypt-prod
    nginx.ingress.kubernetes.io/ssl-redirect: "true"
    nginx.ingress.kubernetes.io/rate-limit: "100"
spec:
  ingressClassName: nginx
  tls:
  - hosts:
    - api.example.com
    - admin.example.com
    secretName: example-tls
  rules:
  - host: api.example.com
    http:
      paths:
      - path: /v1
        pathType: Prefix
        backend:
          service:
            name: api-v1
            port:
              number: 8080
      - path: /v2
        pathType: Prefix
        backend:
          service:
            name: api-v2
            port:
              number: 8080
  - host: admin.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: admin-panel
            port:
              number: 3000
```

### Service Mesh (Istio Example)

Define traffic policies with VirtualService:
```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: myapp-vs
spec:
  hosts:
  - myapp
  http:
  - match:
    - uri:
        prefix: /api
    route:
    - destination:
        host: myapp
        port:
          number: 8080
        subset: v1
      weight: 80
    - destination:
        host: myapp
        port:
          number: 8080
        subset: v2
      weight: 20
```

---

## Storage Solutions

### Storage Classes

Define different types of storage.

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-ssd
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp3
  iops: "3000"
  throughput: "125"
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
```

### StatefulSet with Dynamic Storage

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres-statefulset
spec:
  serviceName: postgres
  replicas: 2
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:14
        ports:
        - containerPort: 5432
        env:
        - name: POSTGRES_PASSWORD
          valueFrom:
            secretKeyRef:
              name: postgres-secret
              key: password
        volumeMounts:
        - name: data
          mountPath: /var/lib/postgresql/data
          subPath: postgres
      volumes:
      - name: data
        persistentVolumeClaim:
          claimName: postgres-pvc
  volumeClaimTemplates:
  - metadata:
      name: data
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: fast-ssd
      resources:
        requests:
          storage: 50Gi
```

### ConfigMaps and Secrets as Volumes

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: app-config
data:
  config.yaml: |
    server:
      port: 8080
      timeout: 30
    database:
      pool_size: 20
---
apiVersion: v1
kind: Secret
metadata:
  name: db-credentials
type: Opaque
data:
  username: cG9zdGdyZXM=
  password: c2VjdXJlcGFzc3dvcmQ=
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: app-with-configs
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        volumeMounts:
        - name: config
          mountPath: /etc/config
          readOnly: true
        - name: secrets
          mountPath: /etc/secrets
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: app-config
          items:
          - key: config.yaml
            path: config.yaml
      - name: secrets
        secret:
          secretName: db-credentials
          defaultMode: 0400
```

---

## RBAC and Security

### Role-Based Access Control (RBAC)

Define user permissions with Roles and RoleBindings.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: app-service-account
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: app-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["get", "list"]
- apiGroups: ["apps"]
  resources: ["deployments"]
  verbs: ["get", "list", "update"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: app-rolebinding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: app-role
subjects:
- kind: ServiceAccount
  name: app-service-account
  namespace: default
```

### ClusterRole for Cluster-wide Permissions

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: monitoring-role
rules:
- apiGroups: [""]
  resources: ["nodes", "pods", "services", "endpoints"]
  verbs: ["get", "list", "watch"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets", "daemonsets"]
  verbs: ["get", "list", "watch"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: monitoring-rolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: monitoring-role
subjects:
- kind: ServiceAccount
  name: monitoring-sa
  namespace: monitoring
```

### Pod Security Standards

Control Pod security policies.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsNonRoot: true
    runAsUser: 1000
    fsGroup: 2000
    seccompProfile:
      type: RuntimeDefault
  containers:
  - name: app
    image: myapp:v1.0
    securityContext:
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      runAsUser: 1000
      capabilities:
        drop:
        - ALL
        add:
        - NET_BIND_SERVICE
    volumeMounts:
    - name: tmp
      mountPath: /tmp
    - name: var-run
      mountPath: /var/run
  volumes:
  - name: tmp
    emptyDir: {}
  - name: var-run
    emptyDir: {}
```

---

## Monitoring and Logging

### Prometheus ServiceMonitor

Scrape metrics from applications.

```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: app-monitor
spec:
  selector:
    matchLabels:
      app: myapp
  endpoints:
  - port: metrics
    interval: 30s
    path: /metrics
```

### Application with Metrics

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: metrics-app
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
      annotations:
        prometheus.io/scrape: "true"
        prometheus.io/port: "8080"
        prometheus.io/path: "/metrics"
    spec:
      containers:
      - name: app
        image: myapp:v1.0
        ports:
        - name: metrics
          containerPort: 8080
        - name: http
          containerPort: 8081
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

### Centralized Logging with Filebeat

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
data:
  filebeat.yml: |
    filebeat.inputs:
    - type: container
      paths:
        - '/var/lib/docker/containers/*/*.log'
    processors:
      - add_kubernetes_metadata:
          in_cluster: true
    output.elasticsearch:
      hosts: ["elasticsearch:9200"]
      index: "k8s-logs-%{+yyyy.MM.dd}"
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      serviceAccountName: filebeat
      containers:
      - name: filebeat
        image: docker.elastic.co/beats/filebeat:8.0.0
        volumeMounts:
        - name: config
          mountPath: /etc/filebeat
        - name: varlog
          mountPath: /var/log
          readOnly: true
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: config
        configMap:
          name: filebeat-config
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```

---

## Helm Package Manager

### Installing Helm

```bash
# macOS
brew install helm

# Linux
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Windows
choco install kubernetes-helm
```

### Creating a Helm Chart

```bash
helm create myapp
cd myapp
```

Chart structure:
```
myapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── _helpers.tpl
└── charts/
```

### values.yaml

```yaml
replicaCount: 2

image:
  repository: myapp
  tag: v1.0
  pullPolicy: IfNotPresent

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: nginx
  hosts:
  - host: myapp.example.com
    paths:
    - path: /
      pathType: Prefix

resources:
  limits:
    cpu: 500m
    memory: 512Mi
  requests:
    cpu: 250m
    memory: 256Mi

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 80
```

### Template (deployment.yaml)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "myapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      {{- include "myapp.selectorLabels" . | nindent 6 }}
  template:
    metadata:
      labels:
        {{- include "myapp.selectorLabels" . | nindent 8 }}
    spec:
      containers:
      - name: {{ .Chart.Name }}
        image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
        imagePullPolicy: {{ .Values.image.pullPolicy }}
        ports:
        - name: http
          containerPort: {{ .Values.service.targetPort }}
        resources:
          {{- toYaml .Values.resources | nindent 12 }}
```

### Helm Commands

```bash
# Validate chart
helm lint myapp

# Install chart
helm install myapp ./myapp --values values.yaml

# Upgrade release
helm upgrade myapp ./myapp

# List releases
helm list

# Rollback release
helm rollback myapp 1

# Uninstall release
helm uninstall myapp

# Package chart
helm package myapp
```

---

## Custom Resource Definitions (CRDs)

Define custom Kubernetes objects.

```yaml
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: databases.example.com
spec:
  group: example.com
  names:
    kind: Database
    plural: databases
  scope: Namespaced
  versions:
  - name: v1
    served: true
    storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              type:
                type: string
                enum: ["mysql", "postgres", "mongodb"]
              version:
                type: string
              storage:
                type: string
              replicas:
                type: integer
                minimum: 1
                maximum: 5
```

### Using a Custom Resource

```yaml
apiVersion: example.com/v1
kind: Database
metadata:
  name: production-db
spec:
  type: postgres
  version: "14"
  storage: 100Gi
  replicas: 3
```

---

## Operators and Controllers

### Operator Pattern Example

An operator watches custom resources and manages applications automatically.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: database-operator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: database-operator
  template:
    metadata:
      labels:
        app: database-operator
    spec:
      serviceAccountName: database-operator
      containers:
      - name: operator
        image: database-operator:v1.0
        env:
        - name: WATCH_NAMESPACE
          value: ""
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: OPERATOR_NAME
          value: database-operator
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: database-operator
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: database-operator
rules:
- apiGroups: ["example.com"]
  resources: ["databases"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
- apiGroups: [""]
  resources: ["services", "configmaps", "secrets"]
  verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: database-operator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: database-operator
subjects:
- kind: ServiceAccount
  name: database-operator
```

---

## Best Practices

### 1. Resource Management

Always set requests and limits:
```yaml
resources:
  requests:
    memory: "128Mi"
    cpu: "100m"
  limits:
    memory: "256Mi"
    cpu: "500m"
```

### 2. Health Checks

Implement comprehensive health checks:
```yaml
livenessProbe:
  httpGet:
    path: /health
    port: 8080
  initialDelaySeconds: 15
  periodSeconds: 20
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 3

readinessProbe:
  httpGet:
    path: /ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
  timeoutSeconds: 1
  successThreshold: 1
  failureThreshold: 3
```

### 3. Graceful Shutdown

```yaml
lifecycle:
  preStop:
    exec:
      command: ["/bin/sh", "-c", "sleep 15"]
terminationGracePeriodSeconds: 30
```

### 4. Horizontal Pod Autoscaling (HPA)

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: myapp-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  minReplicas: 2
  maxReplicas: 10
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 70
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 80
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 300
      policies:
      - type: Percent
        value: 50
        periodSeconds: 15
    scaleUp:
      stabilizationWindowSeconds: 0
      policies:
      - type: Percent
        value: 100
        periodSeconds: 15
```

### 5. Namespace Management

```bash
# Create namespace with labels
kubectl create namespace production
kubectl label namespace production env=production tier=backend

# Apply resource quotas
kubectl apply -f - <<EOF
apiVersion: v1
kind: ResourceQuota
metadata:
  name: production-quota
  namespace: production
spec:
  hard:
    requests.cpu: "100"
    requests.memory: "200Gi"
    limits.cpu: "200"
    limits.memory: "400Gi"
    pods: "1000"
    services.loadbalancers: "2"
EOF
```

### 6. Network Policies

Start with default deny, then explicitly allow traffic:
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```

### 7. Image Security

- Use specific image tags (never `latest`)
- Scan images for vulnerabilities
- Use private registries
- Implement image pull secrets

```yaml
imagePullSecrets:
- name: registry-credentials
```

### 8. GitOps with ArgoCD

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: myapp
  namespace: argocd
spec:
  project: default
  source:
    repoURL: https://github.com/myorg/myrepo
    targetRevision: HEAD
    path: k8s/
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

### 9. Logging Best Practices

- Use structured logging (JSON)
- Include request IDs for tracing
- Use appropriate log levels
- Aggregate logs centrally

```yaml
env:
- name: LOG_LEVEL
  value: "INFO"
- name: LOG_FORMAT
  value: "json"
```

### 10. Backup and Disaster Recovery

```bash
# Backup cluster state
kubectl get all --all-namespaces -o yaml > cluster-backup.yaml

# Backup specific resources
kubectl get pvc --all-namespaces -o yaml > pvc-backup.yaml

# Using velero for automated backups
velero backup create cluster-backup
```

---

## Advanced kubectl Commands

```bash
# Watch resource changes
kubectl get pods --watch

# Get resource in different formats
kubectl get pod myapp -o yaml
kubectl get pod myapp -o json
kubectl get pod myapp -o wide

# Debugging
kubectl describe pod <pod-name>
kubectl logs <pod-name> -c <container-name>
kubectl logs <pod-name> --previous
kubectl exec -it <pod-name> -- /bin/bash

# Port forwarding
kubectl port-forward pod/myapp 8080:8080
kubectl port-forward service/myapp 8080:80

# Copy files
kubectl cp <namespace>/<pod>:/path/to/file ./local/path
kubectl cp ./local/file <namespace>/<pod>:/path/to/file

# Debugging node
kubectl debug node/nodename -it --image=ubuntu

# Apply and wait
kubectl apply -f deployment.yaml && kubectl wait --for=condition=available --timeout=300s deployment/myapp
```

---

## Troubleshooting Guide

### Pod Stuck in Pending

```bash
# Check events
kubectl describe pod <pod-name>

# Check node resources
kubectl top nodes
kubectl describe node <node-name>
```

### Pod CrashLoopBackOff

```bash
# View logs
kubectl logs <pod-name>
kubectl logs <pod-name> --previous

# Check resource limits
kubectl describe pod <pod-name>
```

### Service Not Accessible

```bash
# Check service
kubectl get service <service-name>
kubectl describe service <service-name>

# Test connectivity
kubectl run debug --rm -it --image=ubuntu -- sh
# Inside debug pod:
curl <service-name>:<port>
```

---

## Resources

- [Kubernetes Official Documentation](https://kubernetes.io/docs/)
- [Kubernetes API Reference](https://kubernetes.io/docs/reference/generated/kubernetes-api/)
- [Helm Documentation](https://helm.sh/docs/)
- [Prometheus Operator](https://prometheus-operator.dev/)
- [Istio Documentation](https://istio.io/latest/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)

---

**Last Updated**: 2026-04-28

**Continue Your Kubernetes Journey!** 🚀
