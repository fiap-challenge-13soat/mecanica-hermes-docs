# Subir um único serviço

> **Rótulo:** How-to
> **TL;DR:** Como rodar apenas **uma** das APIs .NET 10 localmente, com dependências mínimas, sem subir o ecossistema completo.
> **Última revisão:** 2026-05-18

## Quando usar

- Você está mexendo só em um serviço e quer iterar rápido (sem rebuild de Docker).
- Quer atachar debugger ao processo.
- Não precisa de comunicação cross-service real (vai mockar ou pular).

Para o ecossistema completo, prefira [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose).

## Pré-requisitos

- [.NET SDK 10.0+](https://dotnet.microsoft.com/download/dotnet/10.0)
- Docker (só para dependências — Postgres, Mongo, RabbitMQ)
- `just` (opcional, atalhos prontos)
- Acesso ao GitHub Packages com PAT classic (`read:packages`) para baixar o SDK — ver [SDK — Como consumir](SDK-Como-consumir).

## Para `api-ordem-servico`

```bash
# Dependências
docker run -d --name hermes-postgres -p 5432:5432 \
  -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=MecHermesOrdemDeServicoDB postgres:16

docker run -d --name hermes-rabbitmq -p 5672:5672 -p 15672:15672 \
  rabbitmq:3-management

docker run -d --name hermes-mongo -p 27017:27017 mongo:7

# Rodar a API
dotnet run --project src/Mecanica.Hermes.Api
# ou: just run
```

API sobe em http://localhost:5016. Scalar em /scalar.

## Para `api-cadastros`

```bash
docker run -d --name pg-cadastros -p 5432:5432 \
  -e POSTGRES_PASSWORD=postgres -e POSTGRES_DB=mecanica_hermes_cadastros \
  postgres:18.1

export ConnectionStrings__DefaultConnection="Host=localhost;Database=mecanica_hermes_cadastros;Username=postgres;Password=postgres"
export AUTH__AUTHORITY=https://test
export AUTH__ISSUER=https://test
export EmailSender__Enabled=false
export WEBHOOK__SIGNING_SECRET=meu-segredo-de-32-bytes-ou-mais-aqui

dotnet run --project src/Mecanica.Hermes.Cadastros.Api
# ou: just run
```

## Para `api-pagamentos`

```bash
# Mongo precisa de replica set
docker run -d --name hermes-pag-mongo -p 27017:27017 mongo:7 \
  --replSet rs0 --bind_ip_all
docker exec hermes-pag-mongo mongosh --eval "rs.initiate()"

docker run -d --name hermes-pag-rabbitmq -p 5672:5672 -p 15672:15672 \
  rabbitmq:3-management
# Ative o plugin de delayed message exchange:
docker exec hermes-pag-rabbitmq rabbitmq-plugins enable rabbitmq_delayed_message_exchange || true

dotnet run --project src/Mecanica.Hermes.Pagamento.Api
```

API sobe em http://localhost:5017.

## Variáveis de ambiente mínimas

Cada serviço tem um `docs/configuration.md` listando **todas** as env vars. As mínimas para rodar em `Development`:

| Variável | Serviço | Exemplo |
|---|---|---|
| `ConnectionStrings__DefaultConnection` | OS, Cadastros | Postgres connection string |
| `Mongo__ConnectionString` | OS (saga), Pagamentos | `mongodb://localhost:27017/?replicaSet=rs0` |
| `MassTransit__RabbitMq__Host` | Todos | `amqp://guest:guest@localhost:5672` |
| `AUTH__AUTHORITY` | Todos | qualquer URL em DEV (bypass ativo) |
| `WEBHOOK__SIGNING_SECRET` | Cadastros | secret de 32+ bytes |
| `MERCADO_PAGO__WEBHOOK_SECRET` | Pagamentos | secret de 32+ bytes |

Ver consolidado em [Variáveis de ambiente](Variaveis-de-ambiente).

## Bypass de autenticação em DEV

Em `Development` e `Testing`, o `DevelopmentAuthenticationMiddleware` substitui o JWT por uma identidade default — você pode chamar endpoints protegidos sem token.

## Páginas relacionadas

- [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose) (alternativa: tudo de uma vez)
- [Comandos `just`](Comandos-just)
- [Variáveis de ambiente](Variaveis-de-ambiente)
