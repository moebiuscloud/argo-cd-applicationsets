# Monitoring - Kube-Prometheus-Stack + Loki

**Complete observability stack with Prometheus, Grafana, Loki, and AlertManager running securely as non-root users.**

## 🚀 Quick Start

```bash
# 1. Label your cluster
kubectl label secret my-cluster monitoring=enabled

# 2. Deploy
kubectl apply -f basic/applicationset.yaml

# 3. Access Grafana
kubectl get secret prometheus-grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 --decode
kubectl port-forward -n monitoring svc/prometheus-grafana 3000:80
```

Visit: http://localhost:3000 (admin / password from above)

## 📦 What's Included

### **Core Stack**
- **Prometheus** - Metrics collection (30d retention, 50GB storage)
- **Grafana** - Visualization with pre-loaded dashboards + Loki datasource
- **Loki** - Log aggregation (10GB storage) 
- **Promtail** - Log collection from all pods
- **AlertManager** - Alert routing (5GB storage)

### **Pre-configured Dashboards**
- Kubernetes Cluster Overview
- Kubernetes Pods 
- Node Exporter Full
- Loki Logs Dashboard

### **🔐 Security**
- All components run as non-root users
- Minimal capabilities, privilege escalation blocked
- Seccomp profiles and read-only filesystems where possible

## 🔧 Common Tasks

**Access other UIs:**
```bash
# Prometheus
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-prometheus 9090:9090

# AlertManager  
kubectl port-forward -n monitoring svc/prometheus-kube-prometheus-alertmanager 9093:9093
```

**Customize retention:**
```yaml
# Metrics (edit ApplicationSet values)
prometheus:
  prometheusSpec:
    retention: 90d
    retentionSize: 100GiB

# Logs (add to Loki config)
loki:
  limits_config:
    retention_period: 30d
```

**Change storage sizes:**
```yaml
# Prometheus
storageSpec:
  volumeClaimTemplate:
    spec:
      resources:
        requests:
          storage: 100Gi

# Loki  
singleBinary:
  persistence:
    size: 50Gi
```

## 🐛 Troubleshooting

**Application not deploying?**
```bash
kubectl get secrets -n argocd --show-labels | grep monitoring=enabled
```

**Pods not running?**
```bash
kubectl get pods -n monitoring
kubectl logs -n monitoring -l app.kubernetes.io/name=loki
```

**No logs in Grafana?**
```bash
kubectl logs -n monitoring -l app.kubernetes.io/name=promtail
```

**Storage issues?**
```bash
kubectl get storageclass
kubectl get pvc -n monitoring
```

---

**Why no ingress?** Too many variables (domains, TLS, controllers, auth). Add your own by editing the ApplicationSet Grafana values.

*Part of [opinionated-applicationsets](../README.md) - ApplicationSets that work securely out of the box.*