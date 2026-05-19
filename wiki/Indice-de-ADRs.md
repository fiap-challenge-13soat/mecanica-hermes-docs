# Índice de ADRs

> **Rótulo:** Referência
> **TL;DR:** Agregação cronológica dos Architecture Decision Records de todos os 9 repositórios.
> **Última revisão:** 2026-05-18

## Por repositório

Links vão direto para os arquivos `docs/adrs.md` (ou `docs/ADRs.md`):

| Repositório | Link |
|---|---|
| `mecanica-hermes-infra` | [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra/blob/main/docs/ADRs.md) |
| `mecanica-hermes-k8s` | [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s/blob/main/docs/ADRs.md) |
| `mecanica-hermes-lambda` | [docs/ADRs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-lambda/blob/main/docs/ADRs.md) |
| `api-ordem-servico` | [docs/adrs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/adrs.md) |
| `api-cadastros` | [docs/adrs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/adrs.md) |
| `api-pagamentos` | [docs/adrs.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/adrs.md) |
| `api-sdk` | _(usa `docs/architecture.md` + `docs/versioning.md`)_ |

## Destaques (cross-repo)

ADRs mais consequenciais para entender o sistema como um todo:

| ADR | Repo | Tópico |
|---|---|---|
| Clean Architecture + DDD + CQRS | api-ordem-servico | Estrutura de camadas adotada por todos os .NET |
| Outbox transacional | api-ordem-servico | Garantia at-least-once dos eventos |
| Soft delete em Cadastros | api-cadastros (ADR-001) | Modelagem divergente — ver [Glossario](Glossario) |
| `BaseDomain` paralelo em Cadastros | api-cadastros | Razao para Tier 2 do SDK |
| Outbox transacional para webhook | api-cadastros (ADR-007) | Garantir entrega cross-service do webhook de orçamento |
| Confirmação dupla (webhook + polling) | api-pagamentos | Resiliência ao webhook do MP |
| Mongo replica set obrigatório | api-pagamentos | Necessário para Outbox transacional |
| Versão única dos 6 pacotes | api-sdk | `Directory.Build.props` |
| Tier 2 do `Shared.MassTransit` | api-sdk | Por que está skeleton em v1.0 |
| Npgsql sem EF Core na Lambda | lambda | Reduzir cold start |

## Histórico cronológico

Ver [Histórico cronológico](Historico-cronologico) para a linha do tempo agregada.

## Como adicionar um ADR

1. Criar entrada no `docs/adrs.md` do repo correspondente.
2. Numerar sequencialmente (ADR-001, ADR-002, ...).
3. Formato sugerido (Michael Nygard):
   - **Status** (proposed / accepted / superseded)
   - **Context**
   - **Decision**
   - **Consequences**
4. Linkar nesta página (Wiki) no PR.

## Veja também

- [Índice de RFCs](Indice-de-RFCs)
- [Histórico cronológico](Historico-cronologico)
