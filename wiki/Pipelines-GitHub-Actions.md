# Pipelines GitHub Actions

> **Rótulo:** Referência
> **TL;DR:** Índice consolidado dos workflows em **todos os 9 repositórios**.
> **Última revisão:** 2026-05-18

## `mecanica-hermes-infra`

| Workflow | Trigger | O que faz |
|---|---|---|
| `AWS - Terraform Create` | `workflow_dispatch` | Provisiona VPC + EKS no `aws/` |
| `AWS - Terraform Destroy` | `workflow_dispatch` | Destrói `aws/` |
| `Learner Lab AWS - Terraform Create/Destroy` | `workflow_dispatch` | `learner-lab/` (AWS Academy) |
| `Cognito - Terraform Create/Destroy` | `workflow_dispatch` (input `environment`) | Módulo `cognito/` |
| `Database - Terraform Create/Destroy` | `workflow_dispatch` (input `environment`) | Módulo `db/` |
| `API Gateway - Terraform Create/Destroy` | `workflow_dispatch` | Módulo `api-gateway/` |

## `mecanica-hermes-k8s`

| Workflow | Trigger | O que faz |
|---|---|---|
| `API - Deploy` | `workflow_dispatch` (inputs `environment`, `image_tag`) | Aplica overlay no EKS |
| `API - Destroy` | `workflow_dispatch` | Deleta recursos do namespace |
| `New Relic - Deploy` | `workflow_dispatch` | Helm install nri-bundle |
| `New Relic - Destroy` | `workflow_dispatch` | Helm uninstall |
| `Force Cleanup - Stuck Namespace` | `workflow_dispatch` | Remove finalizers travados |

## `mecanica-hermes-lambda`

| Workflow | Trigger | O que faz |
|---|---|---|
| `Deploy - Lambda Function` | `workflow_dispatch` (input `environment`) | Descobre infra + cria secret + publica função |
| `Destroy - Lambda Function` | `workflow_dispatch` (confirmação `DESTROY`) | Deleta Lambda + secret |

## `mecanica-hermes-api-ordem-servico`

| Workflow | Trigger | O que faz |
|---|---|---|
| `Build and tests CI` | push, PR | restore + build + test + coverage |
| `SonarCloud` | push `main`, PR | Quality gate |
| `Format CI` | push `main`, PR | `dotnet format --verify-no-changes` |
| `Build and Publish Docker Image` | push `main` | Builda e publica `mechermes/mecanica-hermes-api-ordem-servico:latest` |
| `Trigger K8s Deploy` | push `main` | Aciona o deploy no repo `k8s` |

## `mecanica-hermes-api-cadastros`

Mesmos 5 workflows do `api-ordem-servico` (mesmos nomes).

## `mecanica-hermes-api-pagamentos`

Mesmos 5 workflows do `api-ordem-servico` (mesmos nomes).

## `mecanica-hermes-api-sdk`

| Workflow | Trigger | O que faz |
|---|---|---|
| `Build and tests CI` | push, PR | restore + build + test + pack |
| `Format CI` | push, PR | Lint |
| `SonarCloud` | push `main`, PR | Quality gate |
| `Publish to GitHub Packages` | tag `v*.*.*`, `workflow_dispatch` | Publica os 6 pacotes |

## `mecanica-hermes-tests-e2e`

| Workflow | Trigger | O que faz |
|---|---|---|
| `e2e-tests` | push `main`, push `feature/**`, PR, manual | Sobe Compose + roda 8 suítes + publica Allure em GH Pages (só em main) |

## `mecanica-hermes-docs`

_Sem workflows (ainda). Plano: lint Markdown + validação de links + sync com a Wiki nativa._

## Permissões

- Workflows que publicam em **GitHub Packages** declaram `permissions: packages: write`.
- Workflows que aplicam Kustomize precisam de Secrets AWS (key + token).
- O secret `GITHUB_TOKEN` injetado pelo Actions tem `packages: read` para fazer restore do SDK.

## Veja também

- [Deploy em AWS](Deploy-em-AWS)
- [Cluster Kubernetes](Cluster-Kubernetes)
- [SDK — Como consumir](SDK-Como-consumir) (configurar PAT)
