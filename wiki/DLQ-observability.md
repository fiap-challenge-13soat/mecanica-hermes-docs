# DLQ observability

> **Rótulo:** Explicação
> **TL;DR:** Observador global do MassTransit emite métrica `messaging.consume_faults` quando uma mensagem cai na DLQ definitivamente. É o sinal mais importante de problema de mensageria.
> **Última revisão:** 2026-05-18

## O que é DLQ

Dead Letter Queue — fila onde mensagens vão **depois** de esgotar todos os retries e redeliveries (3 imediatos + 3 atrasados). Não ficam mais sendo retentadas automaticamente.

Uma mensagem em DLQ significa que o sistema **não consegue processar aquele evento** — pode ser bug determinístico, schema drift, dependência externa fora do ar além da janela de retry.

## Como detectamos

Os 3 serviços .NET registram um `IConsumeObserver` global em `Infrastructure/MassTransit/Observability/MessagingFaultObserver.cs`:

```csharp
public Task ConsumeFault<T>(ConsumeContext<T> context, Exception exception)
    where T : class
{
    _metrics.ConsumeFault(
        messageType: typeof(T).Name,
        exceptionType: exception.GetType().Name);
    return Task.CompletedTask;
}
```

Esse hook é chamado **após** os retries esgotarem.

## Métrica

| Métrica | Meter | Tags |
|---|---|---|
| `messaging.consume_faults` (counter) | `MecanicaHermes.Messaging` (OS), `MecanicaHermes.Cadastros.Messaging`, `MecanicaHermes.Pagamento.Messaging` | `message_type`, `exception_type` |

Exportada via OpenTelemetry OTLP → New Relic.

## NRQL recomendado

```sql
SELECT sum(messaging.consume_faults)
FROM Metric
FACET service.name, message_type, exception_type
SINCE 1 hour ago
```

## Alertas sugeridos

| Condição | Severidade |
|---|---|
| `messaging.consume_faults > 0` em qualquer serviço por 5 min | warning |
| `messaging.consume_faults > 10` por 5 min | critical |
| `messaging.consume_faults` com `exception_type=ConcurrencyConflictException` | info (esperado em concorrência) |

## Ações após o alerta

1. Identificar `message_type` e `exception_type`.
2. Buscar logs por `correlation_id` da mensagem (Serilog enriquecido).
3. Inspecionar mensagem na DLQ no RabbitMQ Management UI (`<exchange>_error`).
4. Decidir: corrigir bug + replay, ou descartar (em último caso).

Ver [Resposta a incidentes](Resposta-a-incidentes) para o passo a passo.

## Diferença de Outbox dead-lettered

- **`messaging.consume_faults`** — o **consumer** falhou em processar a mensagem.
- **`outbox.messages.dead_lettered`** — o **publisher** falhou em despachar a mensagem do banco para o broker.

Ambos são sinais de problema; cada um aponta para um lado diferente do pipeline.

## Veja também

- [SAGA com MassTransit](SAGA-com-MassTransit)
- [Outbox transacional](Outbox-transacional)
- [Métricas de mensageria](Metricas-de-mensageria)
- [Resposta a incidentes](Resposta-a-incidentes)
