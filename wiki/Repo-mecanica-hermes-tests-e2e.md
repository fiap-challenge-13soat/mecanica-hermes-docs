# Repo: `mecanica-hermes-tests-e2e`

> **Rótulo:** Referência
> **Papel em uma frase:** Suíte de testes ponta-a-ponta em Robot Framework BDD, cobrindo 8 fluxos cross-service em Docker Compose. Relatório Allure publicado em GitHub Pages.
> **Stack:** Python 3.11, Robot Framework 7.2, Allure, Docker Compose, WireMock
> **Categoria:** Qualidade
> **Última revisão:** 2026-05-18

## Resumo

**Não é .NET.** É um projeto Python que sobe o ecossistema inteiro localmente (via `docker-compose`) e roda 8 suítes BDD validando os fluxos cross-service. É a forma canônica de rodar tudo localmente.

## Como se relaciona com o resto

- **Consome:** imagens Docker `mechermes/*` (publicadas pelos repos de serviço no Docker Hub).
- **Não consome:** SDK (não é .NET).
- **Testa:** OS + Cadastros + Pagamentos integrados, com WireMock simulando o Mercado Pago.

## Pontos-chave

- **8 suítes** cobrem caminho feliz + 7 cenários alternativos (rejeição, recusa, expiração, cancelamento, idempotência, timeout).
- **WireMock** com mappings versionados, montados como bind mount.
- **Allure** publicado em GitHub Pages com histórico das 20 últimas execuções.
- **CI** dispara em `push main` e PRs.
- **HMAC** computado nos testes para validar webhooks.
- **Polling determinístico** com `Wait Until Keyword Succeeds`.

## Suítes

| Arquivo | Cenário |
|---|---|
| `01__caminho_feliz.robot` | Recebida → ... → Entregue |
| `02__pagamento_cancelado_recriado.robot` | Pagamento recusado → OS reverte → 2º pag aprovado |
| `03__orcamento_rejeitado.robot` | Cliente rejeita orçamento |
| `04__cancelamento_em_execucao.robot` | Operador cancela em `EmExecucao` |
| `05__pagamento_expirado.robot` | Pagamento expira por timeout |
| `06__webhook_idempotencia.robot` | Webhook duplicado não avança 2× |
| `07__saga_timeout_protection.robot` | Timeout sentinel da SAGA |
| `08__cancelamento_em_aguardando_pagamento.robot` | Cancelar em AwP cascateia em `Pagamento.Recusado` |

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e/blob/main/README.md) | Visão geral, comandos |
| [CLAUDE.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e/blob/main/CLAUDE.md) | Convenções BDD + estrutura |
| [docs/improvement-plan-2026-05-10.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e/blob/main/docs/improvement-plan-2026-05-10.md) | Backlog |

## Comandos essenciais

```bash
pip install -r requirements.txt
docker compose -f docker-compose/docker-compose.yaml -f docker-compose/docker-compose.e2e.yaml up -d --wait
robot --outputdir results tests/suites/
allure serve allure-results  # opcional
```

## Allure online

> https://fiap-challenge-13soat.github.io/mecanica-hermes-tests-e2e/

## Veja também

- [Testes E2E](Testes-E2E)
- [Rodar a suíte E2E](Rodar-a-suite-E2E)
- [Relatórios Allure](Relatorios-Allure)
- [Fluxos de negócio](Fluxo-Caminho-feliz) (8 páginas, uma por suíte)
