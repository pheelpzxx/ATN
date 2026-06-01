# pedidos-veloz-rolling-otel

**Aluno:** PHELLIPE GABRIEL DOS SANTOS ARAUJO  
**RA:** 141880  
**Turma:** GTINA3

## Objetivo

MVP DevOps da plataforma **Pedidos Veloz**, simulando uma arquitetura de pedidos em microsserviços para a Loja Veloz.

## Arquitetura

- API Gateway HTTP
- Serviço de Pedidos
- Serviço de Pagamentos
- Serviço de Estoque
- PostgreSQL
- Observabilidade com Jaeger/OpenTelemetry em ambiente local
- Kubernetes para produção mínima
- Terraform como esqueleto de infraestrutura como código

## Como executar localmente

```bash
docker compose up --build
```

Teste principal:

```bash
curl -X POST http://localhost:8080/gateway/order   -H "Content-Type: application/json"   -d '{"sku":"SKU-001","quantity":2}'
```

Jaeger local:

```txt
http://localhost:16686
```

## Kubernetes

```bash
kubectl apply -f k8s/00-namespace.yaml
kubectl apply -f k8s/configmap.yaml
kubectl apply -f k8s/secret.example.yaml
kubectl apply -f k8s/deployments/postgres.yaml
kubectl apply -f k8s/deployments/
kubectl apply -f k8s/services/
kubectl apply -f k8s/hpa/
```

## Estratégia de deploy

Rolling Update com maxSurge=1 e maxUnavailable=0 para reduzir indisponibilidade em deploys comuns.

## Escalabilidade

Uso de HPA para `api-gateway` e `pedidos-service`, pois são os serviços mais expostos ao aumento de tráfego durante campanhas promocionais.

## Observabilidade

Prometheus/Grafana conceitual + Jaeger instrumentado via OpenTelemetry. Os logs devem sair em stdout/stderr para coleta pelo ambiente Kubernetes. Os traces permitem acompanhar uma requisição do gateway até pedidos, estoque e pagamentos.

## CI/CD

O pipeline em `.github/workflows/ci-cd.yml` executa validação Python, build das imagens, publicação no GHCR e validação de manifests Kubernetes.

## Terraform

A pasta `terraform/` contém o esqueleto IaC. Em ambiente real, ela seria expandida para provisionar cluster gerenciado, node pools, registry, rede e políticas de acesso.

## Vídeo

Link do vídeo no YouTube: **INSERIR LINK AQUI**

## Fonte pública de referência

Google Cloud Online Boutique / microservices-demo como referência pública de e-commerce em microsserviços em Kubernetes.


## Validação rápida antes da entrega

```bash
python -m compileall api-gateway pedidos-service pagamentos-service estoque-service
python -m pytest api-gateway/tests pedidos-service/tests pagamentos-service/tests estoque-service/tests -q
```

## Observabilidade e endpoints

Cada serviço possui:

- `/health` para liveness probe;
- `/ready` para readiness probe;
- `/metrics` para métrica simples no padrão de texto compatível com raspagem pelo Prometheus.

## Secrets necessários no GitHub Actions

- `GITHUB_TOKEN`: usado automaticamente pelo GitHub para publicar no GHCR;
- `KUBE_CONFIG`: necessário em uma etapa futura de deploy real no cluster;
- credenciais externas, como token do provedor de pagamento, devem ficar fora do repositório.
