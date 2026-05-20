# RabbitMQ / Amazon MQ

> **Rótulo:** Referência
> **TL;DR:** RabbitMQ 4.x (com plugin `rabbitmq_delayed_message_exchange`) em dev local; **Amazon MQ for RabbitMQ** (gerenciado AWS, AMQPS porta 5671) em prod/EKS. Mesmo transport MassTransit em ambos — alterna pela env `MASSTRANSIT__TRANSPORT_SERVICE=AmazonMq` (default) ou `RabbitMq`.
> **Última revisão:** 2026-05-20

## Plugin obrigatório

O `UseDelayedRedelivery` do MassTransit (3 tentativas com backoff 1m/5m/15m) usa o exchange tipo `x-delayed-message`. Esse tipo só existe se o plugin estiver instalado.

Usamos uma **imagem custom** do Rabbit com o plugin pré-instalado:

```dockerfile
FROM rabbitmq:4-management
RUN rabbitmq-plugins enable rabbitmq_delayed_message_exchange
```

Dockerfile vive em `mecanica-hermes-tests-e2e/docker-compose/services/rabbitmq/Dockerfile`. É compartilhado com o ambiente local de todos os repositórios.

## Filas

Nome derivado do consumer via `KebabCaseEndpointNameFormatter` (do SDK).

Exemplos:

| Consumer C# | Fila |
|---|---|
| `OrdemDeServicoAguardandoAprovacaoEventConsumer` (Cadastros) | `ordem-de-servico-aguardando-aprovacao-event-consumer` |
| `OrcamentoAprovadoPeloClienteEventConsumer` (OS) | `orcamento-aprovado-pelo-cliente-event-consumer` |
| `AdicionarProdutoConsumer` (OS) | `adicionar-produto-consumer` |

DLQ é a fila com sufixo `_error` correspondente.

## Management UI

- Local: http://localhost:15672 (`guest` / `guest`).
- Em produção (TODO) — ainda não exposto externamente. Para inspecionar, port-forward via `kubectl port-forward`.

## Política de retry

Ver [Filas, retry, redelivery](Filas-retry-redelivery).

## Em produção (Amazon MQ for RabbitMQ)

Hoje rodamos **Amazon MQ for RabbitMQ** (broker `mechermes-mq`, engine `RabbitMQ 3.13`, instância `mq.m7g.medium`, single-AZ) provisionado via `mecanica-hermes-infra/shared/aws/module-amazonmq.tf`. As credenciais ficam no Secrets Manager (`mechermes-mq-secret`) e são injetadas pelo workflow `mecanica-hermes-k8s/.github/workflows/api-deploy.yml` como secret do Kubernetes (`app-secret`).

- **Endpoint AMQPS**: `amqps://mechermes:****@b-xxxx.mq.us-east-1.amazonaws.com:5671` (TLS 1.2 obrigatório).
- **Heartbeat**: 60s (configurado no host MassTransit — valores menores derrubam conexão sob carga).
- **Plugin `rabbitmq_delayed_message_exchange`**: **não é suportado** no Amazon MQ. Cada serviço resolve isso de forma diferente — ver [Estratégia de scheduler por serviço](#estratégia-de-scheduler-por-serviço) abaixo.

## Estratégia de scheduler por serviço

Como Amazon MQ não suporta o plugin `delayed-message-exchange`, o `UseDelayedMessageScheduler` (que depende dele) não funciona. Cada API faz a sua composição de scheduler no `Infrastructure/MassTransit/Configurations/`:

| Serviço | Estratégia | Por quê |
|---|---|---|
| **Ordem de Serviço** | **Quartz.NET** — `services.AddQuartz()` + `AddQuartzHostedService` + `cfg.UseMessageScheduler(new Uri("queue:quartz"))` + `AddQuartzConsumers()` em `ConfigureSaga`. JobStore RAM (default). | A SAGA usa `.Schedule(...)` para `OperationTimeout`. `AddQuartzConsumers` registra `ScheduleMessageConsumer` + `CancelScheduledMessageConsumer` que dependem de `Quartz.ISchedulerFactory` — sem o `services.AddQuartz()` o `QuartzBusObserver` falha no startup do bus. |
| **Pagamentos** | **`cfg.UseInMemoryScheduler()`** (requer pacote `MassTransit.Quartz` para a extensão estar disponível em compile-time). | A SAGA usa `.Schedule(...)` para `VerificarStatusPagamentoRequest` (polling MP a cada 2min). Em-memória é suficiente porque o polling reconcilia o estado mesmo após restart. |
| **Cadastros** | **Sem scheduler** — só `UseMessageRetry`. | Serviço puramente reativo (consumers de email + webhook + Outbox como BackgroundService); não tem SAGA com `.Schedule(...)`. |

**Trade-off em ambos OS e Pagamentos**: o RAMJobStore do Quartz / InMemoryScheduler são **não-duráveis** — timeouts agendados são perdidos se o pod morrer. Multi-replica também não coordena timeouts entre instâncias. Para durabilidade real, migrar OS para JobStore ADO.NET no Postgres (TODO em ADR futuro).

Alternativas avaliadas mas não adotadas:

- **Amazon SQS** — o MassTransit suporta SQS como transport (`MassTransit.AmazonSQS`); precisaria substituir polling agendado por SQS DelaySeconds (máximo 15min, suficiente). Empacotado no assembly, mas inativo.

## Toggle dev ↔ prod

A escolha entre RabbitMQ auto-hospedado (dev local) e Amazon MQ (prod) é feita pela env var:

```bash
# Dev local (docker-compose):
MASSTRANSIT__TRANSPORT_SERVICE=RabbitMq
MASSTRANSIT__CONNECTION_STRING=amqp://guest:guest@rabbitmq:5672/

# Prod (EKS, default quando env var ausente):
MASSTRANSIT__TRANSPORT_SERVICE=AmazonMq
MASSTRANSIT__CONNECTION_STRING=amqps://mechermes:****@b-xxxx.mq.us-east-1.amazonaws.com:5671
```

Ambos os modos usam o mesmo transport MassTransit RabbitMQ por baixo — a diferença está só na config de TLS e heartbeat.

## Conexão nos pods

Configurada pelo workflow de deploy (`api-deploy.yml`):

```bash
MASSTRANSIT__TRANSPORT_SERVICE=AmazonMq
MASSTRANSIT__CONNECTION_STRING=amqps://<user>:<password>@<broker-host>:5671
```

Em dev local, o hostname `rabbitmq` resolve pelo Service do Docker Compose.

## Veja também

- [SAGA com MassTransit](SAGA-com-MassTransit)
- [Filas, retry, redelivery](Filas-retry-redelivery)
- [DLQ observability](DLQ-observability)
