# Dashboards e alertas

> **Rótulo:** Referência
> **TL;DR:** Lista das queries NRQL recomendadas para dashboards e dos alertas que deveriam estar configurados.
> **Última revisão:** 2026-05-18

## Dashboards sugeridos

### Painel "Visão do produto"

| Painel | Query |
|---|---|
| OSs por hora | `SELECT rate(sum(ordemdeservico.criada), 1 hour) FROM Metric SINCE 1 day ago` |
| Taxa de aprovação de orçamento | `SELECT filter(sum(cadastros.webhook_orcamento.recebido), WHERE decisao='aprovado') / sum(cadastros.webhook_orcamento.recebido) * 100 FROM Metric SINCE 7 days ago` |
| Pagamentos confirmados | `SELECT sum(pagamento.confirmado) FROM Metric FACET metodo SINCE 1 day ago` |
| OSs canceladas | `SELECT sum(ordemdeservico.cancelada) FROM Metric FACET status_origem SINCE 1 day ago` |

### Painel "Saúde da plataforma"

| Painel | Query |
|---|---|
| Taxa de erro HTTP por serviço | `SELECT percentage(count(*), WHERE response.status >= 500) FROM Span FACET service.name SINCE 30 minutes ago` |
| Latência p99 HTTP | `SELECT percentile(duration, 99) FROM Span WHERE span.kind='server' FACET service.name SINCE 30 minutes ago` |
| DLQ ativa | `SELECT sum(messaging.consume_faults) FROM Metric FACET service.name, message_type SINCE 1 hour ago` |
| Outbox dead-lettered | `SELECT sum(outbox.messages.dead_lettered) FROM Metric FACET service.name SINCE 6 hours ago` |
| Pods não-prontos | (métrica kube-state-metrics via nri-bundle) |

### Painel "Mercado Pago"

| Painel | Query |
|---|---|
| Latency POST /preferences | `SELECT percentile(duration, 95) FROM Span WHERE name LIKE 'POST checkout/preferences%' SINCE 1 hour ago` |
| Status codes do MP | `SELECT count(*) FROM Span WHERE name LIKE '%mercadopago%' FACET http.response.status_code SINCE 1 hour ago` |
| Webhooks duplicados | `SELECT sum(pagamento.webhook.duplicado) FROM Metric SINCE 1 day ago` |

## Alertas sugeridos

| Alerta | Condição | Severidade |
|---|---|---|
| Serviço offline | sem dados de `service.name=X` por 2 min | critical |
| Taxa de 5xx alta | > 1% de erro por 5 min | critical |
| Latência p99 alta | > 2s sustained por 5 min | warning |
| DLQ ativa | `messaging.consume_faults > 0` por 5 min | warning |
| DLQ pesada | `messaging.consume_faults > 10` por 5 min | critical |
| Outbox parado | `outbox.processor.batch_size` estável não-zero por 15 min | critical |
| OS travada | OS em estado não-terminal por > 24h | warning |
| MP indisponível | `pagamento.polling.executado` cai a zero | critical |

## Status atual

_TODO: dashboards e alertas ainda não estão materializados no New Relic. Esta página serve como blueprint._

## Veja também

- [Métricas de negócio](Metricas-de-negocio)
- [Métricas de mensageria](Metricas-de-mensageria)
- [Runbook macro](Runbook-macro)
