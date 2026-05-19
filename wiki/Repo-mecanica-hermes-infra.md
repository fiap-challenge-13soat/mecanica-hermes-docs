# Repo: `mecanica-hermes-infra`

> **Rótulo:** Referência
> **Papel em uma frase:** Provisiona toda a infraestrutura AWS via Terraform — VPC, EKS, RDS por microsserviço, Cognito, Amazon MQ (RabbitMQ) e API Gateway.
> **Stack:** Terraform 1.14, AWS, GitHub Actions
> **Categoria:** Infraestrutura
> **Última revisão:** 2026-05-19

## Resumo

Repositório mais a montante do ecossistema. Após a refatoração recente, os módulos Terraform foram divididos em **dois grupos**:

- **`shared/`** — recursos compartilhados pelo ecossistema:
  - `shared/aws/` — VPC, EKS (Kubernetes 1.34), Security Groups + cluster Amazon MQ (RabbitMQ gerenciado, orquestrando o MassTransit).
  - `shared/cognito/` — User Pool + App Client + Resource Server + Domain + Secrets Manager.
  - `shared/api-gateway/` — API Gateway HTTP + Cognito Authorizer + VPC Link + integrações Scalar (rota `/scalar` por microsserviço).
- **`services/`** — recursos por microsserviço (hoje, uma RDS PostgreSQL por API):
  - `services/api-cadastros/`
  - `services/api-ordem-servico/`
  - `services/api-pagamentos/`
- `learner-lab/` — variante para AWS Academy (`LabRole`).

O estado vive em S3 (`mechermes-tf-state-aws` para o shared, `mechermes-tf-state-aws-microsservices` para os DBs por serviço) com lock em DynamoDB (`mechermes-tf-locks`).

## Como se relaciona com o resto

- **Saída:** `mecanica-hermes-k8s`, `mecanica-hermes-lambda` e os 3 serviços .NET dependem dos recursos criados aqui (EKS, RDS por serviço, Cognito, Amazon MQ, API Gateway).
- **Entrada:** depende apenas de credenciais AWS (workflow ou local).

## Pontos-chave

- **Amazon MQ (RabbitMQ gerenciado)** orquestra o MassTransit em produção — substitui o RabbitMQ self-hosted do Docker Compose.
- **Uma RDS por microsserviço** — cada API tem seu próprio módulo Terraform em `services/<api>/` e seu próprio backend state.
- **EKS Kubernetes 1.34** + Node Group `t3.small`, regras de ingress para o NLB do Service da aplicação.
- **RDS PostgreSQL 17.6** (`db.t4g.micro`), criptografada em repouso, sem acesso público, backup retido 7 dias.
- **Cognito** Resource Server com scopes `mechermes/admin`, `mechermes/client` e `servico-scope` (M2M).
- **API Gateway HTTP** com Cognito Authorizer + VPC Link, expõe endpoints e UI Scalar `/scalar` para cada microsserviço via HTTPS.
- **Lambda permissions** do API Gateway gerenciadas via CI (remove + reapply em cada deploy para evitar drift).

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/README.md) | Visão geral + ordem de execução |
| [shared/aws/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/shared/aws/README.md) | Módulo VPC + EKS + Amazon MQ |
| [shared/cognito/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/shared/cognito/README.md) | Módulo Cognito |
| [shared/api-gateway/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/shared/api-gateway/README.md) | Módulo API Gateway + Scalar |
| [services/api-cadastros/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/services/api-cadastros/README.md) | RDS de Cadastros |
| [services/api-ordem-servico/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/services/api-ordem-servico/README.md) | RDS de Ordem de Serviço |
| [services/api-pagamentos/README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/services/api-pagamentos/README.md) | RDS de Pagamentos |
| [docs/Arquitetura.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/docs/Arquitetura.md) | Arquitetura detalhada |
| [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/docs/ADRs.md) | Decisões de IaC |

## Workflows GitHub Actions

| Workflow | O que faz |
|---|---|
| `Learner Lab AWS - Terraform Create/Destroy` | Módulo `learner-lab/` |
| `AWS - Terraform Create/Destroy` | Módulo `shared/aws/` (VPC + EKS + Amazon MQ) |
| `Cognito - Terraform Create/Destroy` | Módulo `shared/cognito/` (input `environment`) |
| `Database - Terraform Create/Destroy` | Módulos `services/<api>/` (inputs `service` + `environment`) |
| `API Gateway - Terraform Create/Destroy` | Módulo `shared/api-gateway/` |
| `terraform-format-validate` | Lint Terraform em PRs |

## Veja também

- [Provisionamento (Terraform)](Provisionamento-Terraform)
- [Diagrama AWS completo](Diagrama-AWS-completo)
- [Deploy em AWS](Deploy-em-AWS)
- [RabbitMQ](RabbitMQ)
