# Monitoring

This directory contains the complete observability stack for the Kubernetes cluster, providing metrics collection, visualization, and alerting capabilities. The monitoring infrastructure is built around the Prometheus ecosystem and managed through GitOps.

## 📁 Structure

```
monitoring/
├── controllers/                # Monitoring component controllers
│   ├── base/                  # Base monitoring configurations
│   │   └── kube-prometheus-stack/
│   │       ├── kustomization.yaml
│   │       ├── namespace.yaml
│   │       ├── release.yaml
│   │       └── repository.yaml
│   └── staging/               # Staging environment overlays
│       ├── kustomization.yaml
│       └── kube-prometheus-stack/
│           └── kustomization.yaml
└── configs/                   # Monitoring configurations
    └── staging/
        ├── kustomization.yaml
        └── kube-prometheus-stack/
            ├── grafana-tls-secret.yaml
            └── kustomization.yaml
```

## 🏗️ Architecture Overview

### Monitoring Stack Components
- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **AlertManager**: Alert routing and notification
- **Node Exporter**: Host-level metrics collection
- **kube-state-metrics**: Kubernetes object metrics
- **Prometheus Operator**: Kubernetes-native Prometheus management

### Data Flow
1. **Metrics Collection**: Exporters and instrumented applications expose metrics
2. **Scraping**: Prometheus scrapes metrics from configured targets
3. **Storage**: Time-series data stored in Prometheus TSDB
4. **Visualization**: Grafana queries Prometheus for dashboard rendering
5. **Alerting**: AlertManager processes and routes alerts based on rules

## 🔧 Component Details

### Kube-Prometheus-Stack

**Comprehensive monitoring solution for Kubernetes clusters**

#### Helm Release Configuration
```yaml
Chart: kube-prometheus-stack
Version: 66.2.2
Repository: prometheus-community
Namespace: monitoring
```

#### Key Features
- **CRD Management**: Automatic creation and updates of Custom Resource Definitions
- **Drift Detection**: Monitors and corrects configuration drift
- **High Availability**: Multi-replica deployments for critical components
- **Persistent Storage**: Configurable storage for metrics retention

### Grafana Configuration

#### Access Details
- **URL**: https://grs.ludfrancisco.com
- **Authentication**: Admin credentials configured
- **Ingress**: Traefik-based routing with TLS termination
- **TLS Secret**: `grafana-tls-secret` for HTTPS encryption

#### Dashboard Features
- **Pre-configured Dashboards**: Kubernetes cluster monitoring
- **Custom Dashboards**: Application-specific metrics visualization
- **Alerting**: Visual alert management and notification
- **Data Sources**: Prometheus integration for metrics queries

#### Security Configuration
```yaml
Ingress Class: traefik
TLS Enabled: true
Certificate: grafana-tls-secret
Domain: grs.ludfrancisco.com
```

### Prometheus Configuration

#### Metrics Collection
- **Cluster Metrics**: Node, pod, and service metrics
- **Application Metrics**: Custom application instrumentation
- **Infrastructure Metrics**: Storage, network, and compute resources
- **Security Metrics**: RBAC and security policy compliance

#### Storage and Retention
- **Time Series Database**: Efficient storage of metrics data
- **Retention Policy**: Configurable data retention periods
- **Backup Strategy**: Regular snapshots and disaster recovery
- **Performance Optimization**: Query optimization and resource tuning

### AlertManager

#### Alert Processing
- **Rule Evaluation**: Prometheus evaluates alerting rules
- **Alert Routing**: Routes alerts based on labels and severity
- **Notification Channels**: Email, Slack, webhook integrations
- **Silencing**: Temporary suppression of known issues

#### Alert Categories
- **Infrastructure Alerts**: Node failures, resource exhaustion
- **Application Alerts**: Service unavailability, performance degradation
- **Security Alerts**: Unauthorized access, policy violations
- **Capacity Alerts**: Storage, memory, and CPU thresholds

## 🔐 Security Implementation

### Access Control
- **RBAC**: Role-based access control for monitoring components
- **Service Accounts**: Dedicated accounts with minimal permissions
- **Network Policies**: Restricted network access between components
- **TLS Encryption**: Secure communication between all components

### Data Protection
- **Encrypted Storage**: Sensitive configuration encrypted with SOPS
- **Secret Management**: Kubernetes secrets for credentials
- **Audit Logging**: All monitoring access and changes logged
- **Data Retention**: Compliance with data retention policies

### Certificate Management
- **TLS Certificates**: Automated certificate provisioning
- **Certificate Rotation**: Regular certificate renewal
- **CA Management**: Certificate authority configuration
- **Ingress Security**: HTTPS enforcement for external access

## 🚀 Deployment Process

### Helm-based Deployment
1. **Repository Setup**: Helm repository configuration
2. **Release Management**: Helm release lifecycle management
3. **Configuration Overlay**: Environment-specific customizations
4. **Health Checks**: Automated deployment validation

### GitOps Integration
- **Flux Management**: Continuous reconciliation of monitoring stack
- **Configuration Drift**: Automatic detection and correction
- **Version Control**: All changes tracked through Git
- **Rollback Capability**: Easy rollback to previous configurations

## 📊 Monitoring Capabilities

### Cluster Monitoring
- **Node Health**: CPU, memory, disk, and network metrics
- **Pod Metrics**: Resource usage and performance indicators
- **Service Monitoring**: Endpoint availability and response times
- **Ingress Metrics**: Traffic patterns and error rates

### Application Monitoring
- **Custom Metrics**: Application-specific performance indicators
- **Business Metrics**: Key performance indicators (KPIs)
- **Error Tracking**: Application error rates and patterns
- **Performance Profiling**: Detailed performance analysis

### Infrastructure Monitoring
- **Storage Metrics**: Disk usage, I/O performance, and capacity
- **Network Metrics**: Bandwidth utilization and connectivity
- **Security Metrics**: Authentication failures and policy violations
- **Capacity Planning**: Resource utilization trends and forecasting

## 🛠️ Management Operations

### Check Monitoring Stack Status
```bash
# View monitoring namespace resources
kubectl get all -n monitoring

# Check Helm release status
helm list -n monitoring

# Verify Prometheus targets
kubectl port-forward -n monitoring svc/kube-prometheus-stack-prometheus 9090:9090
# Access http://localhost:9090/targets

# Check Grafana accessibility
kubectl port-forward -n monitoring svc/kube-prometheus-stack-grafana 3000:80
# Access http://localhost:3000
```

### Prometheus Operations
```bash
# Check Prometheus configuration
kubectl get prometheus -n monitoring

# View Prometheus rules
kubectl get prometheusrules -n monitoring

# Check service monitors
kubectl get servicemonitors -n monitoring

# Verify alert manager
kubectl get alertmanagers -n monitoring
```

### Grafana Management
```bash
# Get Grafana admin password
kubectl get secret -n monitoring kube-prometheus-stack-grafana -o jsonpath="{.data.admin-password}" | base64 --decode

# Check Grafana configuration
kubectl get configmap -n monitoring kube-prometheus-stack-grafana

# View Grafana logs
kubectl logs -n monitoring deployment/kube-prometheus-stack-grafana
```

### Troubleshooting
```bash
# Check Helm release status
helm status kube-prometheus-stack -n monitoring

# View Prometheus operator logs
kubectl logs -n monitoring deployment/kube-prometheus-stack-operator

# Check CRD status
kubectl get crds | grep monitoring

# Verify service discovery
kubectl get endpoints -n monitoring
```

## 📈 Dashboard Management

### Pre-configured Dashboards
- **Kubernetes Cluster Overview**: High-level cluster health
- **Node Metrics**: Individual node performance
- **Pod Metrics**: Container resource usage
- **Ingress Monitoring**: Traffic and performance metrics

### Custom Dashboard Creation
1. **Access Grafana**: Navigate to https://grs.ludfrancisco.com
2. **Create Dashboard**: Use Grafana UI to build custom dashboards
3. **Export Configuration**: Export dashboard JSON for version control
4. **GitOps Integration**: Store dashboard configurations in Git

### Dashboard Best Practices
- **Consistent Naming**: Use standardized dashboard naming conventions
- **Meaningful Metrics**: Focus on actionable metrics and KPIs
- **Alert Integration**: Link dashboards to relevant alerts
- **Documentation**: Include descriptions and usage instructions

## 🚨 Alerting Configuration

### Alert Rules
- **Infrastructure Alerts**: Node down, high resource usage
- **Application Alerts**: Service unavailable, high error rates
- **Security Alerts**: Failed authentication, policy violations
- **Capacity Alerts**: Storage full, memory exhaustion

### Notification Channels
- **Email**: Critical alerts sent to operations team
- **Slack**: Real-time notifications for immediate response
- **Webhooks**: Integration with external incident management
- **PagerDuty**: Escalation for critical production issues

### Alert Management
```bash
# View active alerts
kubectl port-forward -n monitoring svc/kube-prometheus-stack-alertmanager 9093:9093
# Access http://localhost:9093

# Check alert rules
kubectl get prometheusrules -n monitoring

# View alert manager configuration
kubectl get secret -n monitoring alertmanager-kube-prometheus-stack-alertmanager
```

## 📝 Configuration Management

### Helm Values Customization
- **Resource Limits**: CPU and memory constraints
- **Storage Configuration**: Persistent volume settings
- **Ingress Settings**: External access configuration
- **Security Policies**: RBAC and network policies

### Environment-Specific Settings
- **Staging Configuration**: Development and testing settings
- **Production Configuration**: High-availability and performance tuning
- **Resource Scaling**: Environment-appropriate resource allocation
- **Retention Policies**: Data retention based on environment needs

## 🔄 Maintenance and Updates

### Regular Maintenance Tasks
- **Chart Updates**: Regular updates to kube-prometheus-stack
- **Dashboard Reviews**: Periodic review and optimization of dashboards
- **Alert Tuning**: Adjustment of alert thresholds and rules
- **Performance Optimization**: Query optimization and resource tuning

### Backup and Recovery
- **Configuration Backup**: Regular backup of Grafana dashboards and settings
- **Metrics Backup**: Long-term storage of critical metrics data
- **Disaster Recovery**: Procedures for monitoring stack recovery
- **Testing**: Regular testing of backup and recovery procedures

## 📋 Best Practices

### Monitoring Strategy
- **Golden Signals**: Focus on latency, traffic, errors, and saturation
- **SLI/SLO Definition**: Define service level indicators and objectives
- **Alert Fatigue**: Minimize false positives and noise
- **Observability**: Comprehensive visibility into system behavior

### Performance Optimization
- **Query Optimization**: Efficient PromQL queries for dashboards
- **Resource Management**: Appropriate resource allocation for components
- **Data Retention**: Balance between storage costs and data availability
- **Scrape Intervals**: Optimize collection frequency for different metrics

### Security
- **Access Control**: Implement proper RBAC for monitoring access
- **Data Encryption**: Encrypt sensitive monitoring data
- **Audit Logging**: Track all monitoring system access and changes
- **Compliance**: Ensure monitoring meets regulatory requirements

## 🔗 Related Documentation

- [Main Repository README](../README.md)
- [Applications Documentation](../apps/README.md)
- [Infrastructure Documentation](../infrastructure/README.md)
- [Cluster Configuration](../clusters/README.md)
- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)

---

**Note**: The monitoring stack is critical for cluster operations and application health. Changes should be carefully planned and tested to avoid disruption to observability capabilities.
