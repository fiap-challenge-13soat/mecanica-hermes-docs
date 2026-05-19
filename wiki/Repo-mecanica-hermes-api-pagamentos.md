# Repo: `mecanica-hermes-api-pagamentos`

> **Rótulo:** Referência
> **Papel em uma frase:** Integração Mercado Pago — gera link de pagamento, recebe webhook HMAC, faz polling de status como backup, publica callbacks de aprovação/recusa.
> **Stack:** .NET 10, ASP.NET Core, MongoDB 7 (replica set), RabbitMQ, MassTransit 8 + DelayedMessageScheduler
> **Categoria:** Domínio
> **Última revisão:** 2026-05-19

## Resumo

Agregado **`Pagamento`** com state machine própria (`PendenteGeracao → LinkGerado → AguardandoConfirmacao → {Pago | Recusado | Expirado}`). Reconciliação **dupla**:

1. **Webhook HMAC** do Mercado Pago.
2. **Polling agendado** via `IMessageScheduler` (a cada 2 min) até confirmar ou atingir `LimiteVerificacoes`.

Assim, mesmo se o webhook do MP falhar, eventualmente confirmamos via polling.

A imagem é publicada no Docker Hub (`mechermes/mecanica-hermes-api-pagamentos:latest`) a cada merge em `main` pelo workflow `Build and Publish Docker Image`. O `docker-compose` da stack inteira do ecossistema mora no repo [`mecanica-hermes-tests-e2e`](Repo-mecanica-hermes-tests-e2e).

## Como se relaciona com o resto

- **Consome** `ordem-de-servico.aguardando-pagamento.v1` (de OS) para iniciar o fluxo.
- **Consome** `ordem-de-servico.cancelada.v1` (de OS) para recusar/cancelar pagamentos pendentes vinculados à OS.
- **Publica** `link-pagamento-gerado.v1`, `pagamento.confirmado.v1`, `pagamento.recusado.v1`.
- **Chama M2M** `GET /api/clientes/{id}` em Cadastros para obter dados do cliente ao criar preferência MP.
- **Integra HTTP** com Mercado Pago (criação de preferência + consulta de status).

## Pontos-chave

- **SAGA própria** em MongoDB (coleção `saga_pagamento`).
- **Outbox transacional em Mongo** via `IClientSessionHandle` (requer replica set).
- **Dedup de webhook MP** por `mp_event_id`.
- **Polly** para resilience das chamadas HTTP ao MP.
- **M2M handler** com cache de token + serialização de refresh via `SemaphoreSlim`.
- **WireMock** mocka o MP em testes locais e E2E.
- **SDK compartilhado v1.1.0** consumido via GitHub Packages.
- **Consumer cross-service `OrdemDeServicoCanceladaConsumer`** processa cancelamentos vindos da OS (auto-discovery via reflection do assembly de Application).
- **Contract test camelCase** validando serialização JSON dos eventos no contrato `.v1`.

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/README.md) | Visão geral, docker-compose |
| [CLAUDE.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/CLAUDE.md) | Arquitetura interna |
| [docs/payment-flow.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/payment-flow.md) | State machine + webhook + polling |
| [docs/messaging.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/messaging.md) | SAGA + contratos |
| [docs/database.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/database.md) | Coleções Mongo |
| [docs/runbook.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/runbook.md) | Incidentes (MP fora, webhook desync) |
| [docs/adrs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/adrs.md) | ADRs |

## Comandos essenciais

| Comando | O que faz |
|---|---|
| `just build` | Build Release |
| `just run` | Roda API local (requer Mongo replica set + Rabbit) |
| `just test` | Tudo (requer Docker) |
| `just test-unit` | Só unitários |

## Veja também

- [Runbook — Pagamentos](Runbook-Pagamentos)
- [Fluxo — Caminho feliz](Fluxo-Caminho-feliz)
- [Fluxo — Pagamento expirado](Fluxo-Pagamento-expirado)
- [Fluxo — Cancelamento em aguardando pagamento](Fluxo-Cancelamento-em-aguardando-pagamento)
