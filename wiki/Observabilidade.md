# Observabilidade — Visão geral

> **Rótulo:** Explicação
> **TL;DR:** OpenTelemetry exporta traces, métricas e logs via OTLP/HTTP para o **New Relic**. Serilog estrutura logs. `correlation_id` cruza todos os serviços.
> **Última revisão:** 2026-05-18

## Pilares

| Pilar | Ferramenta | Como |
|---|---|---|
| **Logs** | Serilog → stdout → nri-bundle (collector) → New Relic | JSON estruturado com `correlation_id`, `service.name`, `trace_id` |
| **Traces** | `ActivitySource.MecanicaHermes.Api` + libs auto-instrumentadas → OTLP → New Relic | Propagação via header `traceparent` |
| **Métricas** | `Meter.MecanicaHermes.*` (manual) + auto-instrumentation → OTLP → New Relic | Counters, histograms, gauges |

## Endpoints OTLP

Configurados via env vars no pod:

```
OTEL_EXPORTER_OTLP_ENDPOINT=https://otlp.nr-data.net
OTEL_EXPORTER_OTLP_HEADERS=api-key=<NEW_RELIC_LICENSE_KEY>
OTEL_SERVICE_NAME=mecanica-hermes-api-ordem-servico
OTEL_RESOURCE_ATTRIBUTES=service.namespace=mechermes,deployment.environment=prd
```

Kubernetes Secret `newrelic-license-secret` injeta a license key.

## Correlation ID

De ponta a ponta, todo request HTTP / mensagem Rabbit carrega um **`correlation_id`** (UUID v4):

- Gerado por `CorrelationIdMiddleware` (do SDK) na entrada HTTP, se ausente.
- Propagado para Rabbit via cabeçalho `MT-Activity-Id` do MassTransit + propriedade custom `correlation_id`.
- Re-injetado em logs via `LogContext.PushProperty`.

Result: você acha um request no Scalar, copia o `correlation_id` do header de resposta, busca em qualquer log/trace.

## Service Map no New Relic

A propagação de trace gera automáticamente um **service map**: você vê visualmente que `api-ordem-servico` chama `api-cadastros`, e que ambos publicam para `rabbitmq`.

## Páginas seguintes

- [Logs](Logs) — esquema, campos, níveis.
- [Métricas de negócio](Metricas-de-negocio).
- [Métricas de mensageria](Metricas-de-mensageria).
- [Traces distribuídos](Traces-distribuidos).
- [Dashboards e alertas](Dashboards-e-alertas).

## Em DEV

Em `Development` não exportamos pra New Relic. Logs ficam no console; traces só in-process. Para depurar localmente com OTLP, suba um Jaeger:

```bash
docker run -d --name jaeger -p 16686:16686 -p 4318:4318 \
  jaegertracing/all-in-one:latest
export OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4318
```

## Veja também

- [DLQ observability](DLQ-observability)
- [SDK — Visão dos 6 pacotes](SDK-Visao-dos-6-pacotes) (`Shared.Observability`)
