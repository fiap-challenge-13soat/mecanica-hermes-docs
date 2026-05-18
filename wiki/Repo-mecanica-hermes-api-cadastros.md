# Repo: `mecanica-hermes-api-cadastros`

> **Rótulo:** Referência
> **Papel em uma frase:** Cadastros de Clientes e Produtos + webhook HMAC de aprovação/rejeição de orçamento + endpoint M2M consumido por Pagamentos.
> **Stack:** .NET 10, ASP.NET Core, PostgreSQL 18, RabbitMQ, MailKit, EF Core 10
> **Categoria:** Domínio
> **Última revisão:** 2026-05-18

## Resumo

Serviço **reativo** (sem SAGA própria). Dois agregados raiz:

- **`Cliente`** (com veículos) — CPF, e-mail, endereço.
- **`Produto`** — catálogo de peças/serviços.

Guarda o **webhook link** de orçamento (token aleatório, expira em 7 dias) e implementa o handler do webhook com HMAC + dedup. Tem Outbox transacional próprio para os eventos do webhook.

## Como se relaciona com o resto

- **Consome eventos** de OS (`ordem-de-servico.aguardando-aprovacao.v1`) e Pagamentos (`link-pagamento-gerado.v1`) para disparar e-mails.
- **Publica eventos** para OS (`orcamento-aprovado/rejeitado-pelo-cliente.v1`).
- **Expõe** `GET /api/clientes/{id}` consumido por OS e Pagamentos via M2M.

## Pontos-chave

- **Soft delete** — `IsDeleted` em Cliente e Produto + índices únicos parciais.
- **Sem SAGA** — puramente CRUD + reativo a eventos.
- **Outbox transacional** para garantir entrega do evento `orcamento-aprovado/rejeitado` à OS.
- **BaseDomain paralelo ao SDK** — divergente para acomodar soft delete (Tier 2).
- **`AdminOrM2MScope`** habilita chamada M2M com `servico-scope`.

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/README.md) | Visão geral |
| [CLAUDE.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/CLAUDE.md) | Arquitetura interna |
| [docs/architecture.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/architecture.md) | Camadas |
| [docs/messaging.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/messaging.md) | Eventos + webhook flow |
| [docs/database.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/database.md) | Tabelas + soft delete |
| [docs/runbook.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/runbook.md) | Diagnóstico |
| [docs/adrs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/adrs.md) | ADRs (ADR-007: Outbox webhook) |

## Comandos essenciais

| Comando | O que faz |
|---|---|
| `just build` | Build Release |
| `just run` | Roda API local (requer Postgres) |
| `just test` | Tudo (requer Docker) |
| `just test-unit` | Só unitários |

## Veja também

- [Runbook — Cadastros](Runbook-Cadastros)
- [Webhooks assinados (HMAC)](Webhooks-assinados-HMAC)
- [M2M Client Credentials](M2M-Client-Credentials)
