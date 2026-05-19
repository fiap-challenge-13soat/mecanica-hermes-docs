# Comandos `just`

> **Rótulo:** Referência
> **TL;DR:** Os 3 serviços .NET e o SDK usam [`just`](https://github.com/casey/just) para padronizar comandos comuns. Tabela cruzada repo × receita.
> **Última revisão:** 2026-05-18

## Matriz de receitas

| Receita | OS | Cadastros | Pagamentos | SDK |
|---|---|---|---|---|
| `just` (lista) | ✅ | ✅ | ✅ | ✅ |
| `just build` | ✅ | ✅ | ✅ | ✅ |
| `just test` | ✅ | ✅ | ✅ | ✅ |
| `just test-unit` | ✅ | ✅ | ✅ | n/a |
| `just test-coverage` | ✅ | ✅ | ✅ | n/a |
| `just run` | ✅ | ✅ | ✅ | n/a |
| `just format` | ✅ | ✅ | ✅ | ✅ |
| `just format-check` | ✅ | ✅ | ✅ | ✅ |
| `just pack` | n/a | n/a | n/a | ✅ |
| `just pack-version <v>` | n/a | n/a | n/a | ✅ |
| `just clean` | ✅ | ✅ | ✅ | ✅ |
| `just migrate-add <Name>` | ✅ | ✅ | n/a | n/a |
| `just migrate-update` | ✅ | ✅ | n/a | n/a |

## Instalar `just`

```bash
# macOS
brew install just

# Windows (Scoop)
scoop install just

# Linux
curl --proto '=https' --tlsv1.2 -sSf https://just.systems/install.sh | bash
```

## Padrão

Todos os justfiles seguem o mesmo esqueleto:

```text
_default:
  @just --list

build:
  dotnet build -c Release

test:
  dotnet test

run:
  dotnet run --project src/Mecanica.Hermes.<svc>.Api
```

Isso significa: você aprende `just build` uma vez e funciona em qualquer repo.

## Veja também

- [Subir um único serviço](Subir-um-unico-servico)
- [Como contribuir](Como-contribuir)
