# Para operadores

> **Rótulo:** Tutorial (jornada guiada)
> **TL;DR:** Roteiro para quem opera o sistema em produção — o que monitorar e como triagear incidentes.
> **Última revisão:** 2026-05-18

## Em 30 minutos você terá conseguido

- Saber o que está monitorado e onde.
- Triagear um incidente cross-service.
- Saber onde buscar runbook específico por serviço.

## Roteiro

### 1. Mapa do que está rodando (5 min)

- [Diagrama AWS completo](Diagrama-AWS-completo) — todos os recursos provisionados em AWS.
- [Cluster Kubernetes](Cluster-Kubernetes) — namespaces, deployments, HPA, services.

### 2. Observabilidade (10 min)

- [Observabilidade](Observabilidade) — OTLP → New Relic, Serilog, correlation id.
- [Métricas de negócio](Metricas-de-negocio) — eventos ignorados, transições de estado.
- [Métricas de mensageria](Metricas-de-mensageria) — DLQ, retries, outbox.
- [Dashboards e alertas](Dashboards-e-alertas) — queries NRQL recomendadas.

### 3. Triagem de incidentes (10 min)

- [Runbook macro](Runbook-macro) — fluxograma de triagem cross-service.
- [Resposta a incidentes](Resposta-a-incidentes) — replay de DLQ, drenagem, reforçar outbox.

### 4. Runbooks específicos (5 min de leitura inicial)

- [Runbook — Ordem de Serviço](Runbook-Ordem-de-Servico)
- [Runbook — Cadastros](Runbook-Cadastros)
- [Runbook — Pagamentos](Runbook-Pagamentos)

## Sinais de alerta prioritários

| Sinal | Onde | Ação |
|---|---|---|
| `messaging.consume_faults` aumentando | New Relic | Ver [Resposta a incidentes](Resposta-a-incidentes) → DLQ |
| `outbox.messages.dead_lettered` > 0 | New Relic | Investigar evento específico no banco |
| OS travada em estado não-terminal por > 1h | NRQL custom (ver [Dashboards](Dashboards-e-alertas)) | Verificar SAGA + DLQ |
| Webhook MP retornando 5xx | Logs do `api-pagamentos` | Ver [Runbook — Pagamentos](Runbook-Pagamentos) |
| Lambda CognitoToken com `dotnet` cold start > 5s | New Relic | Pode ser falta de memória; ver [Lambda CognitoToken](Lambda-CognitoToken) |

## Para casos críticos

- [Forçar limpeza de namespace](Forcar-limpeza-de-namespace) — workflow `Force Cleanup - Stuck Namespace` quando o cluster fica preso.
