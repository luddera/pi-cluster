# Clusters

This directory contains Flux CD cluster configurations that define how GitOps operates across different environments. Each cluster environment has its own configuration directory with Flux system components and kustomization definitions.

## 📁 Structure

```
clusters/
└── staging/                    # Staging environment configuration
    ├── .sops.yaml             # SOPS encryption configuration
    ├── apps.yaml              # Application kustomizations
    ├── infrastructure.yaml     # Infrastructure kustomizations
    ├── monitoring.yaml        # Monitoring kustomizations
    └── flux-system/           # Flux system components
        ├── gotk-components.yaml
        ├── gotk-sync.yaml
        └── kustomization.yaml
```

## 🏗️ Flux Architecture

### GitOps Workflow
1. **Source Controller**: Monitors Git repository for changes
2. **Kustomize Controller**: Applies Kubernetes manifests
3. **Helm Controller**: Manages Helm releases
4. **Notification Controller**: Sends alerts and notifications

### Reconciliation Process
- **Interval**: 1-10 minutes for different components
- **Source**: Git repository `luddera/pi-cluster`
- **Branch**: `master`
- **Path**: `./clusters/staging`

## 🔧 Configuration Files

### Flux System (`flux-system/`)

#### gotk-sync.yaml
```yaml
# GitRepository source configuration
apiVersion: source.toolkit.fluxcd.io/v1
kind: GitRepository
metadata:
  name: flux-system
spec:
  interval: 1m0s
  ref:
    branch: master
  url: ssh://git@github.com/luddera/pi-cluster
```

#### gotk-components.yaml
- Core Flux controllers and CRDs
- System-level configuration
- RBAC permissions and service accounts

### Kustomization Definitions

#### apps.yaml
- **Purpose**: Manages application deployments
- **Path**: `./apps/staging`
- **Features**:
  - SOPS decryption enabled
  - 10-minute reconciliation interval
  - Automatic pruning of removed resources
  - 5-minute timeout with 1-minute retry

#### infrastructure.yaml
- **Purpose**: Manages infrastructure components
- **Path**: `./infrastructure/controllers/staging`
- **Features**:
  - Core infrastructure services
  - Dependency management
  - Resource lifecycle management

#### monitoring.yaml
- **Purpose**: Manages monitoring stack
- **Path**: `./monitoring/controllers/staging`
- **Features**:
  - Prometheus and Grafana deployment
  - Metrics collection configuration
  - Dashboard and alerting setup

## 🔐 Security Configuration

### SOPS Integration (.sops.yaml)
```yaml
# Age encryption configuration
creation_rules:
  - path_regex: \.yaml$
    age: >-
      age1...  # Age public key for encryption
```

### Secret Management
- **Encryption**: Age-based encryption for sensitive data
- **Decryption**: Automatic decryption during reconciliation
- **Key Management**: Age keys stored as Kubernetes secrets

### Access Control
- **SSH Keys**: Git repository access via SSH
- **RBAC**: Flux controllers have minimal required permissions
- **Namespace Isolation**: Components isolated in flux-system namespace

## 🚀 Bootstrap Process

### Initial Setup
```bash
# Bootstrap Flux in staging cluster
flux bootstrap github \
  --owner=luddera \
  --repository=pi-cluster \
  --branch=master \
  --path=./clusters/staging \
  --personal
```

### Post-Bootstrap Configuration
```bash
# Create SOPS age key secret
kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=./age.agekey

# Verify installation
flux check
```

## 📊 Monitoring and Observability

### Flux Status Monitoring
```bash
# Check all Flux resources
flux get all

# Monitor reconciliation
flux get kustomizations --watch

# View controller logs
flux logs --follow
```

### Health Checks
- **Source Controller**: Git repository connectivity
- **Kustomize Controller**: Manifest application status
- **Helm Controller**: Helm release management
- **Notification Controller**: Alert delivery

## 🛠️ Management Operations

### Common Commands
```bash
# Force reconciliation
flux reconcile source git flux-system
flux reconcile kustomization flux-system

# Suspend/resume reconciliation
flux suspend kustomization apps
flux resume kustomization apps

# Check resource status
kubectl get gitrepositories -n flux-system
kubectl get kustomizations -n flux-system
```

### Troubleshooting
```bash
# Check Flux system health
flux check --pre

# View reconciliation errors
flux get sources git
flux get kustomizations

# Examine controller logs
kubectl logs -n flux-system deployment/source-controller
kubectl logs -n flux-system deployment/kustomize-controller
kubectl logs -n flux-system deployment/helm-controller
```

## 🔄 Environment Management

### Adding New Environments
1. **Create Environment Directory**
   ```bash
   mkdir -p clusters/production
   ```

2. **Copy Base Configuration**
   ```bash
   cp -r clusters/staging/* clusters/production/
   ```

3. **Update Environment-Specific Settings**
   - Modify kustomization paths
   - Update SOPS configuration
   - Adjust reconciliation intervals

4. **Bootstrap New Environment**
   ```bash
   flux bootstrap github \
     --path=./clusters/production
   ```

### Environment Isolation
- **Namespaces**: Each environment uses separate namespaces
- **Secrets**: Environment-specific encryption keys
- **Resources**: Isolated resource management
- **Networking**: Environment-specific ingress rules

## 📝 Best Practices

### Configuration Management
- Keep environment configurations minimal and focused
- Use consistent naming conventions across environments
- Document environment-specific customizations

### Security
- Rotate age keys regularly
- Use separate encryption keys per environment
- Implement proper RBAC for Flux controllers

### Monitoring
- Monitor Flux controller health
- Set up alerts for reconciliation failures
- Track resource drift and compliance

## 🔗 Related Documentation

- [Main Repository README](../README.md)
- [Applications Documentation](../apps/README.md)
- [Infrastructure Documentation](../infrastructure/README.md)
- [Monitoring Documentation](../monitoring/README.md)
- [Flux Documentation](https://fluxcd.io/docs/)

---

**Note**: This configuration is optimized for staging environments. Production deployments should include additional security measures, monitoring, and resource constraints.
