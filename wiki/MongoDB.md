# MongoDB

> **Rótulo:** Referência
> **TL;DR:** MongoDB 7 em **replica set** é usado para o agregado e Outbox de Pagamentos, e para o estado da SAGA da OS.
> **Última revisão:** 2026-05-18

## Por que replica set

MongoDB **só suporta transações multi-documento em replica sets**. Como precisamos atomicidade entre agregado + Outbox (ver [Outbox transacional](Outbox-transacional)), o replica set é obrigatório.

Em desenvolvimento, single-node replica set (`--replSet rs0`) cumpre o requisito sem custo.
Em produção, recomenda-se Atlas ou instância gerenciada com 3 nós.

## Coleções

### Pagamentos

| Coleção | Conteúdo | Indices principais |
|---|---|---|
| `pagamentos` | Documento do agregado `Pagamento` (com histórico embarcado) | `_id` (PagamentoId), `ordem_de_servico_id` (1) |
| `pagamento_outbox` | Eventos pendentes de despacho | `status` (1), `created_at` (1), TTL em `processed_at` (24h após sucesso) |
| `saga_pagamento` | State da SAGA | `correlation_id` (PagamentoId), `current_state` |
| `mp_webhook_events` | Dedup de webhooks do Mercado Pago | `mp_event_id` (1 unique) |

### OS (SAGA state)

| Coleção | Conteúdo |
|---|---|
| `saga_ordem_de_servico` | State da SAGA da OS (não contém o agregado — esse fica em Postgres) |

## Esquema do documento `pagamentos`

```json
{
  "_id": "a1b2c3d4-...",
  "ordem_de_servico_id": "e5f6g7h8-...",
  "cliente_id": "i9j0k1l2-...",
  "valor": 1250.00,
  "status": {
    "nome": "AguardandoConfirmacao",
    "alterado_em": "2026-05-18T14:30:00Z"
  },
  "link_pagamento": {
    "mp_preference_id": "123-abc",
    "init_point": "https://www.mercadopago.com.br/checkout/v1/...",
    "gerado_em": "2026-05-18T14:10:00Z"
  },
  "historico": [
    { "status": "PendenteGeracao", "alterado_em": "2026-05-18T14:00:00Z" },
    { "status": "LinkGerado", "alterado_em": "2026-05-18T14:10:00Z" },
    { "status": "AguardandoConfirmacao", "alterado_em": "2026-05-18T14:30:00Z" }
  ],
  "version": 3,
  "limite_verificacoes": 30
}
```

Número de verificações de polling é limitado para evitar loço infinito caso o MP nunca responda.

## Document/Domain split

Classes mapeadas via `MongoDB.Driver` (`PagamentoDocument`) ficam em `Infrastructure/Persistence/Entities/`. O agregado de domínio (`Pagamento`) **não tem nenhum atributo de persistência**. `PagamentoMapper` converte nos dois sentidos.

## Sem migrations

MongoDB não tem migrations no sentido relacional. Mudanças de schema são feitas por:

1. **Código defensivo** — leitor aceita formato antigo e novo.
2. **Backfill com script** quando necessário.
3. **Versionamento de schema** em campo `schema_version` (não usamos hoje; reservado para o futuro).

## Índices

Criados via `IndexKeysDefinitionBuilder` no startup do serviço (`OnAfterMappingApplied`). Idempotente — se o índice já existe, não falha.

## Veja também

- [Bancos de dados](Bancos-de-dados)
- [Outbox transacional](Outbox-transacional)
- [SAGA com MassTransit](SAGA-com-MassTransit)
