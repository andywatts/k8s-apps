# k8s-infra

Platform infrastructure services and ArgoCD configuration for Kubernetes clusters.

## Structure

```
argocd/
  projects/           # AppProjects (namespace isolation)
    kong.yaml
    sample-app.yaml
  applicationsets/    # ApplicationSets (per environment)
    dev.yaml
    staging.yaml
kong/
  chart/              # Kong Helm chart wrapper
    values.yaml       # Default Kong configuration
  values/             # Environment-specific configs
    dev.yaml
    staging.yaml
```

## What's Deployed

### Kong API Gateway
- **Ingress Controller** in DB-less mode
- **Load Balancer** for traffic routing
- **Admin API** and **Manager GUI** for management
- Deployed via ArgoCD to dev/staging environments

### Sample App
- Example application with Kong ingress configured
- Demonstrates rate limiting and CORS plugins
- Located in separate repo: `/infra/sample-app`

## Quick Start

### Deploy Kong

```bash
# One command deployment
./deploy-kong.sh dev
```

The script will:
- Connect to your GKE cluster
- Apply the ApplicationSet
- Wait for Kong to be ready
- Display the LoadBalancer IP and access instructions

### Manual Deployment

```bash
# Connect to cluster
gcloud container clusters get-credentials dev-cluster \
  --zone=us-west2-a --project=development-690488

# Deploy ArgoCD resources
kubectl apply -f argocd/projects/kong.yaml
kubectl apply -f argocd/applicationsets/dev.yaml

# Check status
kubectl get applications -n argocd
kubectl get pods -n kong
```

### Access Kong

```bash
# Get proxy LoadBalancer IP
kubectl get svc -n kong kong-kong-proxy

# Test Kong
curl http://<KONG_IP>

# Access Admin API (port-forward)
kubectl port-forward -n kong svc/kong-kong-admin 8001:8001

# Access Manager GUI (port-forward)
kubectl port-forward -n kong svc/kong-kong-manager 8002:8002
# Open http://localhost:8002
```

## Add New Infrastructure Service

1. **Create chart structure:**
```bash
mkdir -p my-service/chart/templates my-service/values
```

2. **Add environment configs:**
```bash
# my-service/values/dev.yaml
# my-service/values/staging.yaml
```

3. **Create AppProject:**
```bash
# argocd/projects/my-service.yaml
```

4. **Add to ApplicationSet:**
```yaml
# In argocd/applicationsets/dev.yaml
- app: my-service
  repoPath: my-service/chart
  valuesFile: my-service/values/dev.yaml
```

5. **Deploy:**
```bash
git add . && git commit -m "Add my-service" && git push
kubectl apply -f argocd/applicationsets/dev.yaml
```

## Documentation

- **[KONG.md](./KONG.md)** - Complete Kong API Gateway documentation
- **[deploy-kong.sh](./deploy-kong.sh)** - Automated deployment script

## ArgoCD Management

```bash
# View applications
kubectl get applications -n argocd

# Force sync
kubectl patch app dev-kong -n argocd \
  --type merge -p '{"metadata":{"annotations":{"argocd.argoproj.io/refresh":"hard"}}}'

# View application details
kubectl describe application dev-kong -n argocd
```

---

**GitOps:** Push to repo → ArgoCD syncs automatically (auto-sync enabled, prune enabled)
