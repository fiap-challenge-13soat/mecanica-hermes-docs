# Repo: `mecanica-hermes-lambda`

> **Rótulo:** Referência
> **Papel em uma frase:** AWS Lambda que autentica o cliente por CPF, gera um JWT via Cognito Client Credentials e envia por e-mail.
> **Stack:** .NET 10, AWS Lambda, Npgsql, MailKit, AWS Secrets Manager SDK
> **Categoria:** Infraestrutura (componente do API Gateway)
> **Última revisão:** 2026-05-19

## Resumo

A única função Lambda do projeto, chamada `CognitoTokenLambda`. É a porta de entrada do **cliente final** — ele não tem credenciais OAuth próprias, então:

1. Cliente envia CPF para o endpoint público (`POST /oauth2/token` do API Gateway).
2. API Gateway invoca a Lambda.
3. Lambda consulta o cliente no **RDS PostgreSQL** de Cadastros.
4. Se encontrar, faz Client Credentials no Cognito e obtém JWT com scope `mechermes/client`.
5. Envia o JWT por e-mail via SMTP.

## Como se relaciona com o resto

- **Depende de:** RDS de Cadastros (consulta de cliente), Cognito User Pool, Secrets Manager, SMTP.
- **É invocada por:** API Gateway HTTP — rota `/oauth2/token` (a permissão Lambda↔API Gateway é gerenciada pelo workflow `Deploy - Lambda Function`).

## Pontos-chave

- **Clean Architecture adaptada para Lambda** — Ports & Adapters em um único projeto.
- **Npgsql puro** (sem EF Core) para minimizar cold start.
- **HttpClient e SDK Cognito estáticos** (reutilizados entre invocações quentes).
- **Secret consolidado** — client_id, client_secret, domain, scope + senha do RDS no mesmo Secret.
- **CPF mascarado** em logs (`***3061`).
- **Configuração automática de permissões do API Gateway** — o deploy adiciona/atualiza a `lambda:InvokeFunction` policy para o API Gateway invocar a Lambda.

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-lambda/blob/main/README.md) | Arquitetura, fluxo, deploy |
| [docs/Arquitetura.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-lambda/blob/main/docs/Arquitetura.md) | Detalhes |
| [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-lambda/blob/main/docs/ADRs.md) | Decisões (uso de Npgsql, estáticos, etc.) |

## Comandos

```bash
# Build local
dotnet build CognitoTokenLambda/

# Deploy manual (requer dotnet-lambda tools)
dotnet lambda deploy-function mechermes-prd-cognito-token-lambda \
  --function-role arn:aws:iam::<account>:role/LabRole
```

Via GitHub Actions: workflows `Deploy - Lambda Function` e `Destroy - Lambda Function` (input `environment`).

## Veja também

- [Lambda CognitoToken](Lambda-CognitoToken) — fluxo detalhado e env vars
- [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT)
- [API Gateway + VPC Link](API-Gateway-VPC-Link)
