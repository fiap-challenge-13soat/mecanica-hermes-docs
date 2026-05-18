# Bancos de dados

> **Rótulo:** Referência
> **TL;DR:** PostgreSQL gerenciado (RDS) para OS e Cadastros; MongoDB para Pagamentos e estado de SAGA.
> **Última revisão:** 2026-05-18

## Visão geral

| Banco | Usado por | Onde |
|---|---|---|
| PostgreSQL | OS (`MecHermesOrdemDeServicoDB`), Cadastros (`mecanica_hermes_cadastros`) | RDS em produção; container `postgres:16/18` localmente |
| MongoDB | OS (saga state), Pagamentos (agregado + outbox + saga) | container `mongo:7` (em produção, Atlas ou instância gerenciada com replica set) |

## PostgreSQL gerenciado (RDS)

Ver detalhes em [PostgreSQL](PostgreSQL).

- Engine: 17.6.
- Instância: `db.t4g.micro`.
- Backup: 7 dias.
- Storage: gp3 criptografado.
- Sem public access.

## MongoDB

Ver detalhes em [MongoDB](MongoDB).

- Replica set obrigatório (transações multi-documento).
- Em local: single-node replica set funciona.
- Em produção: recomenda-se Atlas M10+ ou self-hosted 3-node.

## Migrations (Postgres)

Aplicadas no **startup** dos serviços OS e Cadastros via `MigrateDatabaseAsync()`. Primeiro pod aplica; demais esperam.

```bash
dotnet ef migrations add <Name> \
  --project src/Mecanica.Hermes.<svc>.Infrastructure \
  --startup-project src/Mecanica.Hermes.<svc>.Api
```

## Schema do Mongo

Não usa migrations. Mudanças via:

1. **Código defensivo** — leitor aceita formato antigo e novo.
2. **Script de backfill** ad-hoc quando necessário.

## Senhas

- **Postgres** — gerenciada pelo RDS Master Secret; injetada no pod via Kubernetes Secret.
- **Mongo** — sem autenticação em local; em produção, Atlas gerencia.

## Veja também

- [PostgreSQL](PostgreSQL)
- [MongoDB](MongoDB)
- [Outbox transacional](Outbox-transacional)
