# Helm Charts Templates - Universal Microservice Chart

Repositório central com um **Helm Chart genérico e reutilizável** para deploy de **qualquer tipo de aplicação/microserviço** em Kubernetes.

## ✨ Suporte Universal

Este chart foi design para funcionar com:

- **Backend**: Java (Spring Boot), Node.js (Express, Nest.js), Python (FastAPI, Flask), Go, Rust, .NET, PHP
- **Frontend**: React, Vue, Angular, Next.js, Svelte (qualquer SPA/SSG)
- **Bancos**: Database sidecars, cache servers, message brokers
- **Qualquer imagem**: Desde que esteja em um registry Docker

## 📁 Estrutura

```
charts/
└── k8s-helm-templates/
    ├── Chart.yaml              # Metadata do chart
    ├── values.yaml             # Defaults (GENÉRICOS, sem specifics)
    └── templates/
        ├── deployment.yaml     # Deployment principal
        ├── service.yaml        # Service para exposição
        ├── ingress.yaml        # Ingress para roteamento
        ├── configmap.yaml      # ConfigMaps
        ├── secret.yaml         # Secrets
        ├── hpa.yaml            # Horizontal Pod Autoscaler
        ├── pdb.yaml            # Pod Disruption Budget
        ├── networkpolicy.yaml  # Network policies
        └── _helpers.tpl        # Helpers
```

## 🚀 Como Usar - Para Qualquer Aplicação

### 1. Add Helm Repository

```bash
helm repo add k8s-templates https://rg-hermann.github.io/k8s-helm-templates/
helm repo update
```

### 2. Criar seu próprio `values.yaml`

No **seu repositório de infra** (ex: `python-bootstrap-infra`), crie `values-dev.yaml`:

```yaml
# values-dev.yaml - Configuração específica da sua aplicação

image:
  repository: ghcr.io/rg-hermann/python-bootstrap  # SEU REPOSITÓRIO
  tag: ""  # Will be auto-updated by CI/CD
  pullPolicy: Always

containerPort: 8080  # Porta que sua app escuta

replicaCount: 2

probes:
  enabled: true  # Habilita health checks
  
livenessProbe:
  httpGet:
    path: /health  # Configure para seu framework
    port: 8080

readinessProbe:
  httpGet:
    path: /health  # Configure para seu framework
    port: 8080

autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 5

resources:
  limits:
    cpu: 300m
    memory: 384Mi
  requests:
    cpu: 100m
    memory: 128Mi

service:
  type: ClusterIP
  port: 80
  targetPort: 8080

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: my-app.dev.local
      paths:
        - path: /
          pathType: Prefix
```

### 3. Install

```bash
helm install my-app k8s-templates/k8s-helm-templates -f values-dev.yaml
```

---

## 📋 Exemplos por Linguagem/Framework

### Spring Boot (Java)

```yaml
# values.yaml
image:
  repository: ghcr.io/myorg/spring-app
  tag: v1.0.0

containerPort: 8080

probes:
  enabled: true

livenessProbe:
  httpGet:
    path: /actuator/health/liveness
    port: 8080

readinessProbe:
  httpGet:
    path: /actuator/health/readiness
    port: 8080
```

### FastAPI (Python)

```yaml
# values.yaml
image:
  repository: ghcr.io/myorg/fastapi-app
  tag: v1.0.0

containerPort: 8000  # FastAPI default

probes:
  enabled: true

livenessProbe:
  httpGet:
    path: /health
    port: 8000

readinessProbe:
  httpGet:
    path: /health
    port: 8000

env:
  LOG_LEVEL: INFO
  APP_ENV: production
```

### Express.js (Node.js)

```yaml
# values.yaml
image:
  repository: ghcr.io/myorg/express-app
  tag: v1.0.0

containerPort: 3000

probes:
  enabled: true

livenessProbe:
  httpGet:
    path: /health
    port: 3000

readinessProbe:
  httpGet:
    path: /healthz
    port: 3000
```

### React SPA (Frontend)

```yaml
# values.yaml
image:
  repository: ghcr.io/myorg/react-app
  tag: v1.0.0

containerPort: 80

probes:
  enabled: false  # Frontend geralmente não tem health checks

service:
  type: ClusterIP
  port: 80
  targetPort: 80

ingress:
  enabled: true
  className: nginx
  hosts:
    - host: my-app.example.com
      paths:
        - path: /
          pathType: Prefix
```

### .NET Core (ASP.NET)

```yaml
# values.yaml
image:
  repository: ghcr.io/myorg/dotnet-app
  tag: v1.0.0

containerPort: 5000

probes:
  enabled: true

livenessProbe:
  httpGet:
    path: /health
    port: 5000

readinessProbe:
  httpGet:
    path: /healthz
    port: 5000
```

---

## 🔧 Configurações Importantes

### Sempre Configure

| Parâmetro | Descrição | Exemplo |
|-----------|-----------|---------|
| `image.repository` | OBRIGATÓRIO | `ghcr.io/myorg/my-app` |
| `containerPort` | Porta que sua app escuta | `8080`, `3000`, `80` |
| `replicaCount` | Números de pods | `1`, `2`, `3` |
| `resources` | CPU/Memory | Ver exemplos acima |

### Configure Se Aplicável

| Parâmetro | Quando Usar |
|-----------|------------|
| `probes.enabled` | Sua app tem endpoints `/health` ou similar |
| `livenessProbe.httpGet.path` | Se `probes.enabled: true`, set o path correto |
| `ingress.enabled` | Se precisa expor externamente |
| `autoscaling.enabled` | Em produção |
| `env` | Se precisa injetar variáveis de ambiente |
| `config` | ConfigMaps (non-sensitive) |
| `secrets` | Secrets (sensitive data) |

### Não Configure (Deixe Default)

| Parâmetro | Por Quê |
|-----------|---------|
| `podSecurityContext` | Sensible default já aplicado |
| `securityContext` | Sensible default já aplicado |
| `networkPolicy` | Enable só em produção |
| `podDisruptionBudget` | Enable só em produção |

---

## 🏥 Health Checks - Framework Específico

Configure `probes.enabled: true` e customize os paths:

```yaml
# Spring Boot
livenessProbe:
  httpGet:
    path: /actuator/health/liveness

# Python FastAPI/Flask
livenessProbe:
  httpGet:
    path: /health

# Node.js Express
livenessProbe:
  httpGet:
    path: /health

# Go (chi, gin)
livenessProbe:
  httpGet:
    path: /health

# .NET
livenessProbe:
  httpGet:
    path: /health

# Rails
livenessProbe:
  httpGet:
    path: /health_check
```

**Se sua app NÃO expõe health checks, deixe `probes.enabled: false`**

---

## 📦 Variáveis de Ambiente

Injetar variáveis:

```yaml
env:
  DATABASE_URL: "postgresql://db:5432/myapp"
  CACHE_ENABLED: "true"
  LOG_LEVEL: "INFO"
  ENVIRONMENT: "production"
```

Sensibles devem ir em `secrets`:

```yaml
secrets:
  DATABASE_PASSWORD: "secret-password"
  API_KEY: "sk-..."
  JWT_SECRET: "super-secret"
```

---

## 🔐 Segurança

O chart vem com **defaults seguros**:

- ✅ Runs como non-root (UID 1000)
- ✅ Dropped Linux capabilities
- ✅ SecurityContext aplicado
- ✅ Network policies (habilitar em prod)
- ✅ Pod disruption budgets (habilitar em prod)

---

## 🚄 Auto-scaling

Habilitar em ambientes com tráfego variável:

```yaml
autoscaling:
  enabled: true
  minReplicas: 2
  maxReplicas: 10
  targetCPUUtilizationPercentage: 70
  targetMemoryUtilizationPercentage: 80
```

---

## 📚 Validação Local

Antes de fazer deploy, valide seu values.yaml:

```bash
# Add repository
helm repo add k8s-templates https://rg-hermann.github.io/k8s-helm-templates/
helm repo update

# Lint
helm lint . --strict

# Template (veja o YAML gerado)
helm template my-release k8s-templates/k8s-helm-templates -f values.yaml

# Dry-run (simula deployment)
helm install my-release k8s-templates/k8s-helm-templates -f values.yaml --dry-run
```

---

## 📁 Estrutura Típica em um Repo de Infra

```
seu-repo-infra/
├── Chart.yaml
├── values-dev.yaml
├── values-prod.yaml
├── values-staging.yaml  (optional)
├── README.md
└── .github/workflows/
    └── helm-validate.yml
```

---

## 🔄 CI/CD Integration

O chart é agnóstico a CI/CD, mas integra bem com:

- **GitHub Actions**: Auto-update image tags
- **GitLab CI**: Deploy automático
- **ArgoCD**: GitOps automation
- **Flux**: Continuous deployment

---

## 🤝 Contribuindo

Para melhorar o chart:

1. Clone: `git clone https://github.com/rg-hermann/k8s-helm-templates.git`
2. Modifique templates em `charts/k8s-helm-templates/templates/`
3. Update `Chart.yaml` version
4. Push e crie PR

---

## 📞 Suporte

- GitHub Issues: [rg-hermann/k8s-helm-templates/issues](https://github.com/rg-hermann/k8s-helm-templates/issues)
- Exemplos: Ver pasta `examples/` no repositório

---

**Última atualização**: Maio 23, 2026  
**Versão**: 0.1.0  
**Status**: ✅ Production Ready
configmap:
   enabled: true
   data:
      ENV: "production"
      DEBUG: "false"
```

### Ingress

```yaml
ingress:
   enabled: true
   host: "minha-app.local"
```

## Personalização

Adapte os templates conforme as necessidades do seu ambiente. O chart é flexível e pode ser estendido para outros stacks e recursos Kubernetes.

---

Dúvidas ou sugestões? Edite este README ou contribua com novos templates!
