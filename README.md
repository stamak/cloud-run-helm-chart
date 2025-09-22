# Cloud Run Helm Chart

A practical solution for deploying Google Cloud Run services using Helm's templating capabilities to overcome the limitations of manual `gcloud` deployments.

## Why Use Helm for Cloud Run?

While `gcloud run deploy` works great for single services, it quickly becomes unwieldy for complex applications. This Helm chart addresses common pain points:

- **Eliminates Configuration Drift**: No more scattered shell scripts or manual copy-pasting of deployment commands
- **Environment Consistency**: Single chart with environment-specific values files
- **Declarative Infrastructure**: Your entire service configuration as version-controlled code
- **Simplified CI/CD**: Two-command deployment process that integrates seamlessly into pipelines

## Quick Start

```bash
# 1. Generate the Cloud Run manifest
helm template my-app ./cloud-run > service.yaml

# 2. Deploy to Cloud Run
gcloud run services replace service.yaml --region=$REGION
```

## Chart Structure

```
cloud-run/
├── Chart.yaml          # Chart metadata
├── values.yaml         # Default configuration values
└── templates/
    ├── _helpers.tpl    # Template helpers
    └── service.yml     # Knative Service template
```

## Configuration

### Basic Setup

```yaml
# values.yaml
appName: "my-awesome-app"
region: "us-central1"
serviceAccountName: "my-service-account"

image:
  repository: "gcr.io/my-project/my-app"
  tag: "v1.0.0"

service:
  port: 8000
```

### Environment Variables

```yaml
env:
  NODE_ENV: "production"
  DATABASE_URL: "postgresql://..."
  LOG_LEVEL: "info"
```

### GCP Secrets Integration

```yaml
gcp_secrets:
  DATABASE_PASSWORD:
    name: "db-password"
    version: "1"
  API_SECRET:
    name: "api-secret"
    version: "latest"  # Optional, defaults to 'latest'
```

### Scaling & Performance

```yaml
# Scaling configuration
maxScale: 20
minScale: 1
concurrency: 80
timeoutSeconds: 300

# Resource limits
resources:
  limits:
    cpu: "1000m"
    memory: "512Mi"
  requests:
    cpu: "100m"
    memory: "128Mi"
```

## Multi-Environment Deployment

### Development Environment

```yaml
# values-dev.yaml
appName: "my-app-dev"
image:
  tag: "develop"
env:
  NODE_ENV: "development"
  LOG_LEVEL: "debug"
maxScale: 5
```

### Production Environment

```yaml
# values-prod.yaml
appName: "my-app-prod"
image:
  tag: "v1.2.3"
env:
  NODE_ENV: "production"
  LOG_LEVEL: "warn"
maxScale: 100
minScale: 2
resources:
  limits:
    cpu: "2000m"
    memory: "1Gi"
```

Deploy with environment-specific values:

```bash
# Development
helm template my-app-dev ./cloud-run -f values-dev.yaml > dev-service.yaml
gcloud run services replace dev-service.yaml --region=us-central1

# Production
helm template my-app-prod ./cloud-run -f values-prod.yaml > prod-service.yaml
gcloud run services replace prod-service.yaml --region=us-central1
```

### Debugging Commands

```bash
# Render template with debug info
helm template my-app ./cloud-run --debug

# Check differences between environments
helm template my-app ./cloud-run -f values-dev.yaml > dev.yaml
helm template my-app ./cloud-run -f values-prod.yaml > prod.yaml
diff dev.yaml prod.yaml
```


## Why This Approach Works

This chart embodies the principles outlined in ["Scaling Up: Why Your Cloud Run Deployments Need Helm"](https://stamak.github.io/2025/CloudRunAndHelm/):

- **Single Source of Truth**: All configuration in version-controlled files
- **Template Reusability**: One chart, multiple environments
- **Atomic Deployments**: Deploy entire application stack with confidence
- **CI/CD Ready**: Simple two-step deployment process

## Contributing

1. Fork this repository
2. Create a feature branch
3. Test your changes with `helm template`
4. Submit a pull request

For more context on why this approach is beneficial, read the full blog post: [Scaling Up: Why Your Cloud Run Deployments Need Helm](https://stamak.github.io/2025/CloudRunAndHelm/).
