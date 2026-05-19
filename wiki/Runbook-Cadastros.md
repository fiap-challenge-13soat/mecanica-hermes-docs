# Runbook — Cadastros

> **Rótulo:** How-to
> **TL;DR:** Incidentes mais comuns na API Cadastros e como diagnosticá-los.
> **Última revisão:** 2026-05-18

## Cenários

### Cliente reclama que não recebeu o e-mail de orçamento

**Diagnóstico:**

1. OS chegou em `AguardandoAprovacao`? Verifique pela API.
2. Evento `ordem-de-servico.aguardando-aprovacao.v1` foi consumido? Cheque `messaging.consume_attempts` para esse `message_type`.
3. `webhook_link` foi gerado? Inspecione tabela `webhook_links` no Postgres.
4. E-mail foi enviado? Logs do pod — procure `MailKit` + `email_sent`.

**Mitigações:**

- E-mail no spam (acionar usuário).
- SMTP down (logs com `SmtpException`).
- `EmailSender__Enabled=false` mal configurado.

### Webhook do cliente sempre 401 (HMAC)

**Diagnóstico:**

1. `WEBHOOK__SIGNING_SECRET` está configurado e não-vazio?
2. Frontend está calculando HMAC com o mesmo secret?
3. O secret é o mesmo nos 2+ pods (replicas)?

**Mitigação:**

- Rotation de secret precisa ser **coordenada** com o gerador de links (frontend admin).
- Use o keyword Robot `Calcular HMAC SHA256` para reproduzir localmente.

### `GET /api/clientes/{id}` retorna 401 vindo de Pagamentos

**Diagnóstico:**

1. `M2M__*` env vars estão configuradas em Pagamentos?
2. O scope `servico-scope` está sendo solicitado no Client Credentials?
3. O `client_id` tem permissão pra esse scope no Cognito?

**Mitigação:**

- Verifique o secret consolidado `mechermes-<env>-cognito-client-secret`.
- Logs do `M2MAuthorizationDelegatingHandler` mostram o refresh do token.

### Soft delete deixou índice único corrompido (raro)

**Sintoma:** INSERT de cliente com CPF "já usado" mas a UI mostra como deletado.

**Diagnóstico:**

```sql
SELECT id, cpf, is_deleted FROM clientes WHERE cpf = '...';
```

Deve retornar 1 linha com `is_deleted = true`. Se o índice está errado, você vá 2 linhas ou conflict.

**Mitigação:**

- Recriar o índice parcial via migration (não em hot-fix manual).

## Diagnóstico geral

Ver [`docs/runbook.md`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/blob/main/docs/runbook.md).

## Veja também

- [Runbook macro](Runbook-macro)
- [Webhooks assinados (HMAC)](Webhooks-assinados-HMAC)
- [M2M Client Credentials](M2M-Client-Credentials)
