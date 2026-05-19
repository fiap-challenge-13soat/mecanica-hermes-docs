# Runbook — Pagamentos

> **Rótulo:** How-to
> **TL;DR:** Incidentes mais comuns na API Pagamentos. A maioria envolve o Mercado Pago.
> **Última revisão:** 2026-05-18

## Cenários

### Mercado Pago indisponível

**Sintoma:** chamadas `POST /checkout/preferences` falhando, status página `pendenteGeracao` não avança.

**Diagnóstico:**

- Polly está retentando — logs com `RetryAttempt`.
- Status page do MP (https://status.mercadopago.com.br).
- `pagamento.polling.executado` cai a zero? Polling também está falhando.

**Mitigação:**

- Esperar o MP voltar.
- Pagamentos contém "pagamento.PendenteGeracao" por até 30 polls (default) — cobre janela longa.
- Se MP cai por horas, considerar **fallback mão-de-obra** (manual): cancelar pagamento e gerar offline.

### Webhook do MP sempre rejeitado (401 HMAC)

**Diagnóstico:**

- `MERCADO_PAGO__WEBHOOK_SECRET` está configurado?
- Secret bate com o configurado no **painel do Mercado Pago** (Application → Notificações → Webhook URL + secret)?
- O algoritmo HMAC está conforme a documentação do MP?

**Mitigação:**

- Sincronizar secrets entre prod e painel MP.
- O **polling** é backup — mesmo sem webhook, eventualmente confirma.

### Pagamento confirmado mas OS não avança

**Diagnóstico:**

- Mensagem `pagamento.confirmado.v1` foi publicada?
- Foi consumida pela OS? `messaging.consume_attempts` no serviço OS.
- Foi ignorada por idempotência? `ordemdeservico.evento_integracao_ignorado` com `status_atual` != `AguardandoPagamento`.

**Mitigação:**

- Se OS não está em `AguardandoPagamento`, o evento foi ignorado **corretamente** (idempotência). Investigue por que o estado divergiu.
- Se OS está em `AguardandoPagamento` e o evento não chegou: verifique RabbitMQ.

### SAGA do Pagamento travada em `AguardandoConfirmacao`

**Sintoma:** Pagamento não sai de `AguardandoConfirmacao` por horas.

**Diagnóstico:**

- `LimiteVerificacoes` foi atingido? Verifique campo `limite_verificacoes` no documento Mongo.
- Polling continua agendado? Mensagens delayed no Rabbit.

**Mitigação:**

- Atingiu o limite → dispare `Expirar` manualmente (script em `tools/`, ainda não implementado).
- Mensagens delayed sumiram → republicar.

### Polling agressivo demais

**Sintoma:** muitas requisições ao MP (rate limit do MP atingido).

**Diagnóstico:**

- `pagamento.polling.executado` muito alto.
- Multiple pods? Multiplique pelo número de réplicas.

**Mitigação:**

- Aumentar o intervalo de polling em config (default 2 min).
- Reduzir replicas (HPA).

## Diagnóstico geral

Ver [`docs/runbook.md`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/blob/main/docs/runbook.md).

## Veja também

- [Runbook macro](Runbook-macro)
- [Fluxo — Pagamento cancelado e recriado](Fluxo-Pagamento-cancelado-recriado)
- [Fluxo — Pagamento expirado](Fluxo-Pagamento-expirado)
