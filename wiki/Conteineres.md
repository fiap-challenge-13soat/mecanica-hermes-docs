# Contêineres (C4 nível 2)

> **Rótulo:** Explicação
> **TL;DR:** Como o sistema Hermes se decompõe em microsserviços, bancos de dados e mensageria.
> **Última revisão:** 2026-05-18

## Diagrama

```mermaid
flowchart TB
  subgraph EXT[Externo]
    User([Usuário + Cliente])
    MP[Mercado Pago]
    SMTP[SMTP]
  end

  subgraph AWS[AWS]
    APIG[API Gateway HTTP<br/>+ Cognito Authorizer]
    Cognito[Cognito User Pool]
    Lambda[Lambda<br/>CognitoToken]

    subgraph EKS[EKS Cluster]
      OS[API Ordem de Serviço<br/>.NET 10]
      Cad[API Cadastros<br/>.NET 10]
      Pag[API Pagamentos<br/>.NET 10]
    end

    subgraph DB[Persistência]
      RDS[(RDS PostgreSQL<br/>OS + Cadastros)]
      Mongo[(MongoDB<br/>Pagamentos + SAGA state)]
    end

    Rabbit[(RabbitMQ<br/>+ delayed-message plugin)]
  end

  User -->|HTTPS| APIG
  APIG -->|/oauth2/token| Lambda
  APIG -->|VPC Link| OS
  APIG -->|VPC Link| Cad
  APIG -->|VPC Link| Pag
  APIG --> Cognito
  Lambda --> Cognito
  Lambda --> RDS
  Lambda --> SMTP

  OS --> RDS
  OS --> Mongo
  Cad --> RDS
  Pag --> Mongo

  OS <-->|eventos .v1| Rabbit
  Cad <-->|eventos .v1| Rabbit
  Pag <-->|eventos .v1| Rabbit

  Pag <-->|HTTP + webhook HMAC| MP
  Cad -->|HTTP M2M JWT| Cad
  Pag -.->|GET /api/clientes/{id}| Cad
  OS -.->|GET /api/clientes/{id}| Cad
  Cad -->|e-mails| SMTP
  Pag -->|e-mails| SMTP
```

## Contêineres

| Contêiner | Tecnologia | Responsabilidade |
|---|---|---|
| **API Gateway** | AWS API Gateway HTTP | Ponto único de entrada. Valida JWT pelo Cognito Authorizer, roteia para EKS via VPC Link ou para Lambda |
| **Cognito User Pool** | AWS Cognito | Emite JWTs via Client Credentials flow |
| **Lambda CognitoToken** | .NET 10 (Lambda) | Login do cliente final: CPF → valida cadastro no RDS → obtém JWT no Cognito → envia por e-mail |
| **API Ordem de Serviço** | .NET 10 em EKS | Orquestrador principal. State machine da OS. SAGA principal |
| **API Cadastros** | .NET 10 em EKS | Clientes, produtos, webhook de orçamento, endpoint M2M `GET /api/clientes/{id}` |
| **API Pagamentos** | .NET 10 em EKS | Integração Mercado Pago. SAGA própria com webhook + polling |
| **RDS PostgreSQL** | RDS 17.6 (`db.t4g.micro`) | Bancos `MecHermesOrdemDeServicoDB` e `mecanica_hermes_cadastros` |
| **MongoDB** | Mongo 7 (replica set) | Estado da SAGA da OS + agregado e Outbox de Pagamentos |
| **RabbitMQ** | 4.x + plugin `delayed-message-exchange` | Transport do MassTransit. Plugin usado pelo polling de status MP |

## Por que essa decomposição

- **OS x Cadastros x Pagamentos** são **bounded contexts** distintos. Cada um tem seu próprio modelo (state machine vs CRUD vs gateway) e suas próprias regras.
- **PostgreSQL para OS e Cadastros** — dados relacionais com transações ACID. EF Core é confortável.
- **MongoDB para Pagamentos** — documento mais natural para o agregado (com histórico embarcado) e suporte nativo ao Outbox via `IClientSessionHandle` no replica set.
- **RabbitMQ** — transport estável, plugin de delayed-message essencial para o polling do MP.

## Veja também

- [Componentes por serviço](Componentes-por-servico) — zoom interno de cada API
- [Catálogo de eventos](Catalogo-de-eventos) — mensagens que circulam no Rabbit
- [Diagrama AWS completo](Diagrama-AWS-completo) — todos os recursos provisionados
