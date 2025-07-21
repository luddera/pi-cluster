# Pi Cluster - GitOps Repository

A GitOps-managed Kubernetes cluster running on Raspberry Pi infrastructure, utilizing Flux CD for continuous deployment and automated dependency management with Renovate.

## 🏗️ Architecture Overview

This repository follows GitOps principles using Flux CD to manage a Kubernetes cluster. The infrastructure is organized into distinct layers:

- **Applications**: Self-contained application deployments
- **Infrastructure**: Core cluster infrastructure and tooling
- **Monitoring**: Observability stack with Prometheus and Grafana
- **Clusters**: Environment-specific Flux configurations

## 📁 Repository Structure

```
pi-cluster/
├── apps/                           # Application deployments
│   ├── base/                       # Base application configurations
│   │   └── linkding/              # Linkding bookmark manager
│   └── staging/                    # Staging environment overlays
│       └── linkding/              # Environment-specific configs
├── clusters/                       # Flux cluster configurations
│   └── staging/                    # Staging cluster setup
│       ├── apps.yaml              # Application kustomizations
│       ├── infrastructure.yaml     # Infrastructure kustomizations
│       ├── monitoring.yaml        # Monitoring kustomizations
│       └── flux-system/           # Flux system components
├── infrastructure/                 # Infrastructure components
│   └── controllers/
│       ├── base/
│       │   └── renovate/          # Automated dependency updates
│       └── staging/
├── monitoring/                     # Monitoring and observability
│   ├── controllers/
│   │   └── base/
│   │       └── kube-prometheus-stack/  # Prometheus & Grafana
│   └── configs/
└── renovate.json                  # Renovate configuration
```

## 🚀 Technologies Used

- **Kubernetes**: Container orchestration platform
- **Flux CD**: GitOps continuous delivery
- **Kustomize**: Kubernetes configuration management
- **Helm**: Package manager for Kubernetes
- **SOPS**: Secrets management with age encryption
- **Renovate**: Automated dependency updates
- **Prometheus**: Metrics collection and alerting
- **Grafana**: Metrics visualization and dashboards
- **Traefik**: Ingress controller and load balancer

## 🔧 Components

### Applications

#### Linkding
- **Description**: Self-hosted bookmark manager
- **Image**: `sissbruecker/linkding:1.31.0`
- **URL**: https://lds.ludfrancisco.com
- **Features**: 
  - Persistent storage for bookmarks
  - Secure configuration via encrypted secrets
  - Ingress with Traefik routing

### Infrastructure

#### Renovate
- **Purpose**: Automated dependency updates
- **Schedule**: Runs hourly via CronJob
- **Features**:
  - Monitors Kubernetes YAML files for updates
  - Creates pull requests for dependency updates
  - Prevents concurrent executions

### Monitoring

#### Kube-Prometheus-Stack
- **Components**: Prometheus, Grafana, AlertManager
- **Version**: `66.2.2`
- **URL**: https://grs.ludfrancisco.com
- **Features**:
  - Comprehensive cluster monitoring
  - Custom dashboards and alerts
  - Ingress with TLS termination
  - Drift detection enabled

## 🚨 CRITICAL SECURITY NOTICE

**⚠️ SECURITY INCIDENT RESOLVED ⚠️**

**Previous Security Issue**: This repository previously contained exposed cryptographic keys in Git history:
- `age.agekey` (SOPS private key)
- `tls.key` and `tls.crt` (TLS certificates)

**Actions Taken**:
- ✅ Removed sensitive files from repository
- ✅ Updated .gitignore to prevent future exposure
- ✅ Committed security fix

**REQUIRED ACTIONS FOR USERS**:
1. **🔑 Regenerate ALL compromised keys immediately**
2. **🔄 Update all systems using these keys**
3. **🔍 Audit access logs for unauthorized usage**
4. **📋 Review and rotate any dependent credentials**

## 🔐 Security

### Secrets Management
- **SOPS**: Used for encrypting sensitive data with age encryption
- **Age**: Encryption backend for SOPS (keys must be regenerated)
- **Secret References**: Environment variables stored as Kubernetes secrets
- **Key Rotation**: Regular rotation of encryption keys recommended

### Security Best Practices
- **Never commit private keys**: Use .gitignore patterns for sensitive files
- **Use SOPS encryption**: All secrets should be encrypted before committing
- **Regular key rotation**: Implement periodic key rotation policies
- **Access auditing**: Monitor and log access to sensitive resources

### Security Contexts
- Applications run with non-root users
- Privilege escalation disabled
- Proper file system permissions
- Network policies for traffic isolation

### Key Regeneration Guide

#### 1. Generate New Age Key
```bash
# Generate new age key pair
age-keygen -o age.agekey
# Keep this file secure and never commit it!

# Extract public key for SOPS configuration
grep "# public key:" age.agekey
```

#### 2. Update SOPS Configuration
```bash
# Update .sops.yaml with new public key
# Re-encrypt all existing secrets with new key
find . -name "*.yaml" -path "*/staging/*" -exec sops updatekeys {} \;
```

#### 3. Generate New TLS Certificates
```bash
# Generate new TLS certificate and key
openssl req -x509 -newkey rsa:4096 -keyout tls.key -out tls.crt -days 365 -nodes
# Update ingress configurations with new certificates
```

#### 4. Update Kubernetes Secrets
```bash
# Update SOPS age key secret in cluster
kubectl delete secret sops-age -n flux-system
kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=./age.agekey

# Update TLS secrets
kubectl create secret tls grafana-tls-secret \
  --cert=tls.crt \
  --key=tls.key \
  -n monitoring
```

## 🌐 Networking

### Ingress Configuration
- **Controller**: Traefik
- **TLS**: Automated certificate management
- **Domains**: 
  - Linkding: `lds.ludfrancisco.com`
  - Grafana: `grs.ludfrancisco.com`

## 📋 Prerequisites

- Kubernetes cluster (Raspberry Pi or compatible)
- Flux CLI installed
- kubectl configured for cluster access
- SOPS and age for secrets management
- Git repository access (SSH keys configured)

## 🚀 Getting Started

### 1. Bootstrap Flux

```bash
# Install Flux CLI
curl -s https://fluxcd.io/install.sh | sudo bash

# Bootstrap Flux in your cluster
flux bootstrap github \
  --owner=luddera \
  --repository=pi-cluster \
  --branch=master \
  --path=./clusters/staging \
  --personal
```

### 2. Configure Secrets

```bash
# Create SOPS age key secret
kubectl create secret generic sops-age \
  --namespace=flux-system \
  --from-file=age.agekey=./age.agekey

# Verify Flux installation
flux check
```

### 3. Monitor Deployments

```bash
# Watch Flux reconciliation
flux get kustomizations --watch

# Check application status
kubectl get pods -A

# View Flux logs
flux logs --follow --tail=10
```

## 🔄 GitOps Workflow

1. **Code Changes**: Push changes to the repository
2. **Flux Detection**: Flux detects changes within 1-10 minutes
3. **Reconciliation**: Flux applies changes to the cluster
4. **Monitoring**: Prometheus monitors application health
5. **Alerts**: Grafana dashboards provide visibility

## 🤖 Automated Updates

### Renovate Configuration
- **Schedule**: Hourly execution
- **Scope**: Kubernetes YAML files
- **Process**: Creates PRs for dependency updates
- **Repository**: Monitors `luddera/pi-cluster`

### Update Process
1. Renovate scans for outdated dependencies
2. Creates pull requests with updates
3. Manual review and approval
4. Flux applies approved changes

## 📊 Monitoring & Observability

### Access URLs
- **Linkding**: https://lds.ludfrancisco.com - Bookmark management
- **Grafana**: https://grs.ludfrancisco.com - Monitoring dashboards

### Grafana Features
- **Authentication**: Admin credentials configured
- **Dashboards**: Pre-configured cluster monitoring
- **Alerts**: Custom alerting rules

### Prometheus Metrics
- Cluster resource utilization
- Application performance metrics
- Infrastructure health monitoring
- Custom alerting rules

## 🛠️ Maintenance

### Common Operations

```bash
# Force Flux reconciliation
flux reconcile kustomization flux-system

# Suspend/resume reconciliation
flux suspend kustomization apps
flux resume kustomization apps

# Check resource status
kubectl get helmreleases -A
kubectl get kustomizations -A
```

### Troubleshooting

```bash
# Check Flux system status
flux check --pre

# View reconciliation errors
flux get all

# Examine specific component logs
kubectl logs -n flux-system deployment/source-controller
kubectl logs -n flux-system deployment/kustomize-controller
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make changes following GitOps principles
4. Test changes in staging environment
5. Submit a pull request

## 📝 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 🔗 Useful Links

- [Flux Documentation](https://fluxcd.io/docs/)
- [Kustomize Documentation](https://kustomize.io/)
- [SOPS Documentation](https://github.com/mozilla/sops)
- [Renovate Documentation](https://docs.renovatebot.com/)
- [Kubernetes Documentation](https://kubernetes.io/docs/)

---

**Note**: This repository is configured for a staging environment. Adapt configurations for production use with appropriate security measures and resource allocations.
