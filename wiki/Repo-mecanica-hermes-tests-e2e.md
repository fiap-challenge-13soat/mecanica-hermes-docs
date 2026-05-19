# Repo: `mecanica-hermes-tests-e2e`

> **Rótulo:** Referência
> **Papel em uma frase:** Suíte de testes ponta-a-ponta em Robot Framework BDD, cobrindo 8 fluxos cross-service em Docker Compose. Relatório Allure publicado em GitHub Pages.
> **Stack:** Python 3.11, Robot Framework 7.2, Allure CLI, Docker Compose, WireMock
> **Categoria:** Qualidade
> **Última revisão:** 2026-05-19

## Resumo

**Não é .NET.** É um projeto Python que sobe o ecossistema inteiro localmente (via `docker-compose`) e roda 8 suítes BDD validando os fluxos cross-service. É a forma canônica de rodar tudo localmente — os READMEs das 3 APIs apontam para este repo como referência da stack completa.

## Como se relaciona com o resto

- **Consome:** imagens Docker `mechermes/*` (publicadas pelos repos de serviço no Docker Hub). Tags parametrizadas via `OS_IMAGE_TAG`, `CADASTROS_IMAGE_TAG`, `PAGAMENTOS_IMAGE_TAG`.
- **Não consome:** SDK (não é .NET).
- **Testa:** OS + Cadastros + Pagamentos integrados, com WireMock simulando o Mercado Pago.

## Pontos-chave

- **8 suítes** cobrem caminho feliz + 7 cenários alternativos (rejeição, recusa, expiração, cancelamento em execução, cancelamento em AwP, idempotência, timeout da SAGA).
- **Imagem custom do RabbitMQ** buildada in-place com o plugin `rabbitmq_delayed_message_exchange` (necessário para o polling da SAGA da OS).
- **WireMock** com mappings versionados, montados como bind mount.
- **Allure CLI** (não mais a action `simple-elf/allure-report-action`) gera o relatório; publicado em GitHub Pages com histórico das 20 últimas execuções.
- **CI** dispara em `push main` e PRs, com `--pull always` para garantir que as imagens `:latest` sejam re-puxadas a cada execução.
- **HMAC** computado nos testes para validar webhooks.
- **Polling determinístico** com `Wait Until Keyword Succeeds`.
- **SDK v1.1.0** já refletido nas APIs testadas — suíte 08 valida o novo fluxo cross-service de cancelamento.

## Suítes

| Arquivo | Cenário | Tag |
|---|---|---|
| `01__caminho_feliz.robot` | Recebida → ... → Entregue | `caminho-feliz` `smoke` |
| `02__pagamento_cancelado_recriado.robot` | Pagamento recusado → OS reverte → 2º pag aprovado | `pagamento-recusado` `resiliencia` |
| `03__orcamento_rejeitado.robot` | Cliente rejeita orçamento | `orcamento-rejeitado` |
| `04__cancelamento_em_execucao.robot` | Operador cancela em `EmExecucao` | `cancelamento` |
| `05__pagamento_expirado.robot` | Pagamento expira por timeout | `pagamento-expirado` `resiliencia` |
| `06__webhook_idempotencia.robot` | Webhook duplicado não avança 2× | `webhook-idempotencia` |
| `07__saga_timeout_protection.robot` | Timeout sentinel da SAGA | `saga-timeout` |
| `08__cancelamento_em_aguardando_pagamento.robot` | Cancelar em AwP cascateia em `Pagamento.Recusado` via consumer cross-service | `cancelamento` `pagamento-pendente` `cross-service` |

## Onde aprofundar

| Documento | Para quê |
|---|---|
| [README.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e/blob/main/README.md) | Visão geral, comandos, docker-compose |
| [CLAUDE.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e/blob/main/CLAUDE.md) | Convenções BDD + Ecosystem Context |
| [docs/improvement-plan-2026-05-10.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e/blob/main/docs/improvement-plan-2026-05-10.md) | Plano de melhorias da sessão de bump SDK v1.1.0 + parametrização de tags Docker |

## Comandos essenciais

```bash
pip install -r requirements.txt
docker compose \
  -f docker-compose/docker-compose.yaml \
  -f docker-compose/docker-compose.e2e.yaml \
  up -d --wait --build --pull always
robot --outputdir results tests/suites/
allure serve allure-results  # opcional, requer Allure CLI
```

## Serviços no Docker Compose E2E

| Serviço | Porta local |
|---|---|
| `api-os` | 8081 |
| `api-cadastros` | 8082 |
| `api-pagamentos` | 8083 |
| `wiremock` (mock Mercado Pago) | 8090 |
| `postgres` (OS + Cadastros) | 5432 |
| `mongo` (Pagamentos SAGA + Outbox) | 27017 |
| `rabbitmq` + Management UI | 5672 / 15672 |

## Allure online

> https://fiap-challenge-13soat.github.io/mecanica-hermes-tests-e2e/

## Veja também

- [Testes E2E](Testes-E2E)
- [Rodar a suíte E2E](Rodar-a-suite-E2E)
- [Relatórios Allure](Relatorios-Allure)
- [Fluxos de negócio](Fluxo-Caminho-feliz) (8 páginas, uma por suíte)
