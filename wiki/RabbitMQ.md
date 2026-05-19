# RabbitMQ

> **Rótulo:** Referência
> **TL;DR:** RabbitMQ 4.x com plugin `rabbitmq_delayed_message_exchange`. Transport do MassTransit para mensageria entre os 3 serviços.
> **Última revisão:** 2026-05-18

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

## Em produção

Hoje rodamos o Rabbit como Pod no EKS (StatefulSet com PVC). Alternativas para o futuro:

- **Amazon MQ for RabbitMQ** (gerenciado, mas o plugin delayed-message **não é suportado** — ver matriz de plugins).
- **Amazon SQS** — o MassTransit suporta SQS como transport (`MassTransit.AmazonSQS`); precisaria substituir polling agendado por SQS DelaySeconds (máximo 15min, suficiente).

## Conexão nos pods

```bash
MassTransit__RabbitMq__Host=amqp://guest:guest@rabbitmq:5672
```

O hostname `rabbitmq` resolve pelo Service do Kubernetes.

## Veja também

- [SAGA com MassTransit](SAGA-com-MassTransit)
- [Filas, retry, redelivery](Filas-retry-redelivery)
- [DLQ observability](DLQ-observability)
