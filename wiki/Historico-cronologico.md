# Histórico cronológico

> **Rótulo:** Referência
> **TL;DR:** Eventos relevantes na evolução do projeto Mecânica Hermes, em ordem cronológica.
> **Última revisão:** 2026-05-18

## Linha do tempo

```mermaid
timeline
  title Evolução da Mecânica Hermes
  Fase 1 (Domain) : Modelagem DDD da OS<br/>Clean Architecture + CQRS
  Fase 2 (Containers) : Dockerfile multi-stage<br/>Publicação no Docker Hub
  Fase 2 (K8s) : Kustomize com overlays hml/prd<br/>HPA por CPU
  Fase 3 (Microservices) : Quebra em 3 serviços<br/>Mensageria via RabbitMQ/MassTransit<br/>Webhook HMAC<br/>Outbox transacional
  Fase 4 (Quality) : Cobertura ≥ 80% + SonarCloud<br/>Suíte E2E Robot BDD<br/>OpenTelemetry → New Relic<br/>Allure em GitHub Pages
  Fase 5 (Cloud) : Terraform 5 módulos<br/>EKS + RDS + Cognito + API Gateway<br/>Lambda CognitoToken<br/>SDK v1.0 em GitHub Packages<br/>Idempotência + DLQ observability
  Fase 6 (Docs) : Esta Wiki :)
```

## Eventos relevantes

| Data aprox. | Evento |
|---|---|
| Início | Repositório mono-API com .NET 8 + Postgres |
| Mid | Migração para .NET 10 |
| Mid | Adiciona MassTransit + SAGA |
| Mid | Quebra em 3 serviços (OS, Cadastros, Pagamentos) |
| Mid | Migração de Cadastros para Postgres 18 |
| Mid | Migração de Pagamentos para Mongo + replica set |
| Mid | Adiciona Outbox transacional |
| Mid | Adiciona idempotência cross-service nos 4 consumers de integração |
| Mid | Adiciona Lambda CognitoToken |
| Late | Cria repositório do SDK + adoação em Cadastros e Pagamentos |
| Late | E2E Robot Framework com Allure |
| Late | Cria repositório `mecanica-hermes-docs` para esta Wiki |
| 2026-05-18 | Plano editorial + esqueleto da Wiki |

_TODO: substituir "Mid/Late" por datas reais consultando o git log de cada repo._

## Eventos breaking (mensageria)

Nenhum até hoje. Todos os 7 eventos estão em `.v1`.

## Veja também

- [Tech Challenge — FIAP 13SOAT](Tech-Challenge-FIAP-13SOAT)
- [Índice de ADRs](Indice-de-ADRs)
