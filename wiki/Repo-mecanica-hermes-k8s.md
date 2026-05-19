# Repo: `mecanica-hermes-k8s`

> **Rótulo:** Referência
> **Papel em uma frase:** Deploy declarativo das 3 APIs no EKS via Kustomize, com overlays `hml` e `prd`.
> **Stack:** Kustomize, Helm, kubectl, GitHub Actions
> **Categoria:** Infraestrutura
> **Última revisão:** 2026-05-18

## Resumo

Repositório que **publica** as 3 APIs no cluster EKS provisionado pelo `infra`. Usa Kustomize com `base/` (manifests neutros) + `overlays/{hml,prd}/` (configurações por ambiente). Instala também o **New Relic** via Helm para observabilidade.

## Como se relaciona com o resto

- **Depende de:** `mecanica-hermes-infra` (EKS + RDS provisionados).
- **Consome:** imagens Docker `mechermes/*` publicadas pelos repos de serviço.
- **Provisiona:** namespace, Secret, ConfigMap, Deployments, HPAs, Services LoadBalancer.

## Pontos-chave

- 1 namespace por ambiente: `mecanica-hermes-hml`, `mecanica-hermes-prd`.
- Secret `app-secret` consolida JWT key, conn string Postgres, e-mail, New Relic license.
- HPA em CPU (mínimo 2, máximo 10 pods típico).
- Service tipo LoadBalancer cria NLB automático.
- `Force Cleanup - Stuck Namespace` ajuda em casos de namespace travado em `Terminating`.

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/blob/main/README.md) | Deploy local + via Actions |
| [docs/Arquitetura.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/blob/main/docs/Arquitetura.md) | Arquitetura de deploy |
| [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/blob/main/docs/ADRs.md) | Decisões de deploy |

## Workflows GitHub Actions

| Workflow | O que faz |
|---|---|
| `API - Deploy` | Aplica overlay no cluster |
| `API - Destroy` | Deleta recursos do ambiente |
| `New Relic - Deploy/Destroy` | Helm install/uninstall do nri-bundle |
| `Force Cleanup - Stuck Namespace` | Remove finalizers travados |

## Veja também

- [Cluster Kubernetes](Cluster-Kubernetes)
- [Forçar limpeza de namespace](Forcar-limpeza-de-namespace)
- [Observabilidade](Observabilidade) (New Relic)
