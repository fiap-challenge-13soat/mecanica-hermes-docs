# Provisionamento (Terraform)

> **Rótulo:** Referência
> **TL;DR:** 5 módulos Terraform em `mecanica-hermes-infra`, executados em sequência.
> **Última revisão:** 2026-05-18

## Módulos

| Módulo | O que provisiona | Workflow |
|---|---|---|
| `aws/` | VPC, EKS, Node Groups, SGs, S3, DynamoDB | `AWS - Terraform Create` |
| `learner-lab/` | Mesmo do `aws/` mas usa `LabRole` (AWS Academy) | `Learner Lab AWS - Terraform Create` |
| `cognito/` | User Pool, App Client, Resource Server, Domain, Secrets Manager | `Cognito - Terraform Create` |
| `db/` | RDS Postgres + DB Subnet Group | `Database - Terraform Create` |
| `api-gateway/` | API Gateway HTTP, Cognito Authorizer, VPC Link, Lambda Integration | `API Gateway - Terraform Create` |

## Estado remoto

- **S3 bucket:** `mechermes-tf-state-aws`
- **DynamoDB table:** `mechermes-tf-locks`
- Cada módulo grava em uma key distinta (ex.: `cognito/terraform.tfstate`, `db/<env>/terraform.tfstate`).

## Variáveis e secrets do CI

No repositório `mecanica-hermes-infra`:

| Tipo | Nome |
|---|---|
| Secret | `AWS_ACCESS_KEY_ID` |
| Secret | `AWS_SECRET_ACCESS_KEY` |
| Secret | `AWS_SESSION_TOKEN` |
| Secret | `AWS_ACCOUNT_ID` (só `learner-lab/`) |
| Variable | `AWS_DEFAULT_REGION` (ex.: `us-east-1`) |

## Ordem importa

```
1. aws/ (ou learner-lab/)  → cria VPC + EKS
2. cognito/                → cria User Pool
3. db/                     → cria RDS (reusa VPC do #1)
4. mecanica-hermes-k8s     → deploy das APIs
5. mecanica-hermes-lambda  → deploy da Lambda
6. api-gateway/            → cria API Gateway (reusa Cognito + EKS + Lambda)
```

Destruír: ordem inversa.

## Quick start local

```bash
aws configure
aws s3api create-bucket --bucket mechermes-tf-state-aws --region us-east-1
aws dynamodb create-table --table-name mechermes-tf-locks ...
cd aws/
terraform init && terraform apply
cd ../cognito/
terraform init && terraform apply -var environment=hml
# ...
```

Ver Quick Starts específicos em cada módulo do `infra/`.

## Veja também

- [Deploy em AWS](Deploy-em-AWS)
- [Diagrama AWS completo](Diagrama-AWS-completo)
- [Repo: mecanica-hermes-infra](Repo-mecanica-hermes-infra)
