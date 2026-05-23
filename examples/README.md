# Examples - Using k8s-helm-templates for Different Frameworks

This folder contains **ready-to-use examples** of `values.yaml` files for deploying different types of applications with the `k8s-helm-templates` Helm chart.

## 📚 Available Examples

| Framework | Language | File | Health Check |
|-----------|----------|------|--------------|
| Spring Boot | Java | `values-spring-boot.yaml` | `/actuator/health/liveness` |
| FastAPI | Python | `values-python-fastapi.yaml` | `/health` |
| Express.js | Node.js | `values-nodejs-express.yaml` | `/health` |
| React/Vue/Angular | Frontend | `values-frontend-spa.yaml` | Disabled |
| ASP.NET Core | .NET | `values-dotnet-aspnet.yaml` | `/health` |
| Gin/Chi/Echo | Go | `values-go.yaml` | `/health` |

## 🚀 How to Use

### Step 1: Copy the appropriate example

```bash
# For a Spring Boot Java app
cp examples/values-spring-boot.yaml /path/to/your-infra-repo/values-prod.yaml

# For a Python FastAPI app
cp examples/values-python-fastapi.yaml /path/to/your-infra-repo/values-prod.yaml

# For a Node.js Express app
cp examples/values-nodejs-express.yaml /path/to/your-infra-repo/values-prod.yaml
```

### Step 2: Customize for your application

Edit the values file and change:

```yaml
# 1. Change image repository (REQUIRED!)
image:
  repository: ghcr.io/YOUR-ORG/YOUR-APP  # <- Change this

# 2. Adjust port if different
containerPort: 8080  # <- Change if your app uses different port

# 3. Configure health check paths (if enabled)
livenessProbe:
  httpGet:
    path: /health  # <- Change to your framework's endpoint

# 4. Set resource limits based on your app
resources:
  limits:
    cpu: 300m      # <- Adjust for your needs
    memory: 384Mi   # <- Adjust for your needs

# 5. Configure environment variables
env:
  LOG_LEVEL: "INFO"
  # <- Add your app's specific env vars
```

### Step 3: Validate

```bash
# Add the Helm repo
helm repo add k8s-templates https://rg-hermann.github.io/k8s-helm-templates/
helm repo update

# Validate the values
helm template my-release k8s-templates/k8s-helm-templates -f values-prod.yaml

# Dry-run deployment
helm install my-release k8s-templates/k8s-helm-templates -f values-prod.yaml --dry-run
```

### Step 4: Deploy

```bash
helm install my-release k8s-templates/k8s-helm-templates -f values-prod.yaml
```

---

## 🔍 Key Differences by Framework

### Health Checks

**Spring Boot** (Java):
```yaml
livenessProbe:
  httpGet:
    path: /actuator/health/liveness  # Spring Boot specific
```

**FastAPI** (Python):
```yaml
livenessProbe:
  httpGet:
    path: /health  # Generic, you must implement
```

**Frontend (React/Vue/Angular)**:
```yaml
probes:
  enabled: false  # Frontends don't have health endpoints
```

### Ports

- Spring Boot: `8080`
- FastAPI: `8000`
- Express.js: `3000`
- Frontend: `80`
- ASP.NET: `80`
- Go: `8000`

### Resources

**Frontend (light)**:
```yaml
resources:
  limits:
    cpu: 200m
    memory: 256Mi
```

**Backend Java (heavier)**:
```yaml
resources:
  limits:
    cpu: 500m
    memory: 512Mi
```

**Frontend SPA in CDN**: Can be very small (50m CPU, 64Mi RAM)

### Autoscaling

**Disabled** for simple apps:
```yaml
autoscaling:
  enabled: false
```

**Enabled** for production:
```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
```

---

## 🛠️ Framework-Specific Tips

### Spring Boot
- Use `application.properties` or `application.yml` in ConfigMap
- Health endpoints are automatic via actuator
- Set `JAVA_TOOL_OPTIONS` for memory management
- Initialize DB migrations before deployment

### FastAPI
- Ensure app listens on `0.0.0.0` (not `localhost`)
- Implement `/health` endpoint yourself
- Use Uvicorn workers: `--workers 4`
- Set `PYTHONUNBUFFERED=1`

### Express.js
- Ensure app listens on `0.0.0.0`
- Implement `/health` and `/healthz` endpoints
- Use process manager (PM2) in multi-worker setup
- Handle graceful shutdown properly

### Frontend SPA
- No health checks needed
- Pre-compress assets (gzip)
- Set `Cache-Control` headers properly
- Use CDN for static assets (optional)

### ASP.NET Core
- Ensure app listens on `0.0.0.0:80`
- Use health checks middleware
- Set `ASPNETCORE_ENVIRONMENT=Production`
- Configure logging properly

### Go
- Ensure app listens on `0.0.0.0`
- Implement `/health` endpoint
- Use static compilation for security
- Consider multi-stage Docker builds

---

## 📋 Checklist Before Deploying

- [ ] Image repository set correctly
- [ ] Container port matches your application
- [ ] Health check path is correct (if enabled)
- [ ] Resource limits are appropriate
- [ ] Environment variables are set
- [ ] Secrets are not in ConfigMap (use `secrets` field)
- [ ] Ingress host/domain is correct
- [ ] TLS certificate is configured (if needed)
- [ ] Networking/firewall rules allow traffic
- [ ] Storage (PVC) configured if needed

---

## 🔗 Related

- [k8s-helm-templates Main README](../README.md)
- [Helm Chart Values Reference](../charts/k8s-helm-templates/values.yaml)
- [Kubernetes Probes Documentation](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

---

**Need help?** Open an issue: [github.com/rg-hermann/k8s-helm-templates/issues](https://github.com/rg-hermann/k8s-helm-templates/issues)
