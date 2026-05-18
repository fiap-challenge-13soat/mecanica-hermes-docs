# Para avaliadores

> **Rótulo:** Tutorial (jornada guiada)
> **TL;DR:** Roteiro de leitura de 15 minutos para a banca avaliar o entregável.
> **Última revisão:** 2026-05-18

## Em 15 minutos você terá entendido

- O que é a Mecânica Hermes e como ela atende ao enunciado do Tech Challenge.
- O domínio de negócio (oficina mecânica) e o ciclo de uma Ordem de Serviço.
- A arquitetura cross-service e como os 3 microsserviços conversam.
- Quais decisões arquiteturais foram tomadas e por quê.

## Roteiro

### 1. Contexto (3 min)

- [Tech Challenge — FIAP 13SOAT](Tech-Challenge-FIAP-13SOAT) — mapa enunciado → entregável + critérios atendidos.

### 2. Negócio (3 min)

- [Domínio de negócio](Dominio-de-negocio) — quem são os atores, o que é uma OS, como ela transita entre estados.

### 3. Arquitetura (5 min)

- [Arquitetura](Arquitetura) — visão C4 nível 1 (contexto).
- [Contêineres](Conteineres) — C4 nível 2 (os 3 serviços, bancos, mensageria).
- [Catálogo de eventos](Catalogo-de-eventos) — os 7 eventos `.v1` que circulam entre serviços.

### 4. Demonstração (2 min)

- [Fluxo — Caminho feliz](Fluxo-Caminho-feliz) — narrativa passo a passo do que acontece quando uma OS é concluída.
- _Link do vídeo de apresentação (TODO)_.

### 5. Decisões (2 min)

- [Índice de ADRs](Indice-de-ADRs) — decisões arquiteturais agregadas dos 9 repositórios.

## Critérios para avaliação

A página [Tech Challenge](Tech-Challenge-FIAP-13SOAT#critérios-atendidos-auto-avaliação) traz uma auto-avaliação detalhada. Em resumo:

- Microsserviços, bounded contexts, mensageria com versionamento, idempotência, outbox.
- Observabilidade ponta-a-ponta (OTLP → New Relic).
- Cobertura ≥ 80%, suíte E2E em CI.
- IaC com Terraform separado por módulo.
- Autenticação OAuth2 via Cognito.

## Onde aprofundar (opcional)

- [Repositórios](Repositorios) — papel de cada um dos 9 repos.
- [SDK compartilhado](SDK-Visao-dos-6-pacotes) — como compartilhamos abstrações entre serviços.
- [Testes E2E](Testes-E2E) — 8 cenários validados em Robot Framework BDD.
