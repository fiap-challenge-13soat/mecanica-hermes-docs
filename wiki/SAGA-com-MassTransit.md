# SAGA com MassTransit

> **Rótulo:** Explicação
> **TL;DR:** SAGA serializa operações por chave do agregado. Máximo **1 operação em voo por OS** ou por Pagamento. Retry e redelivery automáticos. **Timeouts agendados** via Quartz (OS) ou InMemoryScheduler (Pagamentos) — não dependem do plugin RabbitMQ delayed-message que o Amazon MQ não suporta.
> **Última revisão:** 2026-05-20

## O problema que resolve

Sem SAGA, duas requisições concorrentes para a mesma OS podem ler estado consistente, mas escrever em cima uma da outra (lost update). Ou pior: avançar para `EmExecucao` enquanto outro comando ainda está em `AguardandoAprovacao`.

A SAGA **serializa** — enquanto uma operação está em voo para um `OrdemDeServicoId`, novos comandos para o mesmo ID ficam esperando na fila.

## Implementação

Usamos **MassTransit Sagas** com state machine. O `OrdemDeServicoStateMachine` (em `Infrastructure/MassTransit/`) define:

- **Initial state** — quando o primeiro comando chega para um `OrdemId`.
- **States** — representam onde a saga está esperando (em geral apenas `AwaitingConsumer`, mas pode ter estados intermediários para operações multi-passo).
- **Events** — `Execute*Request` (comandos) e `*Completed` / `*Failed` (callbacks).
- **Behaviors** — retry, redelivery, timeout.

Estado persiste em:

- **MongoDB** para a OS (coleção `saga_ordem_de_servico`) e Pagamentos (coleção `saga_pagamento`).

## Pipeline padrão de um comando

```mermaid
sequenceDiagram
  participant Api
  participant Saga as SAGA<br/>(MassTransit)
  participant Consumer
  participant Agg as Agregado<br/>+ UoW

  Api->>Saga: ExecuteAdicionarProdutoRequest(OrdemId, ProdutoId)
  Saga->>Saga: lock por OrdemId<br/>(grava state)
  Saga->>Consumer: AdicionarProdutoCommand
  Consumer->>Agg: carregar OS
  Consumer->>Agg: AdicionarProduto(...)
  Agg-->>Consumer: Result.Success + DomainEvent
  Consumer->>Agg: SaveChanges (entity + outbox)
  Consumer-->>Saga: AdicionarProdutoCompleted
  Saga->>Saga: libera lock
```

Se um segundo comando para o mesmo `OrdemId` chega enquanto o primeiro está em voo, ele fica na fila até a saga liberar o lock.

## Retry e redelivery

Política padrão (configurada em `BusFactoryConfiguratorExtensions.ConfigureCommonFactory`):

- **Immediate retry:** 3 tentativas no mesmo processo, sem backoff.
- **Delayed redelivery:** 3 tentativas com backoff exponencial (1m, 5m, 15m).
- **Total:** até ~21 minutos até cair na DLQ.

A delayed redelivery usa o plugin `rabbitmq_delayed_message_exchange` em dev local (por isso a imagem custom do Rabbit). Em produção (Amazon MQ for RabbitMQ), o plugin **não** é suportado e o redelivery cai pra reentrega imediata ao topo da fila — ver [Filas, retry, redelivery → Plugin em produção](Filas-retry-redelivery#plugin-delayed-message-exchange).

## Scheduler de timeouts (SAGA)

State machines que chamam `.Schedule(...)` precisam de um `IMessageScheduler` no bus. Como o Amazon MQ não suporta o plugin `delayed-message-exchange`, cada serviço escolheu:

- **OS** — Quartz.NET completo. `services.AddQuartz()` + `AddQuartzHostedService` em `ServiceCollectionExtensions` + `cfg.UseMessageScheduler(new Uri("queue:quartz"))` nos factories (RabbitMq e AmazonMq) + `AddQuartzConsumers()` em `ConfigureSaga`. JobStore RAM (não-durável).
- **Pagamentos** — `cfg.UseInMemoryScheduler()` (extensão fornecida pelo pacote `MassTransit.Quartz` — necessário em compile-time mesmo que não seja Quartz em runtime). Mais leve que Quartz, suficiente porque o polling do Mercado Pago reconcilia o estado se um timeout for perdido.
- **Cadastros** — sem scheduler (não tem SAGA).

Detalhes em [RabbitMQ / Amazon MQ → Estratégia de scheduler por serviço](RabbitMQ#estratégia-de-scheduler-por-serviço).

## Quem usa SAGA

- **API Ordem de Serviço** — SAGA principal. Todos os comandos de mutação passam por ela.
- **API Pagamentos** — SAGA própria, com timer extra para o polling do MP. Ver [Fluxo — Caminho feliz](Fluxo-Caminho-feliz#fase-de-pagamento).
- **API Cadastros** — **não** usa SAGA. É puramente reativo (consumers que enviam e-mail; webhook publica via Outbox).

## Idempotência dos consumers cross-service

Mesmo com SAGA serializando, eventos cross-service podem chegar duas vezes (retry do publisher do outro serviço ou redelivery do MP). Por isso os consumers de integração **verificam o estado-fonte** antes de despachar à SAGA — ver [Idempotência cross-service](Idempotencia-cross-service).

## Veja também

- [Filas, retry, redelivery](Filas-retry-redelivery) — detalhes da política
- [Outbox transacional](Outbox-transacional)
- [DLQ observability](DLQ-observability)
- [Catálogo de eventos](Catalogo-de-eventos)
