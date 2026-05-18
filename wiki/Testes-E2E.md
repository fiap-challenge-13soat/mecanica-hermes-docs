# Testes E2E

> **Rótulo:** Explicação
> **TL;DR:** Suíte BDD em Robot Framework (Python 3.11) valida 8 fluxos cross-service contra o ecossistema rodando em Docker Compose. Allure em GitHub Pages.
> **Última revisão:** 2026-05-18

## Repositório

`mecanica-hermes-tests-e2e` (ver [Repo: mecanica-hermes-tests-e2e](Repo-mecanica-hermes-tests-e2e)).

## 8 suítes

Espelhadas em páginas de fluxo na Wiki:

| Arquivo | Fluxo Wiki |
|---|---|
| `01__caminho_feliz` | [Fluxo — Caminho feliz](Fluxo-Caminho-feliz) |
| `02__pagamento_cancelado_recriado` | [Fluxo — Pagamento cancelado e recriado](Fluxo-Pagamento-cancelado-recriado) |
| `03__orcamento_rejeitado` | [Fluxo — Orçamento rejeitado](Fluxo-Orcamento-rejeitado) |
| `04__cancelamento_em_execucao` | [Fluxo — Cancelamento em execução](Fluxo-Cancelamento-em-execucao) |
| `05__pagamento_expirado` | [Fluxo — Pagamento expirado](Fluxo-Pagamento-expirado) |
| `06__webhook_idempotencia` | [Fluxo — Idempotência de webhook](Fluxo-Idempotencia-de-webhook) |
| `07__saga_timeout_protection` | [Fluxo — Proteção por timeout da SAGA](Fluxo-Protecao-por-timeout-da-SAGA) |
| `08__cancelamento_em_aguardando_pagamento` | [Fluxo — Cancelamento em aguardando pagamento](Fluxo-Cancelamento-em-aguardando-pagamento) |

## BDD Gherkin em Robot

Keywords prefixadas em português:

```robot
*** Test Cases ***
Caminho feliz da ordem de servico
    [Tags]    caminho-feliz    smoke
    Given que existe um cliente cadastrado
    When uma OS é criada
    And o orcamento é aprovado pelo cliente via webhook
    And o pagamento é confirmado via Mercado Pago
    Then a OS deve estar no estado "Entregue"
```

Keywords resolvem em `tests/resources/keywords/*.resource`.

## Ambiente

A suíte não sobe os serviços via `dotnet run` — ela usa as imagens **publicadas no Docker Hub**:

```yaml
services:
  api-os:
    image: mechermes/mecanica-hermes-api-ordem-servico:${OS_IMAGE_TAG:-latest}
```

Tags re-publicadas em cada merge em `main` dos repos de serviço.

Mercado Pago **não** é chamado real — é mockado pelo **WireMock** com mappings versionados em `tests/resources/fixtures/wiremock/mappings/`.

## Polling determinístico

Em vez de `sleep`, usamos `Wait Until Keyword Succeeds`:

```robot
Wait Until Keyword Succeeds    60s    3s
...    Verificar status da OS    ${id}    Entregue
```

Tenta a cada 3s por até 60s. Reduz flakiness mantendo testes rápidos no caminho feliz.

## CI

Workflow `.github/workflows/e2e-tests.yml`:

- Trigger: push `main`, push `feature/**`, PR para `main`, `workflow_dispatch`.
- Sobe Compose com `--wait` (aguarda healthchecks).
- Executa todas as 8 suítes.
- Gera Allure report.
- Publica em GitHub Pages **apenas em push para `main`**.

## Veja também

- [Rodar a suíte E2E](Rodar-a-suite-E2E)
- [Relatórios Allure](Relatorios-Allure)
- [Repo: mecanica-hermes-tests-e2e](Repo-mecanica-hermes-tests-e2e)
