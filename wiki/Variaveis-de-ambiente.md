# Variáveis de ambiente

> **Rótulo:** Referência
> **TL;DR:** Consolidado de env vars dos 4 serviços .NET + Lambda. Para o conjunto completo, ver `docs/configuration.md` de cada repo.
> **Última revisão:** 2026-05-18

## Common (3 APIs)

| Variável | Quem usa | Exemplo |
|---|---|---|
| `ASPNETCORE_ENVIRONMENT` | todos | `Development`, `Testing`, `Production` |
| `OTEL_EXPORTER_OTLP_ENDPOINT` | todos | `https://otlp.nr-data.net` |
| `OTEL_EXPORTER_OTLP_HEADERS` | todos | `api-key=<NR_LICENSE>` |
| `OTEL_SERVICE_NAME` | todos | `mecanica-hermes-api-ordem-servico` |

## API Ordem de Serviço

| Variável | Descrição |
|---|---|
| `ConnectionStrings__DefaultConnection` | Postgres |
| `MassTransit__RabbitMq__Host` | `amqp://...:5672` |
| `Mongo__ConnectionString` | Mongo para SAGA state |
| `AUTH__AUTHORITY` | URL do Cognito |
| `AUTH__ISSUER` | URL do Cognito issuer |

## API Cadastros

| Variável | Descrição |
|---|---|
| `ConnectionStrings__DefaultConnection` | Postgres |
| `AUTH__AUTHORITY`, `AUTH__ISSUER` | Cognito |
| `EmailSender__Enabled` | `true`/`false` |
| `EmailSender__SmtpServer`, `*__SmtpPort`, `*__UserName`, `*__Password` | SMTP |
| `WEBHOOK__SIGNING_SECRET` | HMAC do webhook de orçamento (32+ bytes) |

## API Pagamentos

| Variável | Descrição |
|---|---|
| `Mongo__ConnectionString` | `mongodb://...:27017/?replicaSet=rs0` |
| `MassTransit__RabbitMq__Host` | Rabbit |
| `AUTH__AUTHORITY`, `AUTH__ISSUER` | Cognito |
| `MERCADO_PAGO__BASE_URL` | URL base do MP (sandbox ou prod) |
| `MERCADO_PAGO__ACCESS_TOKEN` | Token do MP |
| `MERCADO_PAGO__WEBHOOK_SECRET` | HMAC do webhook MP |
| `M2M__TOKEN_ENDPOINT` | `https://<cognito-domain>/oauth2/token` |
| `M2M__CLIENT_ID`, `M2M__CLIENT_SECRET`, `M2M__SCOPE` | Client Credentials para Cadastros |
| `CADASTROS__BASE_URL` | `http://api-cadastros:8080` |

## Lambda CognitoToken

| Variável | Descrição |
|---|---|
| `RDS__Host`, `RDS__Port`, `RDS__Database`, `RDS__Username` | Conexão |
| `COGNITO_SECRET_ID` | `mechermes-<env>-cognito-client-secret` |
| `EmailSender__*` | Mesmas vars do Cadastros |

## SDK

| Variável | Descrição |
|---|---|
| `GITHUB_USERNAME` | Handle do GitHub |
| `GITHUB_TOKEN` | PAT classic com `read:packages` |

## Onde aprofundar

- [`api-ordem-servico/docs/configuration.md`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/configuration.md)
- [`api-cadastros/docs/configuration.md`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/configuration.md)
- [`api-pagamentos/docs/configuration.md`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/configuration.md)
- [Lambda README](https://github.com/fiap-challenge-13soat/mecanica-hermes-lambda/blob/main/README.md)

## Veja também

- [Subir um único serviço](Subir-um-unico-servico)
- [SDK — Como consumir](SDK-Como-consumir)
