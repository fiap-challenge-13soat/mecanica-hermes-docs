# Relatórios Allure

> **Rótulo:** Referência
> **TL;DR:** Allure gera relatório rico para a suíte E2E com histórico das 20 últimas execuções. Publicado em GitHub Pages.
> **Última revisão:** 2026-05-18

## URL pública

> https://fiap-challenge-13soat.github.io/mecanica-hermes-tests-e2e/

Atualizado a cada push em `main` do `mecanica-hermes-tests-e2e`.

## Como rodar local

```bash
cd mecanica-hermes-tests-e2e
pip install -r requirements.txt
robot --listener allure_robotframework:allure-results \
      --outputdir results \
      tests/suites/
allure serve allure-results
```

Uma janela do navegador abre com o relatório interativo.

## Anexos

- **Screenshots** — n/a (suíte não tem UI).
- **Logs de cada keyword** — capturados automaticamente.
- **Request/Response HTTP** — anexados aos passos relevantes.
- **Comentar do RabbitMQ Management** se quiser inspecionar fila após falha (TODO automatizar).

## Histórico (Trend)

O Allure consolidado em CI mantém as **20 últimas execuções**, mostrando trend de:

- Pass / fail / broken.
- Duração por suíte.
- Flaky tests (passa em algumas runs, falha em outras).

## Artifacts no Actions

Para baixar relatório de uma execução específica (PR ou rerun), abra a Action correspondente:

```
Actions → e2e-tests → <run> → Artifacts
  - allure-report (retention 30 dias)
  - robot-results (retention 30 dias)
```

## Veja também

- [Testes E2E](Testes-E2E)
- [Rodar a suíte E2E](Rodar-a-suite-E2E)
