# Helm Charts Templates

Repositório central de Helm Charts para Kubernetes. Este projeto fornece um chart genérico (`app-generic`) para deploy de aplicações containerizadas, permitindo configuração flexível via `values.yaml` em cada aplicação.

## Estrutura

- `charts/app-generic/` - Chart genérico para qualquer stack (Java, Node.js, Python, etc)

## Features suportadas

- Deploy de aplicações
- Configuração de volumes persistentes (PVC)
- Uso de Secrets e ConfigMaps
- Configuração de Ingress
- Recursos customizados

## Como usar

1. No repositório da sua aplicação, crie um `values.yaml` com os parâmetros desejados (imagem, porta, volumes, secrets, etc).

2. Renderize o template com Helm:

    ```sh
    helm install <nome-release> <repo-k8s>/charts/app-generic -f <seu-repo>/values.yaml
    ```

3. Exemplos de configuração:

### Volume Persistente

```yaml
persistence:
   enabled: true
   accessMode: ReadWriteOnce
   size: 5Gi
```

### Secret

```yaml
secret:
   enabled: true
   data:
      USERNAME: "admin"
      PASSWORD: "s3cr3t"
```

### ConfigMap

```yaml
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
