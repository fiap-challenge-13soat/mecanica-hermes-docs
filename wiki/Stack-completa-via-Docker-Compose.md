# Stack completa via Docker Compose

> **Rótulo:** Tutorial
> **TL;DR:** Sobe **todo** o ecossistema (3 APIs + Postgres + MongoDB + RabbitMQ + WireMock) em um único comando, usando o repositório de testes E2E.
> **Tempo estimado:** ~5 minutos.
> **Última revisão:** 2026-05-18

## Pré-requisitos

- Docker 24+ com Docker Compose v2.
- Conexão com o Docker Hub para baixar as imagens das 3 APIs (`mechermes/*`).

## Passo a passo

### 1. Clonar o repositório de testes E2E

```bash
git clone https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e.git
cd mecanica-hermes-tests-e2e
```

### 2. Subir a stack

```bash
docker compose \
  -f docker-compose/docker-compose.yaml \
  -f docker-compose/docker-compose.e2e.yaml \
  up -d --wait --build --pull always
```

O que cada flag faz:

- `--wait` — aguarda **todos** os healthchecks passarem antes de retornar.
- `--build` — builda a imagem custom do RabbitMQ (com plugin `rabbitmq_delayed_message_exchange`).
- `--pull always` — re-baixa as `:latest` das 3 APIs (`mechermes/*`) do Docker Hub.

### 3. Verificar healthchecks

```bash
curl http://localhost:8081/health   # OS
curl http://localhost:8082/health   # Cadastros
curl http://localhost:8083/health   # Pagamentos
curl http://localhost:8090/__admin/health  # WireMock (mock do Mercado Pago)
```

### 4. Acessar as UIs

| Recurso | URL |
|---|---|
| Scalar OS | http://localhost:8081/scalar |
| Scalar Cadastros | http://localhost:8082/scalar |
| Scalar Pagamentos | http://localhost:8083/scalar |
| RabbitMQ Management | http://localhost:15672 (`guest` / `guest`) |
| WireMock Admin | http://localhost:8090/__admin |

### 5. Derrubar tudo

```bash
docker compose \
  -f docker-compose/docker-compose.yaml \
  -f docker-compose/docker-compose.e2e.yaml \
  down -v
```

> O `-v` apaga também os volumes (Postgres, Mongo). Sem ele, dados persistem entre subidas.

## Autenticação em modo Testing

As APIs sobem com `ASPNETCORE_ENVIRONMENT=Testing`. Isso ativa o `DevelopmentAuthenticationMiddleware` que **bypassa JWT** — você pode chamar qualquer endpoint sem token. Em produção, JWT do Cognito é obrigatório.

## Tags específicas das imagens

Por padrão o compose usa `:latest`. Para fixar uma versão:

```bash
export OS_IMAGE_TAG=abc123
export CADASTROS_IMAGE_TAG=def456
export PAGAMENTOS_IMAGE_TAG=ghi789
docker compose -f ... up -d --wait
```

As imagens são republicadas a cada merge em `main` pelo workflow `Build and Publish Docker Image` de cada serviço.

## Problemas comuns

| Sintoma | Causa provável | Mitigação |
|---|---|---|
| `--wait` trava | Migration falhou | `docker compose logs <servico>` |
| `Connection refused` ao chamar `/health` | Startup ainda em andamento | Aguarde ~30s; primeiro pull pode levar mais |
| RabbitMQ sobe mas sem plugin delayed-message | Cache de imagem antigo | `docker compose build --no-cache rabbitmq` |
| Mongo não aceita transações | Replica set não inicializado | Verifique log: deve ter `rs.initiate()` rodado |

## Páginas relacionadas

- [Rodar a suíte E2E](Rodar-a-suite-E2E) — executar Robot contra esta stack
- [Catálogo de portas locais](Catalogo-de-portas-locais)
