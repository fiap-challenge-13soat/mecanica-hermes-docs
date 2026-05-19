# Cluster Kubernetes

> **Rótulo:** Referência
> **TL;DR:** EKS 1.34 com 3 APIs publicadas via Kustomize, overlays `hml` e `prd`. New Relic via Helm.
> **Última revisão:** 2026-05-18

## Estrutura do repositório `mecanica-hermes-k8s`

```text
k8s/
  base/
    api-os/         # Deployment, Service, HPA, ConfigMap
    api-cadastros/
    api-pagamentos/
    newrelic/        # values do Helm
  overlays/
    hml/
      namespace.yaml
      kustomization.yaml  # patches para hml
    prd/
      namespace.yaml
      kustomization.yaml
```

## Recursos por ambiente

| Recurso | Quantidade | Notas |
|---|---|---|
| Namespace | 1 (`mecanica-hermes-<env>`) | |
| Secret `app-secret` | 1 | ConnectionString, JWT key, e-mail, New Relic license |
| Secret `newrelic-license-secret` | 1 | License key duplicada (compat com nri-bundle) |
| ConfigMap `mecanica-hermes-api-config` | 1 | URLs internas, feature flags |
| Deployment | 3 (1 por API) | replicas inicial=2 |
| HPA | 3 | CPU target 70%, min 2, max 10 |
| Service LoadBalancer | 3 | gera NLB externo na AWS |

## Deploy via Actions

Workflow `API - Deploy` (input: `environment`, `image_tag`). Faz:

1. `aws eks update-kubeconfig`.
2. `kubectl apply -f overlays/<env>/namespace.yaml`.
3. Cria/atualiza Secrets via `kubectl create secret ... --dry-run -o yaml | kubectl apply -f -`.
4. `kustomize edit set image` para a tag desejada.
5. `kubectl apply -k overlays/<env>`.
6. `kubectl rollout status` aguardando ready.

## Deploy manual

```bash
export ENVIRONMENT=hml  # ou prd
aws eks update-kubeconfig --name mechermes-eks
kubectl apply -f k8s/overlays/$ENVIRONMENT/namespace.yaml
kubectl apply -k k8s/overlays/$ENVIRONMENT
kubectl get pods -n mecanica-hermes-$ENVIRONMENT -w
```

## Acesso externo

O Service tipo LoadBalancer cria um NLB. Extrair endpoint:

```bash
kubectl get svc -n mecanica-hermes-$ENVIRONMENT
```

Em produção, o tráfego entra pelo **API Gateway** (Cognito Authorizer + VPC Link), não direto pelo NLB.

## New Relic

Instalado via Helm:

```bash
helm repo add newrelic https://helm-charts.newrelic.com
helm upgrade --install newrelic-bundle newrelic/nri-bundle \
  --namespace newrelic --create-namespace \
  --values k8s/base/newrelic/helm-values.yaml \
  --set global.licenseKey=$NEW_RELIC_LICENSE_KEY
```

Workflow correspondente: `New Relic - Deploy`.

## Troubleshooting

- Namespace travado em `Terminating` → [Forçar limpeza de namespace](Forcar-limpeza-de-namespace).
- Pods em `CrashLoopBackOff` → `kubectl logs ...` + verificar Secret `app-secret`.
- HPA com `<unknown>` no CPU target → metrics-server faltando (vem com EKS, normalmente).

## Veja também

- [Repo: mecanica-hermes-k8s](Repo-mecanica-hermes-k8s)
- [Observabilidade](Observabilidade) (New Relic)
- [Forçar limpeza de namespace](Forcar-limpeza-de-namespace)
