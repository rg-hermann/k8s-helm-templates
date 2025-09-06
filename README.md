# Helm Charts Templates

Este diretório contém templates Helm Charts para diferentes tipos de aplicações (Java, Node.js, Python, etc). Use os charts em `charts/` para renderizar deployments Kubernetes em qualquer ambiente (local, AKS, EKS, GKE, on-premises).

## Estrutura
- `charts/java-app/` - Chart para aplicações Java
- `charts/node-app/` - Chart para aplicações Node.js
- `charts/python-app/` - Chart para aplicações Python

## Como usar
1. Edite o arquivo `values.yaml` do chart desejado com os parâmetros da sua aplicação (imagem, porta, etc).
2. Renderize o template com Helm:
   ```sh
   helm install <nome-release> ./charts/<app-type> -f ./charts/<app-type>/values.yaml
   ```
3. Adicione novos charts conforme necessário para outros tipos de aplicações.

## Personalização
Adapte os templates conforme as necessidades do seu ambiente (storage, ingress, variáveis, etc).

---

Dúvidas ou sugestões? Edite este README ou adicione novos charts!
