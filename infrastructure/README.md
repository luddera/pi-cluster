# Infrastructure

This directory contains core infrastructure components and tooling that support the Kubernetes cluster operations. Infrastructure components are managed through GitOps and provide essential services for cluster maintenance and automation.

## 📁 Structure

```
infrastructure/
└── controllers/
    ├── base/                   # Base infrastructure configurations
    │   └── renovate/          # Automated dependency management
    │       ├── configmap.yaml
    │       ├── cronjob.yaml
    │       ├── kustomization.yaml
    │       ├── namespace.yaml
    │       └── renovate-container-env.yaml
    └── staging/               # Staging environment overlays
        ├── kustomization.yaml
        └── renovate/
            └── kustomization.yaml
```

## 🏗️ Architecture Pattern

### Base Infrastructure
- **Purpose**: Environment-agnostic infrastructure services
- **Contents**: Core infrastructure components and configurations
- **Reusability**: Shared foundation across all environments

### Environment Overlays
- **Purpose**: Environment-specific infrastructure customizations
- **Contents**: Environment variables, resource limits, scheduling
- **Customization**: Patches and additions to base configurations

## 🔧 Infrastructure Components

### Renovate - Automated Dependency Management

**Automated dependency updates for Kubernetes manifests and container images**

#### Purpose
- **Dependency Scanning**: Monitors YAML files for outdated dependencies
- **Pull Request Creation**: Automatically creates PRs with updates
- **Security Updates**: Keeps container images and dependencies current
- **Compliance**: Ensures cluster components stay up-to-date

#### Configuration Details

##### CronJob Specification
```yaml
Schedule: "@hourly"              # Runs every hour
Concurrency Policy: Forbid       # Prevents overlapping executions
Image: renovate/renovate:latest  # Latest Renovate container
Target Repository: luddera/pi-cluster
```

##### Security Context
- **Namespace**: Isolated `renovate` namespace
- **Secrets**: GitHub token and configuration stored securely
- **Permissions**: Minimal required access for repository operations

##### Environment Configuration
- **ConfigMap**: Non-sensitive Renovate configuration
- **Secrets**: GitHub authentication and sensitive settings
- **Repository**: Targets `luddera/pi-cluster` for updates

#### Renovate Features
- **Kubernetes Support**: Detects and updates Kubernetes YAML files
- **Container Images**: Monitors and updates container image tags
- **Helm Charts**: Updates Helm chart versions
- **Schedule Control**: Configurable update schedules and policies

## 🔐 Security Implementation

### Access Control
- **Namespace Isolation**: Renovate runs in dedicated namespace
- **RBAC**: Minimal permissions for cluster operations
- **Secret Management**: Encrypted storage of authentication tokens

### Container Security
- **Non-Root Execution**: Containers run with non-privileged users
- **Resource Limits**: CPU and memory constraints applied
- **Network Policies**: Restricted network access where applicable

### Data Protection
- **Encrypted Secrets**: SOPS encryption for sensitive configuration
- **Audit Logging**: All infrastructure changes logged and tracked
- **Version Control**: All changes tracked through Git history

## 🚀 Deployment Process

### Automated Workflow
1. **Base Deployment**: Core infrastructure components deployed
2. **Environment Overlay**: Environment-specific configurations applied
3. **Flux Reconciliation**: Continuous monitoring and updates
4. **Health Monitoring**: Automated health checks and alerting

### Dependency Management Flow
1. **Scheduled Execution**: Renovate runs hourly via CronJob
2. **Repository Scanning**: Scans for outdated dependencies
3. **Update Detection**: Identifies available updates
4. **PR Creation**: Creates pull requests with proposed changes
5. **Review Process**: Manual review and approval required
6. **Automated Deployment**: Approved changes deployed via GitOps

## 📊 Monitoring and Observability

### Infrastructure Metrics
- **Job Execution**: CronJob success/failure rates
- **Resource Usage**: CPU and memory consumption
- **Update Statistics**: Number of dependencies updated
- **Error Tracking**: Failed update attempts and reasons

### Logging
- **Execution Logs**: Detailed logs of Renovate runs
- **Update History**: Track of all dependency updates
- **Error Logs**: Troubleshooting information for failures

### Alerting
- **Failed Jobs**: Notifications for failed Renovate executions
- **Security Updates**: Alerts for critical security updates
- **Resource Limits**: Warnings for resource constraint violations

## 🛠️ Management Operations

### Check Infrastructure Status
```bash
# View Renovate namespace resources
kubectl get all -n renovate

# Check CronJob status
kubectl get cronjobs -n renovate

# View recent job executions
kubectl get jobs -n renovate

# Check job logs
kubectl logs -n renovate job/renovate-<timestamp>
```

### Manual Operations
```bash
# Trigger manual Renovate run
kubectl create job --from=cronjob/renovate manual-renovate -n renovate

# Suspend/resume CronJob
kubectl patch cronjob renovate -n renovate -p '{"spec":{"suspend":true}}'
kubectl patch cronjob renovate -n renovate -p '{"spec":{"suspend":false}}'

# Update Renovate configuration
kubectl edit configmap renovate-configmap -n renovate
```

### Troubleshooting
```bash
# Check CronJob configuration
kubectl describe cronjob renovate -n renovate

# View failed job details
kubectl describe job <job-name> -n renovate

# Check secret configuration
kubectl get secrets -n renovate

# Verify RBAC permissions
kubectl auth can-i --list --as=system:serviceaccount:renovate:default
```

## 🔄 Adding New Infrastructure Components

### 1. Create Base Configuration
```bash
mkdir -p infrastructure/controllers/base/new-component
# Add necessary Kubernetes manifests
```

### 2. Define Component Resources
```yaml
# Example: deployment.yaml, service.yaml, configmap.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: new-component
  namespace: infrastructure
```

### 3. Create Environment Overlay
```bash
mkdir -p infrastructure/controllers/staging/new-component
# Add kustomization.yaml with environment-specific patches
```

### 4. Update Kustomization
```yaml
# infrastructure/controllers/staging/kustomization.yaml
resources:
  - renovate
  - new-component  # Add new infrastructure component
```

## 📝 Configuration Management

### Renovate Configuration
- **Repository Settings**: Target repository and branch configuration
- **Update Policies**: Scheduling and approval requirements
- **File Patterns**: Which files to monitor for updates
- **Ignore Patterns**: Dependencies to exclude from updates

### Environment Variables
- **GitHub Token**: Authentication for repository access
- **Renovate Config**: JSON configuration for update behavior
- **Logging Level**: Control verbosity of execution logs

### Resource Management
- **CPU Limits**: Prevent resource exhaustion
- **Memory Limits**: Control memory usage
- **Job History**: Retention policy for completed jobs

## 📋 Best Practices

### Infrastructure as Code
- All infrastructure components defined declaratively
- Version control for all configuration changes
- Automated testing and validation where possible

### Security
- Regular rotation of authentication tokens
- Minimal required permissions for all components
- Encrypted storage of sensitive configuration

### Monitoring
- Comprehensive logging for all infrastructure operations
- Alerting for critical infrastructure failures
- Regular review of infrastructure component health

### Maintenance
- Regular updates of infrastructure component images
- Periodic review of resource usage and limits
- Documentation updates for configuration changes

## 🔗 Related Documentation

- [Main Repository README](../README.md)
- [Applications Documentation](../apps/README.md)
- [Monitoring Documentation](../monitoring/README.md)
- [Cluster Configuration](../clusters/README.md)
- [Renovate Documentation](https://docs.renovatebot.com/)

---

**Note**: Infrastructure components are critical for cluster operations. Changes should be thoroughly tested and reviewed before deployment to production environments.
