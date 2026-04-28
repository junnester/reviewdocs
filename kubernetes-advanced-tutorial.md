# Kubernetes Advanced Tutorial

## Table of Contents
1. [Introduction](#introduction)
2. [Multi-Cluster Architecture](#multi-cluster-architecture)
3. [Advanced Scheduling](#advanced-scheduling)
4. [Service Mesh Deep Dive](#service-mesh-deep-dive)
5. [Advanced Security](#advanced-security)
6. [Performance Optimization](#performance-optimization)
7. [Advanced Storage](#advanced-storage)
8. [Kubernetes Extensibility](#kubernetes-extensibility)
9. [Advanced Monitoring and Observability](#advanced-monitoring-and-observability)
10. [GitOps and CI/CD Integration](#gitops-and-cicd-integration)
11. [Kubernetes Internals](#kubernetes-internals)
12. [Production Hardening](#production-hardening)

---

## Introduction

This advanced tutorial covers production-grade Kubernetes deployment patterns, architectural decisions, and operational excellence. Topics include multi-cluster management, advanced scheduling algorithms, service mesh implementations, security hardening, and performance tuning.

### Prerequisites

- Deep understanding of Kubernetes core concepts
- Experience with intermediate workloads (StatefulSets, DaemonSets, Jobs)
- Proficiency with Helm and package management
- Knowledge of networking and Linux fundamentals
- Experience with container registries and CI/CD pipelines

---

## Multi-Cluster Architecture

### Cluster Federation with KubeFed

Manage multiple clusters as a single logical entity.

```yaml
apiVersion: core.kubefed.io/v1beta1
kind: KubeFedCluster
metadata:
  name: cluster-us-east
  namespace: kube-federation-system
spec:
  apiEndpoint: https://cluster-us-east.example.com
  caBundle: LS0tLS1CRUdJTi... # base64 encoded
  secretRef:
    name: cluster-us-east-secret
---
apiVersion: core.kubefed.io/v1beta1
kind: KubeFedCluster
metadata:
  name: cluster-eu-west
  namespace: kube-federation-system
spec:
  apiEndpoint: https://cluster-eu-west.example.com
  caBundle: LS0tLS1CRUdJTi...
  secretRef:
    name: cluster-eu-west-secret
```

### Federated Deployment

```yaml
apiVersion: apps.kubefed.io/v1beta1
kind: FederatedDeployment
metadata:
  name: global-app
  namespace: production
spec:
  template:
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
          containers:
          - name: app
            image: myapp:v2.0
            resources:
              requests:
                cpu: 500m
                memory: 512Mi
              limits:
                cpu: 1000m
                memory: 1Gi
  placement:
    clusters:
    - name: cluster-us-east
      weight: 60
    - name: cluster-eu-west
      weight: 40
  overrides:
  - clusterName: cluster-us-east
    clusterOverrides:
    - path: /spec/template/spec/containers/0/image
      value: myapp:v2.0-us
  - clusterName: cluster-eu-west
    clusterOverrides:
    - path: /spec/template/spec/containers/0/image
      value: myapp:v2.0-eu
```

### Multi-Cluster Ingress

```yaml
apiVersion: net.gke.io/v1
kind: MultiClusterIngress
metadata:
  name: global-ingress
spec:
  template:
    spec:
      ingressClassName: gce
      rules:
      - host: api.example.com
        http:
          paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: api-service
                port:
                  number: 80
  clusters:
  - name: cluster-us-east
  - name: cluster-eu-west
```

---

## Advanced Scheduling

### Priority Classes

Define priority levels for critical workloads.

```yaml
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: high-priority
value: 1000
globalDefault: false
description: "High priority for critical applications"
---
apiVersion: scheduling.k8s.io/v1
kind: PriorityClass
metadata:
  name: system-critical
value: 2000
globalDefault: false
preemptionPolicy: PreemptLowerPriority
description: "System-critical priority"
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: critical-app
spec:
  template:
    spec:
      priorityClassName: high-priority
      containers:
      - name: app
        image: critical-app:v1.0
```

### Pod Priority and Preemption

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: critical-pod
spec:
  priorityClassName: system-critical
  preemptionPolicy: PreemptLowerPriority
  containers:
  - name: critical-app
    image: critical-app:v1.0
    resources:
      requests:
        cpu: 4000m
        memory: 8Gi
      limits:
        cpu: 8000m
        memory: 16Gi
```

### Custom Scheduler

Deploy a custom scheduler for specialized workloads.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: custom-scheduler
  namespace: kube-system
spec:
  replicas: 2
  selector:
    matchLabels:
      app: custom-scheduler
  template:
    metadata:
      labels:
        app: custom-scheduler
    spec:
      priorityClassName: system-cluster-critical
      serviceAccountName: custom-scheduler
      containers:
      - name: scheduler
        image: custom-scheduler:v1.0
        command:
        - custom-scheduler
        - --kubeconfig=/etc/kubernetes/scheduler.conf
        - --leader-elect=true
        - --port=10282
        volumeMounts:
        - name: kubeconfig
          mountPath: /etc/kubernetes
      volumes:
      - name: kubeconfig
        hostPath:
          path: /etc/kubernetes
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: custom-scheduler
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: custom-scheduler
rules:
- apiGroups: [""]
  resources: ["events"]
  verbs: ["create", "patch"]
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/binding"]
  verbs: ["create"]
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "watch"]
```

### GPU Scheduling

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: gpu-workload
spec:
  replicas: 1
  selector:
    matchLabels:
      workload: gpu
  template:
    metadata:
      labels:
        workload: gpu
    spec:
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: accelerator
                operator: In
                values:
                - nvidia-tesla-v100
      containers:
      - name: gpu-app
        image: cuda-app:v1.0
        resources:
          limits:
            nvidia.com/gpu: 4
          requests:
            nvidia.com/gpu: 4
            memory: 32Gi
            cpu: 8000m
```

---

## Service Mesh Deep Dive

### Istio VirtualService with Advanced Routing

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: advanced-routing
spec:
  hosts:
  - api.example.com
  http:
  - match:
    - uri:
        prefix: /api/v1
      withoutHeaders:
        user-agent:
          exact: bot
    route:
    - destination:
        host: api-v1
        port:
          number: 8080
        subset: stable
      weight: 90
    - destination:
        host: api-v1
        port:
          number: 8080
        subset: canary
      weight: 10
    timeout: 30s
    retries:
      attempts: 3
      perTryTimeout: 10s
  - match:
    - uri:
        prefix: /api/v2
    route:
    - destination:
        host: api-v2
        port:
          number: 8080
        subset: stable
  - route:
    - destination:
        host: api-default
        port:
          number: 8080
```

### Destination Rule with Load Balancing

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: api-destination-rule
spec:
  host: api-service
  trafficPolicy:
    connectionPool:
      http:
        http1MaxPendingRequests: 100
        maxRequestsPerConnection: 2
        h2UpgradePolicy: UPGRADE
      tcp:
        maxConnections: 100
    loadBalancer:
      simple: ROUND_ROBIN
      consistentHash:
        httpCookie:
          name: session-id
          ttl: 1h
    outlierDetection:
      consecutive5xxErrors: 5
      interval: 30s
      baseEjectionTime: 30s
      maxEjectionPercent: 50
      minRequestVolume: 5
      splitExternalLocalOriginErrors: true
  subsets:
  - name: stable
    labels:
      version: stable
    trafficPolicy:
      connectionPool:
        http:
          http1MaxPendingRequests: 50
  - name: canary
    labels:
      version: canary
```

### PeerAuthentication and AuthorizationPolicy

```yaml
apiVersion: security.istio.io/v1beta1
kind: PeerAuthentication
metadata:
  name: default
spec:
  mtls:
    mode: STRICT
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: api-policy
spec:
  selector:
    matchLabels:
      app: api
  rules:
  - from:
    - source:
        principals: ["cluster.local/ns/production/sa/frontend"]
    to:
    - operation:
        methods: ["GET", "POST"]
        paths: ["/api/v1/*"]
  - from:
    - source:
        namespaces: ["monitoring"]
    to:
    - operation:
        ports: ["9090"]
        paths: ["/metrics"]
  - when:
    - key: request.auth.claims[groups]
      values: ["admin"]
    to:
    - operation:
        methods: ["*"]
```

### RequestAuthentication with JWT

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
spec:
  jwtRules:
  - issuer: "https://auth.example.com"
    jwksUri: "https://auth.example.com/.well-known/jwks.json"
    audiences: "api.example.com"
    forwardOriginalToken: true
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: jwt-policy
spec:
  rules:
  - from:
    - source:
        requestPrincipals: ["*"]
    to:
    - operation:
        methods: ["*"]
```

---

## Advanced Security

### Pod Security Policies

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted-psp
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: 'runtime/default'
    apparmor.security.beta.kubernetes.io/allowedProfileNames: 'runtime/default'
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
  - ALL
  allowedCapabilities:
  - NET_BIND_SERVICE
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
  - 'downwardAPI'
  - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  supplementalGroups:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
  readOnlyRootFilesystem: true
  seLinux:
    rule: 'MustRunAs'
    seLinuxOptions:
      level: "s0:c123,c456"
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: restricted-psp-user
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs: ['use']
  resourceNames:
  - restricted-psp
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: restricted-psp-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: restricted-psp-user
subjects:
- kind: Group
  name: system:serviceaccounts
```

### Network Segmentation with Network Policies

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-by-default
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      tier: backend
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          tier: frontend
    - namespaceSelector:
        matchLabels:
          name: production
    ports:
    - protocol: TCP
      port: 8080
---
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-external-dns
spec:
  podSelector: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          name: kube-system
    ports:
    - protocol: UDP
      port: 53
```

### RBAC with Fine-Grained Permissions

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer-role
  namespace: development
rules:
# Pod management - read only
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
- apiGroups: [""]
  resources: ["pods/log"]
  verbs: ["get"]
# Deployment management
- apiGroups: ["apps"]
  resources: ["deployments", "statefulsets"]
  verbs: ["get", "list", "watch", "create", "update", "patch"]
# Resource limits - cannot edit
- apiGroups: [""]
  resources: ["resourcequotas"]
  verbs: ["get", "list"]
  resourceNames: ["development-quota"]
# Cannot delete anything
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["delete"]
  # Empty resourceNames means cannot delete
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: admin-role
  namespace: production
rules:
- apiGroups: ["*"]
  resources: ["*"]
  verbs: ["*"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: dev-team-binding
  namespace: development
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer-role
subjects:
- kind: Group
  name: dev-team@example.com
  apiGroup: rbac.authorization.k8s.io
```

### Admission Controllers

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: ValidatingWebhookConfiguration
metadata:
  name: image-policy-webhook
webhooks:
- name: image-policy.example.com
  admissionReviewVersions: ["v1"]
  clientConfig:
    service:
      name: image-policy-service
      namespace: kube-system
      path: "/validate"
    caBundle: LS0tLS1CRUdJTi... # base64 encoded
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: [""]
    apiVersions: ["v1"]
    resources: ["pods"]
  namespaceSelector:
    matchLabels:
      enforce-image-policy: "true"
  sideEffects: None
  timeoutSeconds: 5
  failurePolicy: Fail
```

---

## Performance Optimization

### Resource Optimization

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: optimized-app
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
      # Node affinity for optimal placement
      affinity:
        nodeAffinity:
          preferredDuringSchedulingIgnoredDuringExecution:
          - weight: 100
            preference:
              matchExpressions:
              - key: cpu-type
                operator: In
                values:
                - high-performance
      containers:
      - name: app
        image: myapp:v1.0
        # Resource optimization
        resources:
          requests:
            cpu: 500m
            memory: 512Mi
          limits:
            cpu: 1000m
            memory: 1Gi
        # Lifecycle optimization
        lifecycle:
          preStop:
            exec:
              command: ["/bin/sh", "-c", "sleep 15"]
        # Startup probe for slow apps
        startupProbe:
          httpGet:
            path: /ready
            port: 8080
          failureThreshold: 30
          periodSeconds: 10
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
          timeoutSeconds: 5
          failureThreshold: 3
        readinessProbe:
          httpGet:
            path: /ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
          timeoutSeconds: 3
          failureThreshold: 2
      terminationGracePeriodSeconds: 30
```

### Cluster Autoscaling

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: ClusterAutoscaler
metadata:
  name: cluster-autoscaler
  namespace: kube-system
spec:
  scaleDownEnabled: true
  scaleDownDelayAfterAdd: 10m
  scaleDownDelayAfterDelete: 10s
  scaleDownDelayAfterFailure: 3m
  scaleDownUnneededTime: 10m
  scaleDownUnreadyTime: 20m
  maxNodeProvisionTime: 15m
  maxTotalUnreadyPercentage: 45
  okTotalUnreadyCount: 3
```

### Vertical Pod Autoscaling

```yaml
apiVersion: autoscaling.k8s.io/v1
kind: VerticalPodAutoscaler
metadata:
  name: vpa-example
spec:
  targetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: myapp
  updatePolicy:
    updateMode: "Auto"
  resourcePolicy:
    containerPolicies:
    - containerName: "*"
      minAllowed:
        cpu: 100m
        memory: 128Mi
      maxAllowed:
        cpu: 2000m
        memory: 2Gi
      controlledResources: ["cpu", "memory"]
```

---

## Advanced Storage

### Storage with Local Volumes

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/local
volumeBindingMode: WaitForFirstConsumer
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 100Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: local-storage
  local:
    path: /mnt/data
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - node-1
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: local-storage
  resources:
    requests:
      storage: 50Gi
```

### Snapshot and Restore

```yaml
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshotClass
metadata:
  name: snapshot-class
driver: ebs.csi.aws.com
deletionPolicy: Delete
---
apiVersion: snapshot.storage.k8s.io/v1
kind: VolumeSnapshot
metadata:
  name: pvc-snapshot
spec:
  volumeSnapshotClassName: snapshot-class
  source:
    persistentVolumeClaimName: my-pvc
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: restored-pvc
spec:
  dataSource:
    name: pvc-snapshot
    kind: VolumeSnapshot
    apiGroup: snapshot.storage.k8s.io
  accessModes:
  - ReadWriteOnce
  storageClassName: ebs-sc
  resources:
    requests:
      storage: 50Gi
```

---

## Kubernetes Extensibility

### Custom Scheduler Framework

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: framework-scheduler
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: framework-scheduler
  template:
    metadata:
      labels:
        app: framework-scheduler
    spec:
      serviceAccountName: framework-scheduler
      containers:
      - name: scheduler
        image: framework-scheduler:v1.0
        command:
        - /scheduler
        - --config=/etc/kubernetes/scheduler-config.yaml
        volumeMounts:
        - name: config
          mountPath: /etc/kubernetes
      volumes:
      - name: config
        configMap:
          name: scheduler-config
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: scheduler-config
  namespace: kube-system
data:
  scheduler-config.yaml: |
    apiVersion: kubescheduler.config.k8s.io/v1
    kind: KubeSchedulerConfiguration
    profiles:
    - schedulerName: custom-scheduler
      plugins:
        preFilter:
          enabled:
          - name: CustomPlugin
        filter:
          enabled:
          - name: CustomPlugin
        preScore:
          enabled:
          - name: CustomPlugin
        score:
          enabled:
          - name: CustomPlugin
```

### Webhook Extension

```yaml
apiVersion: admissionregistration.k8s.io/v1
kind: MutatingWebhookConfiguration
metadata:
  name: mutation-webhook
webhooks:
- name: mutation.example.com
  admissionReviewVersions: ["v1"]
  clientConfig:
    service:
      name: mutation-service
      namespace: kube-system
      path: "/mutate"
    caBundle: LS0tLS1CRUdJTi...
  rules:
  - operations: ["CREATE", "UPDATE"]
    apiGroups: ["apps"]
    apiVersions: ["v1"]
    resources: ["deployments"]
  namespaceSelector:
    matchLabels:
      mutation: enabled
  sideEffects: None
  failurePolicy: Ignore
---
apiVersion: v1
kind: Service
metadata:
  name: mutation-service
  namespace: kube-system
spec:
  ports:
  - port: 443
    targetPort: 8443
  selector:
    app: mutation-webhook
```

---

## Advanced Monitoring and Observability

### Comprehensive Prometheus Setup

```yaml
apiVersion: monitoring.coreos.com/v1
kind: Prometheus
metadata:
  name: prometheus
  namespace: monitoring
spec:
  replicas: 2
  retention: 30d
  retentionSize: 50Gi
  storageSpec:
    volumeClaimTemplate:
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 100Gi
  serviceMonitorSelector: {}
  ruleSelector: {}
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
              - prometheus
          topologyKey: kubernetes.io/hostname
---
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  name: application-rules
spec:
  groups:
  - name: application.rules
    interval: 30s
    rules:
    - alert: HighErrorRate
      expr: |
        (
          sum(rate(http_requests_total{status=~"5.."}[5m]))
          /
          sum(rate(http_requests_total[5m]))
        ) > 0.05
      for: 5m
      labels:
        severity: critical
      annotations:
        summary: "High error rate detected"
    - alert: PodCrashLooping
      expr: |
        rate(kube_pod_container_status_restarts_total[15m]) > 0
      for: 5m
      labels:
        severity: warning
```

### Distributed Tracing with Jaeger

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: jaeger-collector
  namespace: tracing
spec:
  replicas: 2
  selector:
    matchLabels:
      app: jaeger-collector
  template:
    metadata:
      labels:
        app: jaeger-collector
    spec:
      containers:
      - name: jaeger
        image: jaegertracing/jaeger:latest
        ports:
        - name: otlp-grpc
          containerPort: 4317
        - name: otlp-http
          containerPort: 4318
        env:
        - name: COLLECTOR_OTLP_ENABLED
          value: "true"
        - name: SPAN_STORAGE_TYPE
          value: elasticsearch
        - name: ES_SERVER_URLS
          value: http://elasticsearch:9200
---
apiVersion: v1
kind: Service
metadata:
  name: jaeger-collector
  namespace: tracing
spec:
  selector:
    app: jaeger-collector
  ports:
  - name: otlp-grpc
    port: 4317
    targetPort: 4317
  - name: otlp-http
    port: 4318
    targetPort: 4318
  type: ClusterIP
```

### OpenTelemetry Instrumentation

```yaml
apiVersion: opentelemetry.io/v1alpha1
kind: Instrumentation
metadata:
  name: python-instrumentation
  namespace: production
spec:
  propagators:
  - jaeger
  - b3
  sampler:
    type: parentbased_always_on
  python:
    env:
    - name: OTEL_EXPORTER_OTLP_ENDPOINT
      value: http://jaeger-collector:4317
```

---

## GitOps and CI/CD Integration

### ArgoCD Application with Advanced Configuration

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: production-app
  namespace: argocd
spec:
  project: production
  source:
    repoURL: https://github.com/myorg/myrepo
    targetRevision: main
    path: k8s/production
    plugin:
      name: kustomize
      env:
      - name: KUSTOMIZE_IMAGE_TAG
        value: "v1.0.0"
  destination:
    server: https://kubernetes.default.svc
    namespace: production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
      allowEmpty: false
    syncOptions:
    - CreateNamespace=true
    - PrunePropagationPolicy=background
    - RespectIgnoreDifferences=true
    retry:
      limit: 5
      backoff:
        duration: 5s
        factor: 2
        maxDuration: 3m
  revisionHistoryLimit: 10
```

### Kustomize Overlays for Multi-Environment Deployment

```
k8s/
├── base/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── configmap.yaml
│   └── kustomization.yaml
├── overlays/
│   ├── dev/
│   │   ├── kustomization.yaml
│   │   ├── patches.yaml
│   │   └── config/
│   ├── staging/
│   │   ├── kustomization.yaml
│   │   ├── patches.yaml
│   │   └── config/
│   └── production/
│       ├── kustomization.yaml
│       ├── patches.yaml
│       └── config/
```

Base kustomization.yaml:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: default
commonLabels:
  app: myapp
resources:
- deployment.yaml
- service.yaml
- configmap.yaml
```

Production overlay:
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: production
namePrefix: prod-
commonLabels:
  environment: production
commonAnnotations:
  config.kubernetes.io/version: "1.0"
replicas:
- name: myapp
  count: 3
patchesStrategicMerge:
- |-
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: myapp
  spec:
    template:
      spec:
        containers:
        - name: app
          resources:
            requests:
              cpu: 1000m
              memory: 1Gi
            limits:
              cpu: 2000m
              memory: 2Gi
configMapGenerator:
- name: app-config
  behavior: merge
  files:
  - config/production.yaml
```

### Flux CD Configuration

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta2
kind: GitRepository
metadata:
  name: myapp-repo
  namespace: flux-system
spec:
  interval: 1m
  url: https://github.com/myorg/myrepo
  ref:
    branch: main
  secretRef:
    name: github-credentials
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: myapp-production
  namespace: flux-system
spec:
  interval: 10m
  prune: true
  sourceRef:
    kind: GitRepository
    name: myapp-repo
  path: ./k8s/overlays/production
  validation: client
  postBuild:
    substitute:
      ENV: production
      REPLICAS: "3"
  healthChecks:
  - apiVersion: apps/v1
    kind: Deployment
    name: myapp
    namespace: production
  timeout: 5m0s
```

---

## Kubernetes Internals

### API Server Optimization

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kube-apiserver-config
  namespace: kube-system
data:
  apiserver.conf: |
    # Request/response compression
    --enable-compression=true
    # Audit logging
    --audit-log-path=/var/log/kubernetes/audit.log
    --audit-policy-file=/etc/kubernetes/audit-policy.yaml
    # Performance tuning
    --max-requests-inflight=3000
    --max-mutating-requests-inflight=1000
    --request-timeout=1m
    # Enable runtime classes
    --runtime-config=api/all=true
    # RBAC mode
    --authorization-mode=RBAC,Node
```

### etcd Optimization

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: etcd
  namespace: kube-system
spec:
  containers:
  - name: etcd
    image: registry.k8s.io/etcd:3.5.0
    command:
    - etcd
    - --name=etcd-0
    - --listen-client-urls=https://127.0.0.1:2379
    - --advertise-client-urls=https://127.0.0.1:2379
    # Performance tuning
    - --quota-backend-bytes=8589934592
    - --max-request-bytes=33554432
    - --auto-compaction-retention=1
    - --snapshot-count=10000
    env:
    - name: ETCD_HEARTBEAT_INTERVAL
      value: "100"
    - name: ETCD_ELECTION_TIMEOUT
      value: "1000"
```

### Controller Manager Tuning

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: controller-manager-config
  namespace: kube-system
data:
  manager.conf: |
    --controller-workers=50
    --horizontal-pod-autoscaler-sync-period=30s
    --horizontal-pod-autoscaler-cpu-initialization-period=60s
    --node-cidr-mask-size=24
    --node-eviction-rate=0.1
    --secondary-node-eviction-rate=0.01
    --leader-elect=true
    --leader-elect-resource-lock=leases
```

---

## Production Hardening

### Cluster Hardening Checklist

```yaml
# 1. API Server Security
apiVersion: v1
kind: Pod
metadata:
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - name: kube-apiserver
    command:
    - kube-apiserver
    - --encryption-provider-config=/etc/kubernetes/secrets/encryption.yaml
    - --tls-cipher-suites=TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384
    - --audit-log-path=/var/log/audit.log
    - --audit-log-maxage=30
    - --audit-log-maxbackup=10
    - --audit-log-maxsize=100
---
# 2. Kubelet Security
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubelet-config
  namespace: kube-system
data:
  kubelet.conf: |
    {
      "featureGates": {
        "RotateKubeletServerCertificate": true
      },
      "serverTLSBootstrap": true,
      "readOnlyPort": 0,
      "eventRecordQPS": 0,
      "protectKernelDefaults": true,
      "makeIPTablesUtilChains": true,
      "allowedUnsafeSysctls": null
    }
---
# 3. Network Policy Default Deny
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress
---
# 4. Pod Security Standards
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: restricted
spec:
  privileged: false
  allowPrivilegeEscalation: false
  requiredDropCapabilities:
  - ALL
  volumes:
  - 'configMap'
  - 'emptyDir'
  - 'projected'
  - 'secret'
  - 'downwardAPI'
  - 'persistentVolumeClaim'
  hostNetwork: false
  hostIPC: false
  hostPID: false
  runAsUser:
    rule: 'MustRunAsNonRoot'
  fsGroup:
    rule: 'RunAsAny'
```

### Backup and Disaster Recovery

```yaml
apiVersion: velero.io/v1
kind: Backup
metadata:
  name: daily-backup
  namespace: velero
spec:
  includedNamespaces:
  - '*'
  excludedNamespaces:
  - kube-system
  - kube-node-lease
  includedResources:
  - '*'
  excludedResources:
  - nodes
  - events
  - events.events.k8s.io
  storageLocation: default
  volumeSnapshotLocation: default
  ttl: 720h
  schedule: '0 2 * * *'
---
apiVersion: velero.io/v1
kind: Schedule
metadata:
  name: daily-backup
  namespace: velero
spec:
  schedule: '0 2 * * *'
  template:
    includedNamespaces:
    - '*'
    storageLocation: default
    ttl: 720h
```

### Cost Optimization

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: cost-optimization-policy
  namespace: default
data:
  policy: |
    # Implement resource quotas per namespace
    limits:
      cpu: 100
      memory: 200Gi
    requests:
      cpu: 50
      memory: 100Gi
    # Reserved resources per node
    kubelet_reserved:
      cpu: 100m
      memory: 128Mi
    # System reserved resources
    system_reserved:
      cpu: 100m
      memory: 128Mi
    # Eviction thresholds
    eviction_hard:
      memory.available: 5%
      nodefs.available: 5%
```

---

## Troubleshooting Advanced Issues

### Performance Debugging

```bash
# Check API server latency
kubectl get --raw /metrics | grep apiserver_request_duration_seconds

# Monitor etcd latency
kubectl exec -it -n kube-system etcd-master -- \
  etcdctl --endpoints=localhost:2379 \
  member list

# Analyze controller manager
kubectl logs -n kube-system -l component=kube-controller-manager

# Check scheduler decisions
kubectl get events -A --sort-by='.lastTimestamp'
```

### Network Debugging

```bash
# Verify CNI plugin
kubectl get daemonset -n kube-system

# Check network policies
kubectl get networkpolicies --all-namespaces

# Test pod connectivity
kubectl run debug --image=ubuntu --rm -it -- \
  bash -c "apt update && apt install -y curl && \
  curl http://service-name:port"

# Capture traffic
kubectl sniff <pod-name> -n <namespace> | wireshark -k -i -
```

### API Server Issues

```bash
# Check API server logs
kubectl logs -n kube-system -l component=kube-apiserver

# Monitor API server metrics
kubectl get --raw /metrics | grep apiserver | head -20

# Check audit logs
tail -f /var/log/kubernetes/audit.log
```

---

## Resources

- [Kubernetes Advanced Configuration](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/)
- [Kubernetes Security Best Practices](https://kubernetes.io/docs/concepts/security/)
- [Istio Documentation](https://istio.io/latest/docs/)
- [ArgoCD Documentation](https://argo-cd.readthedocs.io/)
- [Kubernetes Performance Tuning](https://kubernetes.io/docs/tasks/debug-application-cluster/resource-metrics-pipeline/)
- [Velero Backup & Restore](https://velero.io/docs/)
- [KubeFed Documentation](https://github.com/kubernetes-sigs/kubefed/wiki)

---

**Last Updated**: 2026-04-28

**Master Kubernetes Production Operations!** 🚀
