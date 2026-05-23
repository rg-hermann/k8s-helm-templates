# Como Usar k8s-helm-templates em Seus Projetos

Documento explicando como **python-bootstrap**, **java-bootstrap** e **qualquer outro microserviço** deve usar o `k8s-helm-templates` chart genérico.

## 🎯 Arquitetura

```
┌─────────────────────────────────────────────────────────────┐
│                   k8s-helm-templates (Shared)               │
│                  Generic Helm Chart (Agnóstico)             │
│                                                             │
│  - Deployment template (funciona com qualquer imagem)      │
│  - Service, Ingress, ConfigMap, Secret templates           │
│  - Health checks (OPCIONAIS)                               │
│  - Autoscaling, PDB, NetworkPolicy templates               │
│                                                             │
│  Default values: GENÉRICOS e VAZIOS                        │
│  (sem assumir Python, Java, Node, etc)                     │
└─────────────────────────────────────────────────────────────┘
         △                 △                 △
         │                 │                 │
    ┌────┴────┐      ┌─────┴────┐      ┌────┴────┐
    │          │      │           │      │         │
    │ Python   │      │   Java    │      │ Node.js │
    │Bootstrap │      │ Bootstrap │      │   API   │
    │  Infra   │      │   Infra   │      │  Infra  │
    │          │      │           │      │         │
    └──────────┘      └───────────┘      └─────────┘
    
    Cada um tem seu:
    - values-dev.yaml
    - values-prod.yaml
    - Chart.yaml (dependency)
```

## 📁 Estrutura de Cada Aplicação

### python-bootstrap-infra

```
python-bootstrap-infra/
├── Chart.yaml                    # Declares dependency
├── values-dev.yaml               # PYTHON-SPECIFIC dev config
├── values-prod.yaml              # PYTHON-SPECIFIC prod config
├── README.md
└── .github/workflows/
    └── helm-test.yml
```

**Chart.yaml**:
```yaml
apiVersion: v2
name: python-bootstrap-infra
description: GitOps configuration for Python Bootstrap Application
type: application
version: 1.0.1
appVersion: "1.0.1"

dependencies:
  - name: k8s-helm-templates      # Generic chart
    version: "0.1.0"
    repository: "https://rg-hermann.github.io/k8s-helm-templates/"
```

**values-dev.yaml** (Python-specific):
```yaml
k8s-helm-templates:
  image:
    repository: ghcr.io/rg-hermann/python-bootstrap
    tag: ""
    pullPolicy: Always
  
  containerPort: 8000  # FastAPI default
  
  probes:
    enabled: true
  
  livenessProbe:
    httpGet:
      path: /health   # Python app endpoint
      port: 8000
  
  readinessProbe:
    httpGet:
      path: /health
      port: 8000
  
  # ... other Python-specific config
```

### java-bootstrap-infra

```
java-bootstrap-infra/
├── Chart.yaml                    # Declares dependency
├── values-dev.yaml               # JAVA-SPECIFIC dev config
├── values-prod.yaml              # JAVA-SPECIFIC prod config
├── README.md
└── .github/workflows/
    └── helm-test.yml
```

**Chart.yaml**:
```yaml
apiVersion: v2
name: java-bootstrap-infra
description: GitOps configuration for Java Bootstrap Application
type: application
version: 1.0.0
appVersion: "1.0.0"

dependencies:
  - name: k8s-helm-templates      # Same generic chart!
    version: "0.1.0"
    repository: "https://rg-hermann.github.io/k8s-helm-templates/"
```

**values-dev.yaml** (Java-specific):
```yaml
k8s-helm-templates:
  image:
    repository: ghcr.io/rg-hermann/java-bootstrap
    tag: ""
    pullPolicy: Always
  
  containerPort: 8080  # Spring Boot default
  
  probes:
    enabled: true
  
  livenessProbe:
    httpGet:
      path: /actuator/health/liveness  # Spring Boot endpoint!
      port: 8080
  
  readinessProbe:
    httpGet:
      path: /actuator/health/readiness
      port: 8080
  
  env:
    SPRING_PROFILES_ACTIVE: "production"
    JAVA_TOOL_OPTIONS: "-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
  
  # ... other Java-specific config
```

---

## ✅ Key Points

### O Chart é Genérico

✅ **Suporta**:
- Python (FastAPI, Flask, Django)
- Java (Spring Boot, Quarkus, Micronaut)
- Node.js (Express, Nest.js, Fastify)
- Go (Gin, Chi, Echo)
- .NET (ASP.NET Core)
- Rust, PHP, Ruby, ou qualquer container

### Cada App Tem Seu Próprio Valores

✅ **python-bootstrap-infra/values-dev.yaml**:
- Aponta para `ghcr.io/rg-hermann/python-bootstrap`
- Configura `containerPort: 8000`
- Usa `/health` para probes

✅ **java-bootstrap-infra/values-dev.yaml**:
- Aponta para `ghcr.io/rg-hermann/java-bootstrap`
- Configura `containerPort: 8080`
- Usa `/actuator/health/liveness` para probes

### O Chart Não Assume Nada

❌ **NÃO faz**:
- ❌ Não assume que será Python
- ❌ Não assume que será Java
- ❌ Não força health checks
- ❌ Não força autoscaling
- ❌ Não força network policies

✅ **FAZ**:
- ✅ Fornece templates reutilizáveis
- ✅ Deixa tudo configurável via values
- ✅ Sensible defaults opcionais
- ✅ Documentação com exemplos

---

## 🔄 Workflow em Prática

### 1. Ao Criar uma Nova Aplicação

```bash
# Criar infra repo
mkdir meu-app-infra
cd meu-app-infra

# Inicializar com Chart.yaml
cat > Chart.yaml << EOF
apiVersion: v2
name: meu-app-infra
version: 1.0.0
appVersion: "1.0.0"
dependencies:
  - name: k8s-helm-templates
    version: "0.1.0"
    repository: "https://rg-hermann.github.io/k8s-helm-templates/"
EOF

# Download dependency
helm dependency update

# Copiar exemplo apropriado
cp k8s-helm-templates/examples/values-nodejs-express.yaml values-dev.yaml

# Customizar para sua app
# ... editar image.repository, porta, etc ...
```

### 2. Validar Localmente

```bash
# Helm lint
helm lint . --strict

# Template rendering
helm template meu-app . -f values-dev.yaml

# Dry-run deployment
helm install meu-app . -f values-dev.yaml --dry-run=client
```

### 3. Deploy

```bash
# To cluster
helm install meu-app . -f values-dev.yaml

# Ou com ArgoCD
# Commit values-dev.yaml e deixar ArgoCD auto-deploy
```

---

## 📊 Comparação: Antes vs Depois

### ANTES (Problema)

```
python-bootstrap-infra/
  Chart.yaml (customizado para Python)
  values-dev.yaml (hardcoded /health)
  templates/ (cópia do Python templates)

java-bootstrap-infra/
  Chart.yaml (customizado para Java)
  values-dev.yaml (hardcoded /actuator/health)
  templates/ (cópia do Java templates)
  
PROBLEMA: Templates duplicadas, sem DRY, difícil manter
```

### DEPOIS (Solução)

```
k8s-helm-templates/
  charts/k8s-helm-templates/
    Chart.yaml (genérico)
    values.yaml (defaults agnósticos)
    templates/ (reutilizável)
    examples/ (reference implementations)

python-bootstrap-infra/
  Chart.yaml (declarou dependency)
  values-dev.yaml (Python-specific values)
  
java-bootstrap-infra/
  Chart.yaml (declarou dependency)
  values-dev.yaml (Java-specific values)

qualquer-outra-app-infra/
  Chart.yaml (declarou dependency)
  values-dev.yaml (App-specific values)
  
SOLUÇÃO: Uma única source of truth, reutilizável, DRY
```

---

## 🎓 Exemplos Práticos

### Exemplo 1: Python FastAPI App

```yaml
# python-bootstrap-infra/values-prod.yaml

k8s-helm-templates:
  image:
    repository: ghcr.io/rg-hermann/python-bootstrap
    tag: "v1.0.0"  # pinned version
    pullPolicy: IfNotPresent
  
  containerPort: 8000
  replicaCount: 3
  
  probes:
    enabled: true
  
  livenessProbe:
    httpGet:
      path: /health
      port: 8000
    initialDelaySeconds: 30
  
  resources:
    limits:
      cpu: 300m
      memory: 384Mi
  
  env:
    LOG_LEVEL: "WARN"
    WORKERS: "4"
```

### Exemplo 2: Java Spring Boot App

```yaml
# java-bootstrap-infra/values-prod.yaml

k8s-helm-templates:
  image:
    repository: ghcr.io/rg-hermann/java-bootstrap
    tag: "v2.0.0"  # pinned version
    pullPolicy: IfNotPresent
  
  containerPort: 8080
  replicaCount: 3
  
  probes:
    enabled: true
  
  livenessProbe:
    httpGet:
      path: /actuator/health/liveness
      port: 8080
    initialDelaySeconds: 40  # Java takes longer
  
  readinessProbe:
    httpGet:
      path: /actuator/health/readiness
      port: 8080
    initialDelaySeconds: 20
  
  resources:
    limits:
      cpu: 500m       # Java precisa de mais CPU
      memory: 512Mi   # Java precisa de mais memory
  
  env:
    SPRING_PROFILES_ACTIVE: "production"
    JAVA_TOOL_OPTIONS: "-XX:+UseContainerSupport -XX:MaxRAMPercentage=75.0"
```

### Exemplo 3: React Frontend

```yaml
# frontend-infra/values-prod.yaml

k8s-helm-templates:
  image:
    repository: ghcr.io/rg-hermann/frontend
    tag: "v1.5.0"
    pullPolicy: IfNotPresent
  
  containerPort: 80
  replicaCount: 2
  
  probes:
    enabled: false  # Frontend não tem health checks
  
  resources:
    limits:
      cpu: 200m       # Frontend é leve
      memory: 256Mi
```

---

## 🚀 Benefícios dessa Arquitetura

| Benefício | Antes | Depois |
|-----------|-------|--------|
| **Reusabilidade** | Cada app copia templates | Uma única chart compartilhada |
| **Manutenção** | N templates pra manter | 1 chart para manter |
| **Consistência** | Configs diferentes | Padrão aplicado a todas |
| **Escalabilidade** | Difícil adicionar nova app | Copiar values, pronto |
| **Updates** | Todos repos precisam update | Atualizar uma vez em k8s-helm-templates |
| **Documentação** | Dispersa | Centralizada |

---

## 📚 Como Começar

1. **Entender o Chart**: Leia [k8s-helm-templates/README.md](../k8s-helm-templates/README.md)
2. **Ver Exemplos**: Analise [examples/](../k8s-helm-templates/examples/)
3. **Para Python**: Use `examples/values-python-fastapi.yaml`
4. **Para Java**: Use `examples/values-spring-boot.yaml`
5. **Para Outra Stack**: Adapte o exemplo mais próximo

---

## ❓ FAQ

**P: Posso manter templates customizadas em cada app-infra?**  
R: Não recomendado, mas tecnicamente sim. Melhor manter agnóstico e resolver tudo via values.

**P: E se minha app precisa de algo específico?**  
R: Abra uma issue/PR no k8s-helm-templates para adicionar suporte genérico.

**P: Como fago para usar versão específica do chart?**  
R: No `Chart.yaml` dependency, especifique `version: "0.1.0"`

**P: Posso ter múltiplas apps no mesmo helm chart?**  
R: Sim, copie values-dev.yaml em `values-app1.yaml`, `values-app2.yaml`, etc

---

**Última atualização**: Maio 23, 2026  
**Status**: ✅ Arquitetura Final Recomendada
