# Helm Charts Templates

Repositório central de Helm Charts para Kubernetes. Armazena templates reutilizáveis para deploy de aplicações em diferentes stacks (Java, Node.js, Python, etc). Permite padronizar, versionar e facilitar o uso de charts em múltiplos projetos, seja em ambientes locais ou em nuvem.

## Estrutura

- `charts/java-app/` - Chart para aplicações Java
- `charts/node-app/` - Chart para aplicações Node.js
- `charts/python-app/` - Chart para aplicações Python

## Features suportadas

- Deploy de aplicações
- Configuração de volumes persistentes (PVC)
- Uso de Secrets e ConfigMaps
- Configuração de Ingress

## Como usar

1. Edite o arquivo `values.yaml` do chart desejado com os parâmetros da sua aplicação (imagem, porta, volumes, secrets, etc).

2. Renderize o template com Helm:

    ```sh
    helm install <nome-release> ./charts/<app-type> -f ./charts/<app-type>/values.yaml
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

Adapte os templates conforme as necessidades do seu ambiente (storage, ingress, variáveis, etc). Os charts são flexíveis e podem ser estendidos para outros stacks.

---

Dúvidas ou sugestões? Edite este README ou adicione novos charts!
