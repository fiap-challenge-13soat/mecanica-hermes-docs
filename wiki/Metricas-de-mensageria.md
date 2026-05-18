# Métricas de mensageria

> **Rótulo:** Referência
> **TL;DR:** Métricas de saúde do pipeline assíncrono: DLQ, retries, outbox.
> **Última revisão:** 2026-05-18

## Meters

| Meter | Serviço |
|---|---|
| `MecanicaHermes.Messaging` | OS |
| `MecanicaHermes.Cadastros.Messaging` | Cadastros |
| `MecanicaHermes.Pagamento.Messaging` | Pagamentos |
| `MecanicaHermes.<svc>.Outbox` | Outbox de cada serviço |

## Counters principais

### Mensageria

| Métrica | Tipo | Tags | Quando |
|---|---|---|---|
| `messaging.consume_faults` | counter | `message_type`, `exception_type` | DLQ definitiva (após retries) |
| `messaging.consume_attempts` | counter | `message_type` | Cada attempt (incluindo retries) |
| `messaging.consume_duration` | histogram | `message_type` | Tempo de processamento |

### Outbox

| Métrica | Tipo | Quando |
|---|---|---|
| `outbox.messages.dispatched` | counter | Evento despachado com sucesso |
| `outbox.messages.failed` | counter | Tentativa falhou (vai retry) |
| `outbox.messages.dead_lettered` | counter | Falha definitiva |
| `outbox.processor.batch_size` | histogram | Tamanho do batch processado |

## NRQL para alarmes

```sql
-- DLQ ativa
SELECT sum(messaging.consume_faults) FROM Metric
WHERE service.name LIKE 'mecanica-hermes%'
FACET service.name, message_type
SINCE 10 minutes ago
HAVING sum(messaging.consume_faults) > 0

-- Outbox empacando
SELECT max(outbox.processor.batch_size) FROM Metric
FACET service.name
SINCE 30 minutes ago

-- Taxa de falha por consumer
SELECT
  filter(sum(messaging.consume_faults), WHERE message_type = 'AdicionarProdutoCommand') /
  filter(sum(messaging.consume_attempts), WHERE message_type = 'AdicionarProdutoCommand')
  AS 'Failure rate'
FROM Metric SINCE 1 hour ago
```

## Alertas sugeridos

| Condição | Severidade |
|---|---|
| `messaging.consume_faults > 0` por 5 min | warning |
| `messaging.consume_faults > 10` por 5 min | critical |
| `outbox.messages.dead_lettered > 0` | critical |
| `outbox.processor.batch_size` cresce monotonicamente | warning (processor pode estar parado) |

## Veja também

- [DLQ observability](DLQ-observability)
- [Outbox transacional](Outbox-transacional)
- [Resposta a incidentes](Resposta-a-incidentes)
