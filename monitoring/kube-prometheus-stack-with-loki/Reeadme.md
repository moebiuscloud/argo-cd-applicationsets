# Monitoring - Enhanced Kube-Prometheus-Stack + Loki

**Complete observability stack with Prometheus, Grafana, Loki, and AlertManager - all running as non-root users.**

## 🏷️ Required Labels

Your ArgoCD clusters need this label:
```bash
kubectl label secret my-cluster monitoring=enabled
```

## 📦 What You Get

### **Metrics Stack**
- **Prometheus** - Metrics collection and storage (30d retention, 50GB)
- **Grafana** - Pre-configured with popular dashboards + Loki data source
- **AlertManager** - Alert routing and notifications (5GB storage)
- **Node Exporter** - Host metrics collection
- **Kube State Metrics** - Kubernetes object metrics

### **Logging Stack**
- **Loki** - Log aggregation and storage (10GB persistence)
- **Promtail** - Log collection from all Kubernetes pods
- **Gateway** - Load balancing for Loki requests

### **🔐 Security Features**
- **Non-root execution**: All components run as non-root users
- **Minimal privileges**: Capabilities dropped, privilege escalation blocked
- **Seccomp profiles**: Runtime default security profiles
- **Read-only filesystems**: Where possible for enhanced security

## 🚀 Deploy

```bash
# 1. Label your cluster
kubectl label secret my-cluster monitoring=enabled

# 2. Apply the ApplicationSet
kubectl apply -f basic/applicationset.yaml

# 3. Wait for deployment (3-5 minutes for full stack)
kubectl get applications -n argocd | grep monitoring

# 4. Check all pods are running
kubectl get pods -n monitoring
```

## 🔐 Access Grafana

**Get the auto-generated password:**
```bash
kubectl get secret prometheus-grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 --decode
```

**Access the UI:**
```bash
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
# Visit: http://localhost:3000
# User: admin
# Password: (from command above)
```

## 📊 What's Included

**Pre-configured Grafana dashboards:**
- **Metrics:**
  - Kubernetes Cluster Overview (ID: 7249)
  - Kubernetes Pods (ID: 6417)  
  - Node Exporter Full (ID: 1860)
- **Logs:**
  - Loki Logs Dashboard (ID: 13639)

**Pre-configured Data Sources:**
- **Prometheus** - Metrics from the monitoring stack
- **Loki** - Logs from all Kubernetes pods and events

**Storage:**
- Prometheus: 50GB (metrics retention)
- Grafana: 10GB (dashboards and config)
- Loki: 10GB (log storage)
- AlertManager: 5GB (alert state)

## 🛡️ Security Details

**User IDs by component:**
- **Prometheus**: UID 1000, GID 2000
- **AlertManager**: UID 1000, GID 2000  
- **Grafana**: UID 472, GID 472
- **Loki**: UID 10001, GID 10001
- **Gateway**: UID 101, GID 101
- **Promtail**: UID 0* (requires host log access, but capabilities restricted)

**Security contexts applied:**
- `runAsNonRoot: true` (except Promtail)
- `allowPrivilegeEscalation: false`
- `capabilities.drop: ["ALL"]`
- `seccompProfile.type: RuntimeDefault`
- `readOnlyRootFilesystem: true` (where possible)

## ⚙️ Customization Points

**Change log retention (Loki):**
```yaml
loki:
  commonConfig:
    compactor_working_directory: /var/loki/boltdb-shipper-compactor
    shared_store: filesystem
  limits_config:
    retention_period: 30d  # Add retention
```

**Change metrics retention:**
```yaml
prometheus:
  prometheusSpec:
    retention: 90d  # Change from 30d
    retentionSize: 100GiB  # Change from 50GiB
```

**Change storage sizes:**
```yaml
# Prometheus storage
storageSpec:
  volumeClaimTemplate:
    spec:
      resources:
        requests:
          storage: 100Gi  # Change from 50Gi

# Loki storage  
singleBinary:
  persistence:
    size: 50Gi  # Change from 10Gi
```

**Set specific storage class:**
```yaml
storageClassName: fast-ssd  # Instead of default
```

## 🔍 Accessing Components

**Prometheus UI:**
```bash
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090
# Visit: http://localhost:9090
```

**AlertManager UI:**
```bash
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-alertmanager 9093:9093
# Visit: http://localhost:9093
```

**Loki API (for testing):**
```bash
kubectl port-forward -n monitoring svc/loki-gateway 3100:80
# API endpoint: http://localhost:3100
```

## ❓ Why No Ingress?

**Too opinionated** - Every setup has different requirements:
- Domain names vary
- TLS certificate approaches differ  
- Ingress controllers vary (Traefik, NGINX, etc.)
- Authentication methods differ

**To add ingress**, edit the ApplicationSet and add to the Grafana values:
```yaml
grafana:
  ingress:
    enabled: true
    hosts:
    - grafana.your-domain.com
```

## 🔧 Troubleshooting

**ApplicationSet not creating Applications?**
```bash
# Check your cluster has the right label
kubectl get secrets -n argocd --show-labels | grep monitoring=enabled
```

**Loki deployment failing?**
```bash
# Check Loki pods
kubectl get pods -n monitoring -l app.kubernetes.io/name=loki

# Check logs
kubectl logs -n monitoring -l app.kubernetes.io/name=loki --tail=50
```

**Promtail not shipping logs?**
```bash
# Check Promtail is running on all nodes
kubectl get daemonset -n monitoring promtail

# Check Promtail logs
kubectl logs -n monitoring -l app.kubernetes.io/name=promtail --tail=50
```

**Storage issues?**
```bash
# Check if your cluster has a default storage class
kubectl get storageclass

# Check PVC status
kubectl get pvc -n monitoring
```

**Security context issues?**
```bash
# Check pod security policies if using older Kubernetes
kubectl get psp

# Check if containers are running as expected users
kubectl get pods -n monitoring -o custom-columns="NAME:.metadata.name,SECURITY-CONTEXT:.spec.securityContext"
```

## 📋 Log Collection Details

**What Promtail collects:**
- All Kubernetes pod logs (stdout/stderr)
- Kubernetes events
- Container metadata (namespace, pod, container names)
- Node information

**Log parsing:**
- CRI log format parsing
- JSON structured logging support
- Automatic labeling with Kubernetes metadata

**Access patterns:**
- Logs available immediately in Grafana Explore
- Search by namespace, pod, container
- Time-based log browsing
- Log correlation with metrics

---

*Part of [opinionated-applicationsets](../README.md) - ApplicationSets that actually work, securely.*