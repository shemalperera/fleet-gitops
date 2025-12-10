# Fleet Management GitOps Repository

This repository contains Helm charts and ArgoCD configurations for deploying Fleet Management microservices.

## Directory Structure

```
fleet-gitops/
├── charts/
│   └── fleet-service/              # 🔹 Reusable Helm chart for all services
│       ├── Chart.yaml
│       ├── values.yaml             # Default values (optional)
│       └── templates/
│           ├── _helpers.tpl
│           ├── deployment.yaml
│           ├── service.yaml
│           ├── database.yaml
│           ├── hpa.yaml
│           └── ingress.yaml
│
├── envs/                           # 🔹 Environment-specific values
│   ├── dev/
│   │   ├── driver-values.yaml
│   │   ├── maintenance-values.yaml
│   │   └── vehicle-values.yaml
│   ├── staging/
│   │   ├── driver-values.yaml
│   │   ├── maintenance-values.yaml
│   │   └── vehicle-values.yaml
│   └── prod/
│       ├── driver-values.yaml
│       ├── maintenance-values.yaml
│       └── vehicle-values.yaml
│
└── argocd/                         # 🔹 ArgoCD application manifests
    ├── applications-dev.yaml
    ├── applications-staging.yaml
    └── applications-prod.yaml
```

## Services

1. **Driver Service** (Java/Spring Boot) - Port 6001
2. **Maintenance Service** (Python/Flask) - Port 5001
3. **Vehicle Service** (.NET/C#) - Port 7001

## Usage

### Manual Deployment with Helm

Deploy a specific service to an environment:

```bash
# Development
helm install driver-service ./fleet-gitops/charts/fleet-service \
  -f ./fleet-gitops/envs/dev/driver-values.yaml

helm install maintenance-service ./fleet-gitops/charts/fleet-service \
  -f ./fleet-gitops/envs/dev/maintenance-values.yaml

helm install vehicle-service ./fleet-gitops/charts/fleet-service \
  -f ./fleet-gitops/envs/dev/vehicle-values.yaml
```

```bash
# Staging
helm install driver-service ./fleet-gitops/charts/fleet-service \
  -f ./fleet-gitops/envs/staging/driver-values.yaml

# Production
helm install driver-service ./fleet-gitops/charts/fleet-service \
  -f ./fleet-gitops/envs/prod/driver-values.yaml
```

### GitOps Deployment with ArgoCD

#### 1. Apply ArgoCD Applications

```bash
# Development
kubectl apply -f fleet-gitops/argocd/applications-dev.yaml

# Staging
kubectl apply -f fleet-gitops/argocd/applications-staging.yaml

# Production
kubectl apply -f fleet-gitops/argocd/applications-prod.yaml
```

#### 2. Verify Applications

```bash
kubectl get applications -n argocd

argocd app list
```

#### 3. Sync Applications

```bash
# Development (auto-syncs)
argocd app sync driver-service-dev
argocd app sync maintenance-service-dev
argocd app sync vehicle-service-dev

# Production (manual sync required)
argocd app sync driver-service-prod
argocd app sync maintenance-service-prod
argocd app sync vehicle-service-prod
```

## Customization

### Adding a New Service

1. Create values file in `envs/{environment}/new-service-values.yaml`
2. Configure service-specific settings
3. Add ArgoCD application manifest
4. Apply and sync

Example values structure:
```yaml
serviceName: new-service
namespace: fleet-management-dev
replicaCount: 1

image:
  repository: new-service
  tag: dev
  pullPolicy: Always

service:
  type: ClusterIP
  port: 8080
  targetPort: 8080

env:
  KEY: "value"

database:
  enabled: true
  name: postgres-newservice
  storage: 1Gi
```

### Updating Image Tags

Edit the values file for the service and environment:

```yaml
image:
  tag: v1.2.3  # Update this
```

Commit and push. ArgoCD will auto-sync (dev/staging) or wait for manual sync (prod).

## Environment Configurations

### Development
- **Namespace**: `fleet-management-dev`
- **Replicas**: 1 per service
- **Auth**: Disabled
- **Sync**: Automated
- **Resources**: Minimal (100m CPU, 128Mi RAM)
- **Ingress**: Disabled

### Staging
- **Namespace**: `fleet-management-staging`
- **Replicas**: 2 per service
- **Auth**: Disabled
- **Sync**: Automated
- **Resources**: Medium (250m CPU, 256Mi RAM)
- **Ingress**: Enabled with staging TLS
- **HPA**: Enabled (2-5 replicas)

### Production
- **Namespace**: `fleet-management-prod`
- **Replicas**: 3 per service
- **Auth**: Enabled (Keycloak)
- **Sync**: Manual (requires approval)
- **Resources**: High (500m CPU, 512Mi RAM)
- **Ingress**: Enabled with prod TLS
- **HPA**: Enabled (3-10 replicas)
- **Storage**: 10Gi per database

## GitOps Workflow

```
deployment branch (dev)
    ↓
staging branch (staging)
    ↓
main branch (production)
```

### Development Flow
1. Commit to `deployment` branch
2. ArgoCD auto-syncs to dev namespace
3. Changes applied immediately

### Staging Flow
1. Merge `deployment` → `staging`
2. ArgoCD auto-syncs to staging namespace
3. QA testing performed

### Production Flow
1. Merge `staging` → `main`
2. ArgoCD detects changes (no auto-sync)
3. Manual review and approval
4. Execute: `argocd app sync {service}-prod`

## Monitoring

```bash
# Watch application status
argocd app watch driver-service-dev

# View logs
kubectl logs -n fleet-management-dev -l app.kubernetes.io/name=driver-service

# Check pod status
kubectl get pods -n fleet-management-dev
```

## Troubleshooting

### Application Out of Sync
```bash
argocd app sync driver-service-dev --force
```

### View Diff
```bash
argocd app diff driver-service-dev
```

### Rollback
```bash
argocd app history driver-service-dev
argocd app rollback driver-service-dev <revision>
```

## Best Practices

1. **Never commit directly to main** - Use PR workflow
2. **Test in dev/staging first** - Never skip environments
3. **Use semantic versioning** - Tag production images (v1.0.0)
4. **Backup before updates** - Especially database PVCs
5. **Monitor after deployment** - Check logs and metrics
6. **Keep secrets secure** - Use Kubernetes Secrets or external vaults

## Support

For issues or questions, refer to the project documentation or create an issue in the repository.
