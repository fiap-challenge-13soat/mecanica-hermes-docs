# Logs

> **Rótulo:** Referência
> **TL;DR:** Serilog produz JSON estruturado para stdout. Coletado pelo nri-bundle e enviado ao New Relic.
> **Última revisão:** 2026-05-18

## Esquema padrão

Cada linha de log é um JSON com pelo menos:

```json
{
  "@t": "2026-05-18T14:30:00.123Z",
  "@l": "Information",
  "@mt": "OS {OrdemId} avancou para {NovoStatus}",
  "@m": "OS abc... avancou para EmExecucao",
  "OrdemId": "abc...",
  "NovoStatus": "EmExecucao",
  "correlation_id": "7e2c...",
  "trace_id": "4a8...",
  "span_id": "f1c...",
  "service.name": "mecanica-hermes-api-ordem-servico",
  "deployment.environment": "prd"
}
```

- `@t` — timestamp UTC.
- `@l` — nível.
- `@mt` — message template (estruturado).
- `@m` — mensagem renderizada (legível).

## Níveis

| Nível | Quando |
|---|---|
| `Debug` | Detalhes verbosos. Desligado em prod. |
| `Information` | Eventos de negócio (OS criada, avançada, cancelada) |
| `Warning` | Algo inesperado mas tolerado (retry, idempotência skip) |
| `Error` | Falha de operação (timeout external, validação falhou) |
| `Fatal` | Erro irrecuperável (não conseguimos subir o pod) |

Nível mínimo em prod: `Information`.

## Enriquecimento

O middleware `CorrelationIdMiddleware` (do SDK) faz `LogContext.PushProperty("correlation_id", ...)`. Todo log do mesmo request herda.

Método `WithSpan()` do OpenTelemetry adiciona `trace_id` e `span_id` automaticamente.

## O que **não** logamos

- **Tokens JWT** (nem do cliente, nem o M2M).
- **Payload completo de webhook** (pode ter dados pessoais).
- **Senhas, secrets, API keys** — nem em mensagens de exception.
- **CPF completo** — mascaramos para `***3061` em logs da Lambda.

## NRQL útil

```sql
-- Erros recentes em um serviço
SELECT message FROM Log
WHERE service.name = 'mecanica-hermes-api-ordem-servico'
  AND level = 'Error'
SINCE 1 hour ago

-- Trace de uma OS específica
SELECT message FROM Log
WHERE OrdemId = 'abc...'
ORDER BY timestamp ASC
```

## Veja também

- [Observabilidade](Observabilidade)
- [Traces distribuídos](Traces-distribuidos)
