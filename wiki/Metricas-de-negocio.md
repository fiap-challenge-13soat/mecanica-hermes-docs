# Métricas de negócio

> **Rótulo:** Referência
> **TL;DR:** Métricas que respondem perguntas do produto, não do sistema. Volume de OSs, distribuição por estado, eventos ignorados.
> **Última revisão:** 2026-05-18

## Meters

| Meter | Serviço |
|---|---|
| `MecanicaHermes.Business` | OS |
| `MecanicaHermes.Cadastros.Business` | Cadastros |
| `MecanicaHermes.Pagamento.Business` | Pagamentos |

## Counters publicados

### OS

| Métrica | Tipo | Tags |
|---|---|---|
| `ordemdeservico.criada` | counter | — |
| `ordemdeservico.transicao` | counter | `status_origem`, `status_destino` |
| `ordemdeservico.evento_integracao_ignorado` | counter | `evento`, `status_atual` |
| `ordemdeservico.cancelada` | counter | `status_origem` |

### Cadastros

| Métrica | Tipo | Tags |
|---|---|---|
| `cadastros.cliente.criado` | counter | — |
| `cadastros.cliente.deletado_soft` | counter | — |
| `cadastros.webhook_orcamento.recebido` | counter | `decisao` |
| `cadastros.webhook_orcamento.duplicado` | counter | — |

### Pagamentos

| Métrica | Tipo | Tags |
|---|---|---|
| `pagamento.link_gerado` | counter | — |
| `pagamento.confirmado` | counter | `metodo` (webhook/polling) |
| `pagamento.recusado` | counter | `motivo` (rejected/expired/cancelled) |
| `pagamento.polling.executado` | counter | — |

## Exemplos NRQL

```sql
-- OS criadas por hora
SELECT rate(sum(ordemdeservico.criada), 1 hour)
FROM Metric SINCE 1 day ago

-- Top 5 transições da OS
SELECT sum(ordemdeservico.transicao) FROM Metric
FACET status_origem, status_destino
SINCE 1 day ago LIMIT 5

-- Taxa de aprovação de orçamento
SELECT
  filter(sum(cadastros.webhook_orcamento.recebido), WHERE decisao = 'aprovado') /
  sum(cadastros.webhook_orcamento.recebido) * 100
  AS 'Taxa aprovação %'
FROM Metric SINCE 7 days ago

-- Pagamentos confirmados por método
SELECT sum(pagamento.confirmado) FROM Metric
FACET metodo SINCE 1 day ago
```

## Veja também

- [Métricas de mensageria](Metricas-de-mensageria)
- [Dashboards e alertas](Dashboards-e-alertas)
