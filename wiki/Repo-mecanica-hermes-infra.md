# Repo: `mecanica-hermes-infra`

> **Rótulo:** Referência
> **Papel em uma frase:** Provisiona toda a infraestrutura AWS via Terraform — VPC, EKS, RDS, Cognito, API Gateway.
> **Stack:** Terraform, AWS, GitHub Actions
> **Categoria:** Infraestrutura
> **Última revisão:** 2026-05-18

## Resumo

Repositório mais a montante do ecossistema. Contém **5 módulos Terraform** independentes que constroem a infraestrutura em sequência:

1. `aws/` (ou `learner-lab/`) — VPC + EKS + Security Groups + S3/DynamoDB para tfstate.
2. `cognito/` — User Pool + App Client + Resource Server + Domain + Secrets Manager.
3. `db/` — RDS PostgreSQL (reutiliza VPC do passo 1).
4. `api-gateway/` — API Gateway HTTP + Cognito Authorizer + VPC Link.

O estado vive em S3 (`mechermes-tf-state-aws`) com lock em DynamoDB (`mechermes-tf-locks`).

## Como se relaciona com o resto

- **Saída:** todos os repositórios de serviços e o `k8s` dependem dos recursos criados aqui.
- **Entrada:** depende apenas de credenciais AWS (workflow ou local).

## Pontos-chave

- 2 módulos `aws/`: padrão (admin) e `learner-lab/` (AWS Academy com role `LabRole`).
- EKS Kubernetes 1.34 + Node Group `t3.small`.
- RDS criptografado em repouso, sem public access.
- Cognito Resource Server com scopes customizados.
- API Gateway HTTP com Cognito Authorizer + VPC Link.

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/README.md) | Visão geral + ordem de execução |
| [aws/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/aws/README.md) | Módulo VPC + EKS |
| [cognito/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/cognito/README.md) | Módulo Cognito |
| [db/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/db/README.md) | Módulo RDS |
| [api-gateway/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/api-gateway/README.md) | Módulo API Gateway |
| [docs/Arquitetura.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/docs/Arquitetura.md) | Arquitetura detalhada |
| [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/docs/ADRs.md) | Decisões de IaC |

## Workflows GitHub Actions

| Workflow | O que faz |
|---|---|
| `Learner Lab AWS - Terraform Create/Destroy` | Módulo `learner-lab/` |
| `AWS - Terraform Create/Destroy` | Módulo `aws/` |
| `Cognito - Terraform Create/Destroy` | Módulo `cognito/` (input `environment`) |
| `Database - Terraform Create/Destroy` | Módulo `db/` |
| `API Gateway - Terraform Create/Destroy` | Módulo `api-gateway/` |

## Veja também

- [Provisionamento (Terraform)](Provisionamento-Terraform)
- [Diagrama AWS completo](Diagrama-AWS-completo)
- [Deploy em AWS](Deploy-em-AWS)
