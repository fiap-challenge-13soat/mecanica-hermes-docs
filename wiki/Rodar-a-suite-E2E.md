# Rodar a suíte E2E

> **Rótulo:** How-to
> **TL;DR:** Executar as 8 suítes BDD em Robot Framework contra a stack Docker.
> **Última revisão:** 2026-05-18

## Pré-requisitos

- Python 3.11+
- A stack rodando — ver [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose).
- (Opcional) [Allure CLI](https://github.com/allure-framework/allure2/releases) para relatórios ricos.

## Passo a passo

### 1. Instalar dependências Python

```bash
cd mecanica-hermes-tests-e2e
pip install -r requirements.txt
```

### 2. Rodar todas as suítes

```bash
robot --outputdir results tests/suites/
```

Relatório nativo do Robot em `results/report.html`.

### 3. Rodar apenas smoke

```bash
robot --include smoke --outputdir results tests/suites/
```

### 4. Rodar uma suíte específica

```bash
robot --outputdir results tests/suites/01__caminho_feliz.robot
```

### 5. Com Allure (recomendado)

```bash
robot --listener allure_robotframework:allure-results \
      --outputdir results \
      tests/suites/
allure serve allure-results
```

## Suítes disponíveis

| Suíte | Fluxo | Tags |
|---|---|---|
| `01__caminho_feliz` | OS feliz até `Entregue` | `caminho-feliz` `smoke` |
| `02__pagamento_cancelado_recriado` | Pagamento recusado → OS reverte → 2º pagamento aprovado | `pagamento-recusado` `resiliencia` |
| `03__orcamento_rejeitado` | Cliente rejeita orçamento via webhook | `orcamento-rejeitado` |
| `04__cancelamento_em_execucao` | Operador cancela em `EmExecucao` | `cancelamento` |
| `05__pagamento_expirado` | MP expira o pagamento por timeout | `pagamento-expirado` `resiliencia` |
| `06__webhook_idempotencia` | Webhook duplicado não avança OS duas vezes | `webhook-idempotencia` |
| `07__saga_timeout_protection` | Timeout sentinel da SAGA | `saga-timeout` |
| `08__cancelamento_em_aguardando_pagamento` | Cancelar OS em `AguardandoPagamento` cascateia para `Pagamento.Recusado` | `cancelamento` `pagamento-pendente` `cross-service` |

## CI / Allure online

A cada push em `main`, o workflow `e2e-tests.yml` roda as 8 suítes e publica o Allure em **GitHub Pages** com histórico das 20 últimas execuções:

> https://fiap-challenge-13soat.github.io/mecanica-hermes-tests-e2e/

## Páginas relacionadas

- [Testes E2E](Testes-E2E) — estratégia e convenções BDD
- [Relatórios Allure](Relatorios-Allure)
- [Fluxo — Caminho feliz](Fluxo-Caminho-feliz) e demais fluxos
