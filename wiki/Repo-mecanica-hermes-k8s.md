# Repo: `mecanica-hermes-k8s`

> **Rótulo:** Referência
> **Papel em uma frase:** Deploy declarativo das 3 APIs no EKS via Kustomize, com overlays `hml` e `prd` e uma pasta por microsserviço.
> **Stack:** Kustomize, Helm, kubectl, GitHub Actions
> **Categoria:** Infraestrutura
> **Última revisão:** 2026-05-19

## Resumo

Repositório que **publica** as 3 APIs no cluster EKS provisionado pelo `infra`. Usa Kustomize com **uma pasta por microsserviço** em `src/` — cada microsserviço tem seu próprio `base/` (manifests neutros) + `overlays/{hml,prd}/` (configurações por ambiente). Instala também o **New Relic** via Helm para observabilidade.

Estrutura:

```text
src/
├── api-ordem-servico/{base,overlays/{hml,prd}}
├── api-cadastros/{base,overlays/{hml,prd}}
├── api-pagamentos/{base,overlays/{hml,prd}}
└── global-configs/newrelic/   (helm-values.yaml + namespace.yaml)
```

## Como se relaciona com o resto

- **Depende de:** `mecanica-hermes-infra` (EKS + RDS + Cognito + Secrets Manager provisionados).
- **Consome:** imagens Docker `mechermes/*` publicadas pelos repos de serviço.
- **Provisiona:** namespace por ambiente, Secret `app-secret`, ConfigMap por API, Deployment + HPA + Service (LoadBalancer) **por API**.

## Pontos-chave

- **1 Deployment por microsserviço** (`mecanica-hermes-api-ordem-servico`, `-cadastros`, `-pagamentos`) — refatorado para isolar configuração e ciclo de release por API.
- 1 namespace por ambiente: `mecanica-hermes-hml`, `mecanica-hermes-prd`.
- Secret `app-secret` consolida JWT key, conn strings (Postgres + Mongo), credenciais do Mercado Pago, e-mail, New Relic license e secrets de webhook.
- Cada Deployment usa `envFrom: configMapRef + secretRef` + variáveis OTLP do New Relic injetadas dinamicamente.
- Probes (`startup`/`liveness`/`readiness`) apontam para `/healthz/live` e `/healthz/ready`.
- HPA em CPU (mínimo 2, máximo 10 pods típico).
- Service tipo LoadBalancer cria NLB automático; URLs HTTPS expostas pelo Scalar usam o CIDR da VPC como `KnownProxies`.
- `Force Cleanup - Stuck Namespace` ajuda em casos de namespace travado em `Terminating`.
- New Relic kubelet desabilitado por erro persistente de TLS (componente removido do `helm-values.yaml`).

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/blob/main/README.md) | Deploy local + via Actions |
| [docs/Arquitetura.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/blob/main/docs/Arquitetura.md) | Arquitetura de deploy |
| [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/blob/main/docs/ADRs.md) | Decisões de deploy |
| [src/global-configs/newrelic/](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/tree/main/src/global-configs/newrelic) | `helm-values.yaml` do `nri-bundle` |

## Workflows GitHub Actions

| Workflow | O que faz |
|---|---|
| `API - Deploy` | Aplica overlay do(s) microsserviço(s) no cluster (inputs: `environment` e `image_tag`) |
| `API - Destroy` | Deleta recursos do ambiente |
| `New Relic - Deploy/Destroy` | Helm install/uninstall do nri-bundle |
| `Force Cleanup - Stuck Namespace` | Remove finalizers travados |

## Veja também

- [Cluster Kubernetes](Cluster-Kubernetes)
- [Forçar limpeza de namespace](Forcar-limpeza-de-namespace)
- [Observabilidade](Observabilidade) (New Relic)
