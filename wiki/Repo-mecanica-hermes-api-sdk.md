# Repo: `mecanica-hermes-api-sdk`

> **Rótulo:** Referência
> **Papel em uma frase:** SDK compartilhado em 6 pacotes NuGet publicados em GitHub Packages, consumido pelos 3 serviços .NET.
> **Stack:** .NET 10, NuGet, GitHub Packages
> **Categoria:** Plataforma
> **Última revisão:** 2026-05-19

## Resumo

Repositório de **biblioteca**. Contém 6 pacotes que encapsulam código compartilhado entre OS, Cadastros e Pagamentos:

| Pacote | Dependências |
|---|---|
| `Mecanica.Hermes.Shared.Core` | (nenhuma) |
| `Mecanica.Hermes.Shared.Application` | MediatR |
| `Mecanica.Hermes.Shared.AspNetCore` | ASP.NET Core, FluentValidation, Serilog |
| `Mecanica.Hermes.Shared.Observability` | OpenTelemetry |
| `Mecanica.Hermes.Shared.MassTransit` (Tier 2) | MassTransit |
| `Mecanica.Hermes.Contracts` | MassTransit.Abstractions — **8 eventos `.v1`** |

Publicação em **GitHub Packages** na org `fiap-challenge-13soat`. Consumo via `nuget.config` + PAT classic com `read:packages`.

## Versão atual

**v1.1.0** — adicionou `OrdemDeServicoCanceladaEvent` (`ordem-de-servico.cancelada.v1`) para o publisher/consumer cross-service entre OS e Pagamentos. Todos os consumidores (OS, Cadastros, Pagamentos) já estão atualizados.

## Como se relaciona com o resto

- **Consumido por:** os 3 serviços .NET.
- **Não depende de:** nenhum repositório do ecossistema.

## Pontos-chave

- **Versão única** para todos os 6 pacotes (`Directory.Build.props`).
- **SemVer estrito** — bump MAJOR em qualquer rename/remoção de tipo público ou de `EntityName` de contrato.
- **Tier 2 (skeleton)** em v1.0: `Shared.MassTransit`.
- **Smoke tests** — `IntegrationEventNamesTests` garante que `EntityName`s não mudaram.
- **Source Link** habilitado — step-into no debugger do consumidor.

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk/blob/main/README.md) | Visão geral dos pacotes |
| [CLAUDE.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk/blob/main/CLAUDE.md) | Arquitetura |
| [docs/architecture.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk/blob/main/docs/architecture.md) | Pirâmide dos 6 pacotes |
| [docs/versioning.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk/blob/main/docs/versioning.md) | Política SemVer |
| [docs/consuming.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk/blob/main/docs/consuming.md) | Como configurar nos consumidores |
| [docs/ci-cd.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk/blob/main/docs/ci-cd.md) | Workflows do GitHub Actions |

## Comandos essenciais

| Comando | O que faz |
|---|---|
| `just build` | Build Debug |
| `just test` | Smoke tests |
| `just pack` | Gera os 6 `.nupkg` em `./artifacts` |
| `just format-check` | Lint (igual ao CI) |

## Veja também

- [SDK — Visão dos 6 pacotes](SDK-Visao-dos-6-pacotes)
- [SDK — Como consumir](SDK-Como-consumir)
- [SDK — Versionamento](SDK-Versionamento)
- [Catálogo de eventos](Catalogo-de-eventos)
