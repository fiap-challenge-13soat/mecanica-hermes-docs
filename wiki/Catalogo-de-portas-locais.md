# Catálogo de portas locais

> **Rótulo:** Referência
> **TL;DR:** Quais portas cada serviço usa em desenvolvimento.
> **Última revisão:** 2026-05-18

## APIs .NET

| Serviço | Porta (modo standalone) | Porta (modo Compose E2E) |
|---|---|---|
| API Ordem de Serviço | 5016 | 8081 |
| API Cadastros | (a fixar; varável) | 8082 |
| API Pagamentos | 5017 | 8083 |

## Infraestrutura

| Serviço | Porta |
|---|---|
| PostgreSQL | 5432 |
| MongoDB | 27017 |
| RabbitMQ AMQP | 5672 |
| RabbitMQ Management UI | 15672 (`guest`/`guest`) |
| WireMock (mock MP) | 8090 |

## UIs e diagnóstico

| URL | O que é |
|---|---|
| http://localhost:8081/scalar | OpenAPI OS |
| http://localhost:8082/scalar | OpenAPI Cadastros |
| http://localhost:8083/scalar | OpenAPI Pagamentos |
| http://localhost:15672 | RabbitMQ Management |
| http://localhost:8090/__admin | WireMock Admin |

## Em produção

Tudo passa pelo **API Gateway HTTPS** em produção. Acesso direto aos pods só via `kubectl port-forward`.

## Veja também

- [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose)
- [Subir um único serviço](Subir-um-unico-servico)
