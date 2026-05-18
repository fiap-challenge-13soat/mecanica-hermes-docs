# Glossário

> **Rótulo:** Referência
> **TL;DR:** Termos técnicos e de negócio repetidos pela Wiki, definidos em uma única página.
> **Última revisão:** 2026-05-18

## Negócio

- **Ordem de Serviço (OS)** — agregado central; representa o atendimento completo de um veículo, do recebimento à entrega.
- **Orçamento** — soma de produtos + serviços de uma OS antes da aprovação do cliente.
- **Snapshot de preço** — cópia do preço unitário no momento da inclusão na OS (mudanças no catálogo não retroagem).
- **Webhook de orçamento** — endpoint público assinado por HMAC que o cliente acessa por link enviado por e-mail para aprovar/rejeitar o orçamento.

## Arquitetura

- **Clean Architecture** — separação em camadas com regra de dependência estrita: `Domain ← Application ← Infrastructure`, `Api → all`. Ver [Clean Architecture + DDD + CQRS](Clean-Architecture-DDD-CQRS).
- **DDD (Domain-Driven Design)** — modelagem orientada a domínio com agregados, value objects, eventos de domínio.
- **CQRS** — separação entre comandos (escrita) e queries (leitura). Mediado por MediatR.
- **State Pattern** — cada status do agregado é uma classe que conhece suas próprias transições. Ver [State Pattern](State-Pattern).
- **Result Pattern** — operações retornam `Result<T>` com `HttpStatusCode`, `IsSuccess` e `Errors`. Ver [Result Pattern](Result-Pattern).
- **Ports & Adapters (Hexagonal)** — interfaces (Ports) na camada de Application, implementações (Adapters) na Infrastructure.

## Mensageria

- **SAGA** — máquina de estados distribuída que serializa operações por chave (ex.: `OrdemDeServicoId`). Implementada via MassTransit. Ver [SAGA com MassTransit](SAGA-com-MassTransit).
- **Outbox transacional** — eventos persistidos na mesma transação do agregado e despachados depois por um background service. Garantia *at-least-once*. Ver [Outbox transacional](Outbox-transacional).
- **Idempotência cross-service** — consumers verificam o estado-fonte antes de despachar à SAGA, ignorando entregas tardias/duplicadas. Ver [Idempotência cross-service](Idempotencia-cross-service).
- **DLQ (Dead Letter Queue)** — fila de mensagens que falharam após todos os retries. Observada via métrica `messaging.consume_faults`. Ver [DLQ observability](DLQ-observability).
- **`.v1`** — sufixo de versão obrigatório em todo `[EntityName]` de evento cross-service. Mudança de schema cria `.v2` em paralelo. Ver [Versionamento de contratos](Versionamento-de-contratos).
- **At-least-once** — política de entrega que garante que cada evento será entregue pelo menos uma vez (pode duplicar — por isso idempotência).

## Infraestrutura e plataforma

- **EKS** — Amazon Elastic Kubernetes Service. Cluster privado onde rodam as 3 APIs.
- **Kustomize** — ferramenta de overlay declarativo para Kubernetes (`base/` + `overlays/{hml,prd}/`).
- **VPC Link** — recurso do API Gateway HTTP que permite chamar serviços privados no EKS.
- **OTLP** — OpenTelemetry Protocol; usado para exportar traces/métricas/logs ao New Relic.
- **CPM (Central Package Management)** — versões NuGet declaradas em `Directory.Packages.props`; `.csproj` referencia sem `Version=`.

## Segurança

- **Cognito** — AWS Cognito User Pool. Fonte de tokens JWT.
- **Client Credentials Flow** — fluxo OAuth2 usado para autenticação machine-to-machine (M2M) e para o login de cliente via CPF (via Lambda).
- **M2M** — Machine-to-Machine; chamadas HTTP entre serviços usando JWT obtido via Client Credentials.
- **HMAC** — Hash-based Message Authentication Code; usado para assinar webhooks (orçamento e Mercado Pago).
- **Scope** — escopos JWT (`mechermes/admin`, `mechermes/client`, `servico-scope`) que mapeiam para políticas de autorização.

## Tecnologias

- **MassTransit** — framework .NET de mensageria com suporte a Sagas, Outbox e múltiplos transports.
- **MediatR** — biblioteca .NET para CQRS in-process (request/response e notification).
- **Autofac** — container de DI usado pelos 3 serviços .NET.
- **EF Core** — ORM para PostgreSQL nos serviços OS e Cadastros.
- **Npgsql** — driver PostgreSQL puro (usado pela Lambda sem EF Core para reduzir cold start).
- **Testcontainers** — sobe containers Docker (Postgres, Mongo, RabbitMQ) durante testes de integração.
- **WireMock.Net** — mock HTTP usado para emular o Mercado Pago em testes.
- **Scalar** — UI interativa de OpenAPI (substitui Swagger UI nas APIs).
- **just** — task runner (`justfile`); atalhos `just build`, `just test`, `just run`.
- **Allure** — relatórios ricos de teste, publicados em GitHub Pages para a suíte E2E.

## Páginas relacionadas

- [Arquitetura](Arquitetura)
- [Catálogo de eventos](Catalogo-de-eventos)
