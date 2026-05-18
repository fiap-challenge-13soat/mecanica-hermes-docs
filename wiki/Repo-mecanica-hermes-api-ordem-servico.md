# Repo: `mecanica-hermes-api-ordem-servico`

> **Rótulo:** Referência
> **Papel em uma frase:** Orquestrador principal — state machine da Ordem de Serviço, com SAGA MassTransit e Outbox transacional.
> **Stack:** .NET 10, ASP.NET Core, PostgreSQL 16, RabbitMQ, MongoDB (saga), MassTransit 8, EF Core 10
> **Categoria:** Domínio
> **Última revisão:** 2026-05-18

## Resumo

Serviço **central** do ecossistema. Expõe um único agregado de domínio — `OrdemDeServico` — com state machine de 8 estados + 2 terminais. Todos os comandos de mutação são **assíncronos** (`202 Accepted`) e passam pela SAGA do MassTransit.

## Como se relaciona com o resto

- **Consome eventos** de Cadastros (`orcamento-aprovado/rejeitado-pelo-cliente.v1`) e de Pagamentos (`link-pagamento-gerado.v1`, `pagamento.confirmado/recusado.v1`).
- **Publica eventos** para Cadastros (`ordem-de-servico.aguardando-aprovacao.v1`) e Pagamentos (`ordem-de-servico.aguardando-pagamento.v1`).
- **Chama HTTP M2M** em Cadastros para `GET /api/clientes/{id}`.

Ver [Catálogo de eventos](Catalogo-de-eventos).

## Pontos-chave

- **State Pattern** — cada status é uma classe (ver [State Pattern](State-Pattern)).
- **SAGA principal do ecossistema** — 1 operação em voo por `OrdemDeServicoId`.
- **Outbox transacional** em Postgres com `SELECT ... FOR UPDATE SKIP LOCKED` para multi-instância.
- **Idempotência cross-service** em 4 consumers de integração.
- **DLQ observability** via `MessagingFaultObserver`.
- **Endpoints retornam 202** — cliente pola para ver resultado.

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/README.md) | Visão geral, endpoints |
| [CLAUDE.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/CLAUDE.md) | Arquitetura interna detalhada |
| [docs/architecture.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/architecture.md) | Camadas e padrões |
| [docs/service-order-flow.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/service-order-flow.md) | State machine completa |
| [docs/messaging.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/messaging.md) | SAGA, Outbox, contratos |
| [docs/database.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/database.md) | ER, índices, migrations |
| [docs/runbook.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/runbook.md) | Incidentes operacionais |
| [docs/adrs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/adrs.md) | ADRs |

## Comandos essenciais

| Comando | O que faz |
|---|---|
| `just build` | Build Release |
| `just run` | Roda API local (requer Postgres + Rabbit + Mongo) |
| `just test` | Roda toda a suíte de testes |
| `just test-unit` | Só unitários (sem Docker) |
| `just test-coverage` | Com cobertura |

## Endpoints (resumo)

Base: `/api/ordens-de-servico`. Detalhes em [API pública](API-publica).

## Veja também

- [Runbook — Ordem de Serviço](Runbook-Ordem-de-Servico)
- [Componentes por serviço](Componentes-por-servico)
- [Domínio de negócio](Dominio-de-negocio)
