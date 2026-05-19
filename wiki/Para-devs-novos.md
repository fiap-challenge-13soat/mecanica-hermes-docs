# Para devs novos

> **Rótulo:** Tutorial (jornada guiada)
> **TL;DR:** Roteiro de onboarding para um dev novo subir o ambiente e fazer a primeira contribuição.
> **Última revisão:** 2026-05-18

## Em 1 hora você terá conseguido

- Subir o ecossistema completo localmente.
- Executar a suíte E2E.
- Entender em qual repositório mexer para cada tipo de mudança.
- Conhecer as convenções do código (Clean Arch, Result, eventos `.v1`).

## Roteiro

### 1. Rodar o ecossistema (15 min)

- [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose) — sobe Postgres, MongoDB, RabbitMQ, WireMock + as 3 APIs com 1 comando.
- Confira healthchecks em `:8081/:8082/:8083`.

### 2. Entender o que está rodando (15 min)

- [Contêineres (C4 nível 2)](Conteineres) — diagrama do que sobe.
- [Componentes por serviço](Componentes-por-servico) — camadas internas de cada serviço.
- [Repositórios](Repositorios) — qual repo cuida do quê.

### 3. Aprender as convenções (20 min)

- [Clean Architecture + DDD + CQRS](Clean-Architecture-DDD-CQRS) — regra de dependência e estrutura de projetos.
- [Result Pattern](Result-Pattern) — como tratamos sucesso/falha.
- [State Pattern](State-Pattern) — como modelamos transições de estado.
- [SAGA com MassTransit](SAGA-com-MassTransit) e [Outbox transacional](Outbox-transacional) — padrões de mensageria.
- [Catálogo de eventos](Catalogo-de-eventos) — quem publica e consome o quê.

### 4. Rodar os testes (5 min)

- [Rodar a suíte E2E](Rodar-a-suite-E2E) — BDD em Robot Framework com Allure.
- [Estratégia de testes](Estrategia-de-testes) — pirâmide e convenções.

### 5. Contribuir (5 min)

- [Como contribuir](Como-contribuir).
- [Padrão de commits e PRs](Padrao-de-commits-e-PRs).
- [Code review checklist](Code-review-checklist).

## Matéria-prima de cada repo

| Repo | Tecnologia | Onde você provavelmente vai mexer |
|---|---|---|
| `api-ordem-servico` | .NET 10 + Postgres + Rabbit + Mongo (saga) | Adicionar comando, ajustar state machine, novo consumer |
| `api-cadastros` | .NET 10 + Postgres | Cadastros de clientes/produtos, webhook de orçamento |
| `api-pagamentos` | .NET 10 + Mongo | Integração Mercado Pago, reconciliação webhook+polling |
| `api-sdk` | .NET 10 | Eventos `.v1`, abstrações compartilhadas |
| `tests-e2e` | Python 3.11 + Robot | Novas suítes E2E |
| `infra` | Terraform | Novos recursos AWS |
| `k8s` | Kustomize | Manifests, HPA, Secrets |
| `lambda` | .NET 10 (Lambda) | Login do cliente por CPF |
| `docs` | Markdown | Esta Wiki :) |

## Próximos passos

- Faça sua primeira PR com algo pequeno (typo, melhoria de mensagem, teste extra).
- Peça review no Discord (handles em [Equipe](Equipe)).
