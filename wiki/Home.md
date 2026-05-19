# 🔧 Mecânica Hermes

> **Backend distribuído para gestão de uma oficina mecânica.**
> Recebe veículos, monta orçamentos, recolhe a aprovação do cliente por e-mail/webhook, integra cobrança com Mercado Pago e devolve o carro ao cliente quando o pagamento é confirmado.
>
> Projeto desenvolvido pelo grupo do **Tech Challenge 13SOAT — FIAP**.

## Contexto em 1 diagrama

```mermaid
flowchart LR
  User([👤 Cliente / Atendente]):::actor
  MP([💳 Mercado Pago]):::ext
  SMTP([📧 SMTP]):::ext
  NR([📊 New Relic]):::ext

  subgraph Hermes[Mecânica Hermes]
    APIG[API Gateway + Cognito]
    Lambda[Lambda CognitoToken]
    OS[API Ordem de Serviço]
    Cad[API Cadastros]
    Pag[API Pagamentos]
  end

  User -->|HTTPS + JWT| APIG
  APIG --> Lambda
  APIG --> OS
  APIG --> Cad
  APIG --> Pag
  Pag <-->|preferences + webhook HMAC| MP
  Cad -->|orçamento + token webhook| SMTP
  Lambda -->|token JWT por e-mail| SMTP
  OS -.-> NR
  Cad -.-> NR
  Pag -.-> NR

  classDef actor fill:#2d6a4f,color:#fff
  classDef ext fill:#6c757d,color:#fff
```

## Comece por aqui

Escolha a sua jornada:

| Você é... | Comece por |
|---|---|
| 🎓 **Avaliador / banca FIAP** | [Para avaliadores](Para-avaliadores) → [Tech Challenge — FIAP 13SOAT](Tech-Challenge-FIAP-13SOAT) → [Domínio de negócio](Dominio-de-negocio) → [Arquitetura](Arquitetura) |
| 👩‍💻 **Dev novo no time** | [Para devs novos](Para-devs-novos) → [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose) → [Componentes por serviço](Componentes-por-servico) → [Como contribuir](Como-contribuir) |
| 🛠️ **Operador / SRE** | [Para operadores](Para-operadores) → [Runbook macro](Runbook-macro) → [Observabilidade](Observabilidade) |
| 🔌 **Integrador externo** | [Para integradores externos](Para-integradores-externos) → [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT) → [API pública](API-publica) |

## O que tem aqui

A Wiki está organizada em 13 seções (veja o menu lateral). As mais consultadas:

- 🏛️ **[Arquitetura](Arquitetura)** — visão C4 do sistema e padrões transversais (Clean Architecture, SAGA, Outbox, State Pattern).
- 🔁 **[Fluxos de negócio](Fluxo-Caminho-feliz)** — os 8 cenários cobertos pela suíte E2E, descritos passo a passo.
- 📂 **[Repositórios](Repositorios)** — os 9 repos do ecossistema, seus papéis e como se conectam.
- 📨 **[Catálogo de eventos](Catalogo-de-eventos)** — os 8 contratos `.v1` que circulam no RabbitMQ.
- 🚨 **[Runbook macro](Runbook-macro)** — triagem inicial para incidentes cross-service.

## Status do projeto

| Serviço | Build / coverage |
|---|---|
| `api-ordem-servico` | [![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=fiap-challenge-13soat_mecanica-hermes-api-ordem-servico&metric=alert_status&token=6aa5812f4a2c7616442ace938524a8b0999b56c3)](https://sonarcloud.io/summary/new_code?id=fiap-challenge-13soat_mecanica-hermes-api-ordem-servico) [![Coverage](https://sonarcloud.io/api/project_badges/measure?project=fiap-challenge-13soat_mecanica-hermes-api-ordem-servico&metric=coverage&token=6aa5812f4a2c7616442ace938524a8b0999b56c3)](https://sonarcloud.io/summary/new_code?id=fiap-challenge-13soat_mecanica-hermes-api-ordem-servico) |
| `api-cadastros` | [![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=fiap-challenge-13soat_mecanica-hermes-cadastros&metric=alert_status&token=71b077675868d45c85c3f6a512e5ac8d9cc2f8d2)](https://sonarcloud.io/summary/new_code?id=fiap-challenge-13soat_mecanica-hermes-cadastros) [![Coverage](https://sonarcloud.io/api/project_badges/measure?project=fiap-challenge-13soat_mecanica-hermes-cadastros&metric=coverage&token=71b077675868d45c85c3f6a512e5ac8d9cc2f8d2)](https://sonarcloud.io/summary/new_code?id=fiap-challenge-13soat_mecanica-hermes-cadastros) |
| `api-pagamentos` | [![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=fiap-challenge-13soat_mecanica-hermes-api-pagamentos&metric=alert_status&token=87e572f763af380c4014521d5d907701425e530c)](https://sonarcloud.io/summary/new_code?id=fiap-challenge-13soat_mecanica-hermes-api-pagamentos) [![Coverage](https://sonarcloud.io/api/project_badges/measure?project=fiap-challenge-13soat_mecanica-hermes-api-pagamentos&metric=coverage&token=87e572f763af380c4014521d5d907701425e530c)](https://sonarcloud.io/summary/new_code?id=fiap-challenge-13soat_mecanica-hermes-api-pagamentos) |
| `api-sdk` | [![Quality Gate](https://sonarcloud.io/api/project_badges/measure?project=fiap-challenge-13soat_mecanica-hermes-shared-sdk&metric=alert_status&token=cb19f9f842a7972ed0b9a881bf4ba19155ac172e)](https://sonarcloud.io/summary/new_code?id=fiap-challenge-13soat_mecanica-hermes-shared-sdk) |

_Cobertura mínima exigida pelo CI: **≥ 80%** de linhas em todos os serviços .NET._

## Sobre esta Wiki

Esta Wiki é mantida em arquivos Markdown versionados no repositório [`mecanica-hermes-docs`](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs), seguindo o **[plano editorial](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/WIKI-PLAN.md)**. Mudanças passam por Pull Request com revisão.

> **Última atualização:** 2026-05-19
