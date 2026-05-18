# Tech Challenge — FIAP 13SOAT

> **Rótulo:** Explicação
> **TL;DR:** Como o projeto Mecânica Hermes mapeia para o enunciado do Tech Challenge 13SOAT da FIAP.
> **Última revisão:** 2026-05-18

## Contexto acadêmico

O **Tech Challenge** é a entrega final integrada de cada fase da pós-graduação em **Software Architecture (SOAT)** da FIAP. O grupo escolheu o domínio de **gestão de uma oficina mecânica** — a Mecânica Hermes — para aplicar os conceitos de cada fase em um sistema com complexidade realista.

O grupo é da turma **13SOAT** (13ª turma de Software Architecture).

## Fases entregues

| Fase | Tema da fase | Como atendemos |
|---|---|---|
| **1** | Domínio + arquitetura limpa | Modelagem DDD do agregado `OrdemDeServico` com State Pattern + Clean Architecture + CQRS. Ver [Domínio de negócio](Dominio-de-negocio) |
| **2** | Containers + Kubernetes | Imagens Docker das 3 APIs publicadas em Docker Hub + deploy via Kustomize em EKS. Ver [Cluster Kubernetes](Cluster-Kubernetes) |
| **3** | Microsserviços + integração | Quebra em 3 serviços (OS, Cadastros, Pagamentos) integrados por **mensageria** (RabbitMQ/MassTransit com SAGA e Outbox) e **HTTP M2M**. Ver [Catálogo de eventos](Catalogo-de-eventos) |
| **4** | Observabilidade + qualidade | OpenTelemetry → New Relic, Serilog estruturado, SonarCloud com cobertura ≥ 80%, suíte E2E em Robot Framework com Allure. Ver [Observabilidade](Observabilidade) e [Testes E2E](Testes-E2E) |
| **5** | Cloud + autenticação | Infraestrutura AWS via Terraform (VPC, EKS, RDS, Cognito, API Gateway), autenticação OAuth2 via Cognito + Lambda CPF para clientes. Ver [Diagrama AWS completo](Diagrama-AWS-completo) e [Lambda CognitoToken](Lambda-CognitoToken) |

## Mapeamento enunciado → entregável

| Requisito do enunciado | Onde está atendido |
|---|---|
| Modelagem de domínio com agregados | [Domínio de negócio](Dominio-de-negocio) · `api-ordem-servico/src/Mecanica.Hermes.Domain/` |
| Clean Architecture / Ports & Adapters | [Clean Architecture + DDD + CQRS](Clean-Architecture-DDD-CQRS) |
| Comunicação assíncrona entre microsserviços | [SAGA com MassTransit](SAGA-com-MassTransit) · [Catálogo de eventos](Catalogo-de-eventos) |
| Banco relacional + NoSQL | [PostgreSQL](PostgreSQL) (OS, Cadastros) · [MongoDB](MongoDB) (Pagamentos + SAGA state) |
| Containerização | Dockerfiles multi-stage por serviço + `docker-compose` no repo de E2E |
| Orquestração com Kubernetes | [Cluster Kubernetes](Cluster-Kubernetes) — Kustomize com overlays `hml`/`prd` |
| Observabilidade | [Observabilidade](Observabilidade) — OTLP → New Relic + Serilog |
| Testes automatizados (cobertura ≥ 80%) | [Estratégia de testes](Estrategia-de-testes) · [Cobertura e SonarCloud](Cobertura-e-SonarCloud) |
| Testes ponta-a-ponta (BDD) | [Testes E2E](Testes-E2E) — 8 suítes Robot Framework |
| Autenticação e autorização | [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT) · [Autorização por scopes](Autorizacao-por-scopes) |
| Infraestrutura como código (cloud) | [Provisionamento (Terraform)](Provisionamento-Terraform) — 5 módulos |
| Decisões arquiteturais documentadas (ADRs) | [Índice de ADRs](Indice-de-ADRs) |

## Vídeo de demonstração

_TODO — a confirmar com o time o link público da apresentação._

## Critérios atendidos (auto-avaliação)

- ✅ Microsserviços com bounded contexts claros (OS, Cadastros, Pagamentos)
- ✅ Mensageria com versionamento de contratos (`.v1`)
- ✅ Idempotência cross-service (consumers verificam estado-fonte)
- ✅ Outbox transacional (atomicidade evento + agregado)
- ✅ Observabilidade ponta-a-ponta (traces, logs, métricas)
- ✅ Cobertura ≥ 80% nos 3 serviços + SDK
- ✅ Suíte E2E rodando em CI com publicação de Allure no GitHub Pages
- ✅ IaC com Terraform separado por módulo, estado em S3, lock em DynamoDB
- ✅ Deploy automatizado por ambiente (hml/prd)
- ✅ ADRs registrados nos 3 serviços + SDK

## Páginas relacionadas

- [Domínio de negócio](Dominio-de-negocio)
- [Equipe](Equipe)
- [Arquitetura](Arquitetura)
