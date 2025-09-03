# Container Registry - Harbor

**Production-ready container registry with vulnerability scanning, image signing, and RBAC running securely as non-root users.**

## 🚀 Quick Start

```bash
# 1. Label your cluster
kubectl label secret my-cluster registry=enabled

# 2. Deploy
kubectl apply -f basic/applicationset.yaml

# 3. Access Harbor
kubectl port-forward -n harbor svc/harbor 8080:80
```

Visit: http://localhost:8080
- Username: `admin`
- Password: `password12345` ⚠️ **CHANGE THIS IMMEDIATELY**

## 📦 What's Included

### **Core Components**
- **Harbor Portal** - Web UI for registry management
- **Harbor Registry** - OCI-compliant container registry (50GB storage)
- **Trivy Scanner** - Vulnerability scanning for container images (20GB storage)
- **PostgreSQL** - Internal database for Harbor metadata (10GB storage)
- **Redis** - Caching and job queue (5GB storage)

### **Features**
- **Vulnerability Scanning** - Automatic security scanning with Trivy
- **Image Signing** - Content trust and image signing capabilities
- **RBAC** - Role-based access control with projects and users
- **Replication** - Multi-registry replication support
- **Webhooks** - Integration with CI/CD pipelines
- **Garbage Collection** - Automated cleanup of unused images

### **🔐 Security**
- All components run as non-root users
- Minimal capabilities, privilege escalation blocked
- Seccomp profiles and read-only filesystems where possible
- Internal TLS disabled for simplicity (add your own for production)

## 🔧 Common Tasks

**Change admin password (REQUIRED):**
```bash
# After first login, go to: Administration → Users → admin → Edit
# Or update the ApplicationSet values before deployment:
```
```yaml
helm:
  values: |
    harborAdminPassword: "your-secure-password-here"
```

**Create a project:**
1. Login to Harbor UI
2. Go to Projects → New Project
3. Set project name and access level (public/private)

**Push/Pull images:**
```bash
# Login to Harbor
docker login localhost:8080

# Tag and push
docker tag my-image:latest localhost:8080/my-project/my-image:latest
docker push localhost:8080/my-project/my-image:latest

# Pull
docker pull localhost:8080/my-project/my-image:latest
```

**Access other endpoints:**
```bash
# Registry API
curl http://localhost:8080/v2/_catalog

# Health check
curl http://localhost:8080/api/v2.0/health
```

**Enable metrics collection:**
```yaml
# Add to ApplicationSet values
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
    additionalLabels:
      release: prometheus
```

## ⚙️ Configuration

**Change storage sizes:**
```yaml
persistence:
  persistentVolumeClaim:
    registry:
      size: 100Gi  # Registry storage
    trivy:
      size: 50Gi   # Vulnerability DB
    database:
      size: 20Gi   # PostgreSQL
```

**Customize scanning:**
```yaml
trivy:
  vulnType: "os,library"
  severity: "HIGH,CRITICAL"  # Only scan for high/critical
  timeout: 10m0s
```

**Add external URL (for production):**
```yaml
# Update externalURL in ApplicationSet values
externalURL: https://harbor.your-domain.com
```

## 🐛 Troubleshooting

**Application not deploying?**
```bash
kubectl get secrets -n argocd --show-labels | grep registry=enabled
```

**Pods not running?**
```bash
kubectl get pods -n harbor
kubectl logs -n harbor -l app=harbor-core
```

**Can't push/pull images?**
```bash
# Check registry pod
kubectl logs -n harbor -l component=registry

# Check storage
kubectl get pvc -n harbor
```

**Scanning not working?**
```bash
kubectl logs -n harbor -l component=trivy
```

**Database connection issues?**
```bash
kubectl logs -n harbor -l component=database
kubectl get svc -n harbor
```

## 🔒 Production Considerations

**Security:**
- Change default admin password immediately
- Enable internal TLS for production
- Configure external authentication (LDAP/OIDC)
- Set up proper RBAC with projects and users

**Networking:**
- Configure ingress with TLS termination
- Use proper DNS names instead of localhost
- Consider network policies for pod-to-pod communication

**Storage:**
- Use high-performance storage classes
- Configure backup strategies for PostgreSQL
- Monitor storage usage and set up alerting

---

**Why no ingress?** Too many variables (domains, TLS, controllers, auth). Add your own by editing the ApplicationSet expose configuration.

**Why ClusterIP?** Perfect for port-forward during setup. Switch to LoadBalancer or add Ingress for production access.

*Part of [opinionated-applicationsets](../README.md) - ApplicationSets that work securely out of the box.*
