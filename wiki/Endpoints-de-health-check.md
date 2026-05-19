# Endpoints de health-check

> **Rótulo:** Referência
> **TL;DR:** Liveness e readiness em todos os serviços, em paths padronizados.
> **Última revisão:** 2026-05-18

## Por serviço

| Serviço | Liveness | Readiness |
|---|---|---|
| API Ordem de Serviço | `/healthz/live` | `/healthz/ready` |
| API Cadastros | `/healthz/live` | `/healthz/ready` |
| API Pagamentos | `/healthz/live` | `/healthz/ready` |
| Lambda CognitoToken | n/a (Lambda) | n/a |
| WireMock | n/a | `/__admin/health` |
| Robot E2E suite | n/a | `:8081/health`, `:8082/health`, `:8083/health` |

## Diferença entre live e ready

- **Liveness** — "o processo está vivo?". Só falha se o app travar (sem responder). Kubernetes mata o pod nesse caso.
- **Readiness** — "o processo está pronto para receber tráfego?". Valida dependências externas (Postgres, Mongo, Rabbit). Kubernetes remove do Service se falhar.

## Tags de readiness

O SDK fornece constantes em `HealthCheckTags`:

```csharp
builder.Services.AddHealthChecks()
    .AddNpgSql(connString, tags: [HealthCheckTags.Ready, HealthCheckTags.Database])
    .AddRabbitMQ(rabbitUri, tags: [HealthCheckTags.Ready, HealthCheckTags.Messaging]);
```

O endpoint `/healthz/ready` filtra por tag `Ready`. O endpoint `/healthz/live` não filtra.

## Em produção

O Kubernetes Deployment usa:

```yaml
livenessProbe:
  httpGet:
    path: /healthz/live
    port: 8080
  initialDelaySeconds: 30
  periodSeconds: 10

readinessProbe:
  httpGet:
    path: /healthz/ready
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 5
```

## Veja também

- [Cluster Kubernetes](Cluster-Kubernetes)
- [Observabilidade](Observabilidade)
