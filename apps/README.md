# Applications

This directory contains all application deployments managed by the GitOps workflow. Applications are organized using Kustomize with base configurations and environment-specific overlays.

## 📁 Structure

```
apps/
├── base/                    # Base application configurations
│   └── linkding/           # Linkding bookmark manager base
└── staging/                # Staging environment overlays
    └── linkding/           # Staging-specific configurations
```

## 🏗️ Architecture Pattern

### Base Configurations
- **Purpose**: Environment-agnostic application definitions
- **Contents**: Core Kubernetes resources (Deployment, Service, PVC, etc.)
- **Reusability**: Shared across all environments

### Environment Overlays
- **Purpose**: Environment-specific customizations
- **Contents**: Ingress rules, secrets, environment variables
- **Customization**: Patches and additions to base configurations

## 📱 Applications

### Linkding
**Self-hosted bookmark manager for organizing and sharing links**

#### Base Configuration (`base/linkding/`)
- **Deployment**: Container specification and resource requirements
- **Service**: Internal cluster networking
- **Storage**: Persistent volume claim for data persistence
- **Namespace**: Isolated namespace for the application

#### Staging Configuration (`staging/linkding/`)
- **Ingress**: External access via `lds.ludfrancisco.com`
- **Secrets**: Environment-specific configuration and credentials
- **TLS**: Certificate management for secure connections

## 🔧 Configuration Details

### Linkding Specifications
```yaml
Image: sissbruecker/linkding:1.31.0
Port: 9090
Storage: Persistent volume for bookmark data
Security: Non-root user execution (www-data)
```

### Environment Variables
- Managed via Kubernetes secrets
- SOPS encryption for sensitive data
- Environment-specific configuration injection

## 🚀 Deployment Process

1. **Base Resources**: Applied first to establish core application structure
2. **Environment Overlay**: Kustomize patches applied for environment-specific settings
3. **Flux Reconciliation**: Continuous monitoring and automatic updates
4. **Health Checks**: Kubernetes probes ensure application availability

## 🔐 Security Considerations

### Container Security
- Non-privileged container execution
- Security context with specific user/group IDs
- No privilege escalation allowed

### Data Protection
- Persistent storage with proper permissions
- Encrypted secrets using SOPS
- Network policies for traffic isolation

## 📊 Monitoring Integration

### Metrics Collection
- Prometheus scraping configuration
- Application-specific metrics exposure
- Health and performance monitoring

### Logging
- Centralized log collection
- Structured logging format
- Log retention policies

## 🛠️ Management Commands

### Check Application Status
```bash
# View application pods
kubectl get pods -n linkding

# Check application logs
kubectl logs -n linkding deployment/linkding

# Describe application resources
kubectl describe deployment linkding -n linkding
```

### Force Reconciliation
```bash
# Trigger Flux reconciliation
flux reconcile kustomization apps

# Check reconciliation status
flux get kustomizations
```

## 🔄 Adding New Applications

### 1. Create Base Configuration
```bash
mkdir -p apps/base/new-app
# Add deployment.yaml, service.yaml, namespace.yaml, etc.
```

### 2. Create Environment Overlay
```bash
mkdir -p apps/staging/new-app
# Add kustomization.yaml with patches and additions
```

### 3. Update Kustomization
```yaml
# apps/staging/kustomization.yaml
resources:
  - linkding
  - new-app  # Add new application
```

## 📝 Best Practices

### Resource Organization
- One directory per application
- Consistent naming conventions
- Proper resource labeling

### Configuration Management
- Use ConfigMaps for non-sensitive data
- Use Secrets for sensitive information
- Environment-specific values in overlays

### Security
- Always specify security contexts
- Use non-root containers when possible
- Implement proper RBAC policies

## 🔗 Related Documentation

- [Main Repository README](../README.md)
- [Infrastructure Documentation](../infrastructure/README.md)
- [Monitoring Documentation](../monitoring/README.md)
- [Cluster Configuration](../clusters/README.md)

---

**Note**: All applications follow GitOps principles with declarative configuration and automated deployment through Flux CD.
