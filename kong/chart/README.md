# Kong Chart

This Helm chart deploys Kong API Gateway with Ingress Controller in DB-less mode.

## Components

### Kong API Gateway
- **Mode**: DB-less (declarative configuration, no database required)
- **Version**: Kong 3.5 (Helm chart v2.38.0)
- **Replicas**: 1 (configurable per environment)

### Kong Ingress Controller (KIC)
- Manages Kubernetes Ingress resources and converts them to Kong configuration
- Supports Gateway API with feature gates enabled
- Installs Custom Resource Definitions (CRDs) for advanced routing

### Services

#### Proxy Service
- **Type**: LoadBalancer
- **Ports**: 
  - HTTP: 80 → 8000
  - HTTPS: 443 → 8443
- **Purpose**: Main entry point for all API traffic

#### Admin API
- **Type**: ClusterIP (internal only)
- **Port**: 8001
- **Purpose**: REST API for programmatic Kong configuration (CLI, scripts, automation)

#### Manager (Admin GUI)
- **Type**: ClusterIP (internal only)
- **Port**: 8002
- **Purpose**: Web-based visual interface for managing Kong (uses Admin API)
- **Access**: `kubectl port-forward -n kong svc/kong-kong-manager 8002:8002`

### Disabled Features
- Developer Portal
- Enterprise features

## Configuration

Default values are defined in `values.yaml`. Per-environment overrides:
- `../values/dev.yaml` - Development environment
- `../values/staging.yaml` - Staging environment

## Deployment

This chart is deployed via ArgoCD ApplicationSets in `/argocd/applicationsets/`.

### Resource Configuration
- **CPU**: 250m (request) / 500m (limit)
- **Memory**: 256Mi (request) / 512Mi (limit)
- **Autoscaling**: Disabled by default (1-5 replicas when enabled)

