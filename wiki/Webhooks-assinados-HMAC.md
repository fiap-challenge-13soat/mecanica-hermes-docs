# Webhooks assinados (HMAC)

> **Rótulo:** Explicação
> **TL;DR:** Webhooks públicos (orçamento + Mercado Pago) são validados por HMAC-SHA256 e tem dedup por event-id.
> **Última revisão:** 2026-05-18

## Dois webhooks públicos

| Webhook | Serviço que **recebe** | Quem **chama** |
|---|---|---|
| `POST /api/webhooks/orcamentos/{token}` | Cadastros | Cliente final (browser, ao clicar no e-mail) |
| `POST /api/webhooks/mercadopago` | Pagamentos | Mercado Pago |

## Webhook de orçamento (Cadastros)

### Fluxo

1. OS publica `ordem-de-servico.aguardando-aprovacao.v1`.
2. Consumer em Cadastros gera um `webhook_link` com:
   - `token` aleatório (32 bytes, base64url).
   - `ordem_de_servico_id`.
   - `expira_em` = +7 dias.
3. Cadastros envia e-mail com link: `https://<host>/api/webhooks/orcamentos/{token}?decisao=aprovado`.
4. Cliente clica.
5. Endpoint valida:
   - Token existe e não expirou.
   - HMAC do header `X-Signature` confere com payload + `WEBHOOK__SIGNING_SECRET`.
   - `webhook_event_id` não foi processado antes (dedup).
6. Persiste decisão + publica `orcamento-aprovado-pelo-cliente.v1` (ou `rejeitado`) via Outbox transacional.

### Algoritmo HMAC

```csharp
using var hmac = new HMACSHA256(Encoding.UTF8.GetBytes(secret));
var hash = hmac.ComputeHash(Encoding.UTF8.GetBytes(payload));
var signature = Convert.ToHexString(hash).ToLowerInvariant();
```

Verificação usa `CryptographicOperations.FixedTimeEquals` para evitar timing attacks.

### Idempotência

Tabela `webhook_events`:

```sql
CREATE TABLE webhook_events (
  webhook_event_id text PRIMARY KEY,
  received_at timestamptz NOT NULL DEFAULT NOW()
);
```

INSERT com `ON CONFLICT DO NOTHING`. Se conflito, retorna 200 sem reprocessar.

## Webhook do Mercado Pago (Pagamentos)

### Fluxo

1. Pagamentos cria preferência no MP via API.
2. Cliente paga.
3. MP envia `POST /api/webhooks/mercadopago` com payload tipo `payment.created`/`payment.updated`.
4. Endpoint valida:
   - HMAC do header `X-Signature` contra `MERCADO_PAGO__WEBHOOK_SECRET`.
   - `mp_event_id` (do payload) não processado antes (coleção `mp_webhook_events`).
5. Publica `MercadoPagoCallbackRecebidoRequest` na SAGA.
6. SAGA consulta status real do pagamento no MP (não confia no payload do webhook), e dependendo do status confirma/recusa/expira.

### Defesa em profundidade

O MP **já envia HMAC**. Mas confiamos também em:

- **Dedup por event-id** — webhook pode chegar 2-3 vezes (recovery do MP).
- **Refresh do status real** — mesmo recebendo "approved" no webhook, fazemos `GET /v1/payments/{id}` antes de confirmar.

## Polling como backup

Se o webhook **não chegar** (rede, MP indisponível), o polling agendado pela SAGA descobre o status. Ver [Fluxo — Caminho feliz](Fluxo-Caminho-feliz).

## Segredos

| Variável | Serviço |
|---|---|
| `WEBHOOK__SIGNING_SECRET` | Cadastros (gerado por nós) |
| `MERCADO_PAGO__WEBHOOK_SECRET` | Pagamentos (recebido do painel MP) |

Ambos com **mínimo 32 bytes** (256 bits) e armazenados em Kubernetes Secret (em produção) ou env var (em dev).

## Logs

- Não logamos payload completo (pode conter dados pessoais).
- Logamos: hash do payload (truncado), `event_id`, status do HMAC (ok/falha), `correlation_id`.

## Veja também

- [Idempotência cross-service](Idempotencia-cross-service)
- [Fluxo — Idempotência de webhook](Fluxo-Idempotencia-de-webhook)
- [Modelo de ameaças](Modelo-de-ameacas)
