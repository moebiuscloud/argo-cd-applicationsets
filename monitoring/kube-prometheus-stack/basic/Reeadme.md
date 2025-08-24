# Monitoring - Kube-Prometheus-Stack

**Complete monitoring stack with Prometheus, Grafana, and AlertManager.**

## 🏷️ Required Labels

Your ArgoCD clusters need this label:
```bash
kubectl label secret my-cluster monitoring=enabled
```

## 📦 What You Get

- **Prometheus** - Metrics collection and storage (30d retention, 50GB)
- **Grafana** - Pre-configured with popular Kubernetes dashboards
- **AlertManager** - Alert routing and notifications (5GB storage)
- **Node Exporter** - Host metrics collection
- **Kube State Metrics** - Kubernetes object metrics

## 🚀 Deploy

```bash
# 1. Label your cluster
kubectl label secret my-cluster monitoring=enabled

# 2. Apply the ApplicationSet
kubectl apply -f basic-prometheus.yaml

# 3. Wait for deployment (2-3 minutes)
kubectl get applications -n argocd | grep monitoring
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
- Kubernetes Cluster Overview (ID: 7249)
- Kubernetes Pods (ID: 6417)  
- Node Exporter Full (ID: 1860)

**Storage:**
- Prometheus: 50GB (default storage class)
- Grafana: 10GB persistence
- AlertManager: 5GB

## ⚙️ Customization Points

**Change retention:**
```yaml
prometheus:
  prometheusSpec:
    retention: 90d  # Change from 30d
```

**Change storage:**
```yaml
storageSpec:
  volumeClaimTemplate:
    spec:
      resources:
        requests:
          storage: 100Gi  # Change from 50Gi
```

**Set specific storage class:**
```yaml
storageClassName: fast-ssd  # Instead of default
```

## ❓ Why No Ingress?

**Too opinionated** - Every setup has different requirements:
- Domain names vary
- TLS certificate approaches differ  
- Ingress controllers vary (Traefik, NGINX, etc.)
- Authentication methods differ

**To add ingress**, edit the ApplicationSet and add:
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

**Grafana password not working?**
```bash
# The password is auto-generated on first install
kubectl get secret prometheus-grafana -n monitoring -o jsonpath='{.data.admin-password}' | base64 --decode
```

**Storage issues?**
```bash
# Check if your cluster has a default storage class
kubectl get storageclass
```

---

*Part of [opinionated-applicationsets](../README.md) - ApplicationSets that actually work.*