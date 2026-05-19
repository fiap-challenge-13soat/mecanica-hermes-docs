# Resposta a incidentes

> **Rótulo:** How-to
> **TL;DR:** Como replay de DLQ, drenar fila `.v1` e forçar reprocessamento do Outbox.
> **Última revisão:** 2026-05-18

## DLQ — mensagem na fila `_error`

### Inspecionar

```bash
kubectl port-forward -n mecanica-hermes-prd svc/rabbitmq 15672
# Abra http://localhost:15672 (guest/guest)
# Queues → filtre por `_error`
# Clique na fila → Get Message(s) (sem ack para não remover)
```

Copie o payload. Veja:

- `MT-Fault-Exception-Type` no header.
- `MT-Fault-Message`.
- `MT-Original-MessageId`.

### Replay (manual)

```bash
# Pelo RabbitMQ Management UI:
# Queue de erro → Get message → ack=true
# Em seguida, publish na queue original com o mesmo payload e headers
```

Alternativa via `rabbitmqadmin` CLI ou script Python.

### Quando NUNCA fazer replay

- Se o erro for **determinístico** (bug no consumer) sem fix deployado: replay vai cair de novo.
- Se a mensagem for de schema antigo descontinuado.
- Se você não **leu** o payload e validou que ele é legítimo.

## Outbox `DeadLettered`

### Inspecionar (Postgres — OS, Cadastros)

```sql
SELECT id, entity_name, status, retry_count, error_message, created_at
FROM outbox_messages
WHERE status = 'DeadLettered'
ORDER BY created_at DESC LIMIT 50;
```

### Inspecionar (Mongo — Pagamentos)

```js
db.pagamento_outbox.find({ status: "DeadLettered" })
  .sort({ created_at: -1 }).limit(50);
```

### Reprocessar (Postgres)

Depois de **corrigir a causa raiz**:

```sql
UPDATE outbox_messages
SET status = 'Pending', retry_count = 0, error_message = NULL
WHERE id IN ('id1', 'id2', ...);
```

O `OutboxProcessor` (background service) pega no próximo poll (2s).

### Reprocessar (Mongo)

```js
db.pagamento_outbox.updateMany(
  { _id: { $in: [...] } },
  { $set: { status: "Pending", retry_count: 0, error_message: null } }
);
```

## Drenar fila `.v1` após migração para `.v2`

1. Confirmar que **todos** os consumers da `.v2` estão ativos.
2. **Producer para de publicar `.v1`** (PR + deploy).
3. Aguardar a fila `.v1` zerar.
4. Excluir a queue `.v1` no RabbitMQ Management (cuidado — irreversível).
5. Marcar `.v1` como `[Obsolete]` no SDK.

## SAGA travada

Não **não** delete o documento de saga. Em vez disso:

1. Inspecionar:

```js
db.saga_ordem_de_servico.findOne({ _id: "<OrdemId>" });
```

2. Identificar em qual estado a saga está.
3. Aguardar sentinel timeout (alguns minutos).
4. Se persistir, abrir issue com dump do documento.

## Veja também

- [DLQ observability](DLQ-observability)
- [Outbox transacional](Outbox-transacional)
- [Runbook macro](Runbook-macro)
