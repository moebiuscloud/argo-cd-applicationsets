# Opinionated ArgoCD ApplicationSets

**Battle-tested ApplicationSet templates that actually work.**

## The Problem

Everyone needs monitoring, ingress, cert-manager, and other core services. Everyone struggles with ApplicationSet YAML. Nobody wants to spend hours debugging templates.

## The Solution

Copy-paste ApplicationSets for the most common Kubernetes patterns. Opinionated, but working out of the box.

## 📁 Available Templates

- **[monitoring/](monitoring/)** - kube-prometheus-stack with Grafana dashboards
- 🚧 **cert-manager/** - Coming soon
- 🚧 **ingress/** - Coming soon

## 🎯 Philosophy

**Very opinionated** - We don't know your requirements, so these templates make sensible defaults:

- ✅ **Copy-paste ready** - Working ApplicationSets you can use immediately
- ✅ **Selector-based** - Uses ArgoCD cluster labels (add via IaC)
- ✅ **Secure defaults** - Auto-generated secrets, no hardcoded passwords
- ❌ **No ingress** - Too opinionated, every setup is different
- ❌ **No custom domains** - You know your DNS setup, we don't

## 🚀 Quick Start

1. **Label your clusters** in ArgoCD:
   ```bash
   kubectl label secret my-cluster monitoring=enabled
   ```

2. **Copy the ApplicationSet** you need:
   ```bash
   curl -O https://raw.githubusercontent.com/mobius/opinionated-applicationsets/main/monitoring/basic-prometheus.yaml
   ```

3. **Apply to your cluster**:
   ```bash
   kubectl apply -f basic-prometheus.yaml
   ```

4. **Access your services** (see individual READMEs for details)

## 🏷️ Cluster Label Requirements

Each template requires specific labels on your ArgoCD clusters:

- **monitoring**: `monitoring=enabled`
- **cert-manager**: `cert-manager=enabled` (coming soon)
- **ingress**: `ingress=enabled` (coming soon)

Add these labels via Infrastructure as Code (Terraform, Pulumi, etc.) for best results.

## 🤝 Contributing

Found a bug? Have a better default? PRs welcome!

- Keep templates opinionated and minimal
- Document label requirements clearly  
- Test on real clusters before submitting

## 📖 Learn More

Each template directory contains:
- Working ApplicationSet YAML
- Detailed README with customization points
- Instructions for accessing deployed services

---

*Built with ❤️ for platform engineers who want things that just work.*