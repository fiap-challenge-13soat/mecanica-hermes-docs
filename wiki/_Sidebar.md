**🏠 [Home](Home)**

**📖 Visão geral**
- [Tech Challenge — FIAP 13SOAT](Tech-Challenge-FIAP-13SOAT)
- [Domínio de negócio](Dominio-de-negocio)
- [Equipe](Equipe)
- [Glossário](Glossario)

**🧭 Como usar a Wiki**
- [Para avaliadores](Para-avaliadores)
- [Para devs novos](Para-devs-novos)
- [Para operadores](Para-operadores)
- [Para integradores externos](Para-integradores-externos)

**🚀 Início rápido**
- [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose)
- [Subir um único serviço](Subir-um-unico-servico)
- [Rodar a suíte E2E](Rodar-a-suite-E2E)
- [Deploy em AWS](Deploy-em-AWS)

**🏛️ Arquitetura**
- [Visão geral (C4 nível 1)](Arquitetura)
- [Contêineres (C4 nível 2)](Conteineres)
- [Componentes por serviço (C4 nível 3)](Componentes-por-servico)

_Padrões transversais_
- [Clean Architecture + DDD + CQRS](Clean-Architecture-DDD-CQRS)
- [Result Pattern](Result-Pattern)
- [State Pattern](State-Pattern)
- [SAGA com MassTransit](SAGA-com-MassTransit)
- [Outbox transacional](Outbox-transacional)
- [Idempotência cross-service](Idempotencia-cross-service)
- [DLQ observability](DLQ-observability)

_Mensageria_
- [Catálogo de eventos .v1](Catalogo-de-eventos)
- [Versionamento de contratos](Versionamento-de-contratos)
- [Filas, retry, redelivery](Filas-retry-redelivery)

_Persistência_
- [PostgreSQL](PostgreSQL)
- [MongoDB](MongoDB)

_Segurança_
- [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT)
- [Autorização por scopes](Autorizacao-por-scopes)
- [M2M Client Credentials](M2M-Client-Credentials)
- [Webhooks assinados (HMAC)](Webhooks-assinados-HMAC)
- [Modelo de ameaças](Modelo-de-ameacas)

**📂 [Repositórios](Repositorios)**
- [`infra`](Repo-mecanica-hermes-infra)
- [`k8s`](Repo-mecanica-hermes-k8s)
- [`lambda`](Repo-mecanica-hermes-lambda)
- [`api-ordem-servico`](Repo-mecanica-hermes-api-ordem-servico)
- [`api-cadastros`](Repo-mecanica-hermes-api-cadastros)
- [`api-pagamentos`](Repo-mecanica-hermes-api-pagamentos)
- [`api-sdk`](Repo-mecanica-hermes-api-sdk)
- [`tests-e2e`](Repo-mecanica-hermes-tests-e2e)
- [`docs`](Repo-mecanica-hermes-docs)

**🔁 Fluxos de negócio**
- [Caminho feliz](Fluxo-Caminho-feliz)
- [Orçamento rejeitado](Fluxo-Orcamento-rejeitado)
- [Pagamento recusado e recriado](Fluxo-Pagamento-cancelado-recriado)
- [Pagamento expirado](Fluxo-Pagamento-expirado)
- [Cancelamento em execução](Fluxo-Cancelamento-em-execucao)
- [Cancelamento em aguardando pagamento](Fluxo-Cancelamento-em-aguardando-pagamento)
- [Idempotência de webhook](Fluxo-Idempotencia-de-webhook)
- [Proteção por timeout da SAGA](Fluxo-Protecao-por-timeout-da-SAGA)

**🛠️ Infraestrutura e deploy**
- [Diagrama AWS completo](Diagrama-AWS-completo)
- [Provisionamento (Terraform)](Provisionamento-Terraform)
- [Cluster Kubernetes](Cluster-Kubernetes)
- [API Gateway + VPC Link](API-Gateway-VPC-Link)
- [Cognito](Cognito-User-Pool)
- [Lambda CognitoToken](Lambda-CognitoToken)
- [Bancos de dados](Bancos-de-dados)
- [RabbitMQ](RabbitMQ)
- [Pipelines GitHub Actions](Pipelines-GitHub-Actions)

**📊 Observabilidade**
- [Visão geral](Observabilidade)
- [Logs](Logs)
- [Métricas de negócio](Metricas-de-negocio)
- [Métricas de mensageria](Metricas-de-mensageria)
- [Traces distribuídos](Traces-distribuidos)
- [Dashboards e alertas](Dashboards-e-alertas)

**🧪 Qualidade**
- [Estratégia de testes](Estrategia-de-testes)
- [Testes E2E](Testes-E2E)
- [Cobertura e SonarCloud](Cobertura-e-SonarCloud)
- [Architecture Tests](Architecture-Tests)
- [Relatórios Allure](Relatorios-Allure)

**🚨 Operação**
- [Runbook macro](Runbook-macro)
- [Runbook — Ordem de Serviço](Runbook-Ordem-de-Servico)
- [Runbook — Cadastros](Runbook-Cadastros)
- [Runbook — Pagamentos](Runbook-Pagamentos)
- [Resposta a incidentes](Resposta-a-incidentes)
- [Forçar limpeza de namespace](Forcar-limpeza-de-namespace)

**🧩 SDK compartilhado**
- [Visão dos 6 pacotes](SDK-Visao-dos-6-pacotes)
- [Como consumir](SDK-Como-consumir)
- [Versionamento](SDK-Versionamento)
- [Como contribuir com o SDK](SDK-Como-contribuir)

**👥 Contribuição**
- [Como contribuir](Como-contribuir)
- [Padrão de commits e PRs](Padrao-de-commits-e-PRs)
- [Branching e ambientes](Branching-e-ambientes)
- [Code review checklist](Code-review-checklist)
- [Editar a Wiki](Editar-a-Wiki)

**📚 Referência**
- [API pública](API-publica)
- [Variáveis de ambiente](Variaveis-de-ambiente)
- [Catálogo de portas locais](Catalogo-de-portas-locais)
- [Endpoints de health-check](Endpoints-de-health-check)
- [Comandos `just`](Comandos-just)

**🧠 Decisões**
- [Índice de ADRs](Indice-de-ADRs)
- [Índice de RFCs](Indice-de-RFCs)
- [Histórico cronológico](Historico-cronologico)
