# Para integradores externos

> **Rótulo:** Tutorial (jornada guiada)
> **TL;DR:** Roteiro para quem vai consumir as APIs da Mecânica Hermes de fora (frontend cliente, app móvel, parceiro).
> **Última revisão:** 2026-05-18

## Em 30 minutos você terá conseguido

- Obter um token JWT válido.
- Chamar os endpoints públicos.
- Entender a semântica assíncrona (`202 Accepted`).
- Validar webhooks assinados.

## Roteiro

### 1. Autenticação (10 min)

- [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT) — fluxo OAuth2 Client Credentials.
- [Autorização por scopes](Autorizacao-por-scopes) — quais escopos liberam quais endpoints.
- [Lambda CognitoToken](Lambda-CognitoToken) — fluxo de login para clientes finais (CPF → JWT por e-mail).

### 2. API pública (10 min)

- [API pública](API-publica) — lista de endpoints com links para o Scalar (OpenAPI interativo) de cada serviço.
- [Endpoints de health-check](Endpoints-de-health-check) — `/healthz/live` e `/healthz/ready`.

### 3. Semântica assíncrona (5 min)

Endpoints de mutação (POST/PATCH/DELETE) das 3 APIs retornam **`202 Accepted` imediatamente** — o trabalho é enfileirado e processado por consumers MassTransit. O cliente deve:

1. Receber o `202` e guardar o ID do recurso.
2. **Polar** `GET /<recurso>/{id}` até o estado refletir a mudança (ou consumir webhook, se aplicável).
3. **Não** assumir que a operação terminou só porque a resposta foi 202.

Ver [SAGA com MassTransit](SAGA-com-MassTransit) para entender o pipeline interno.

### 4. Webhooks (5 min)

Se você **recebe** webhooks da Mecânica Hermes (não aplicável hoje, mas planejado):

- Todos os webhooks que **enviamos** são assinados via HMAC-SHA256 no header `X-Signature`.
- Implemente dedup por `event-id`.

Se você **expor** um webhook que vamos chamar (ex.: parceiro):

- Valide HMAC do nosso lado.
- Ver [Webhooks assinados (HMAC)](Webhooks-assinados-HMAC) para o algoritmo.

## Limites e SLAs

_TODO — documentar rate limits do API Gateway e SLOs por endpoint._

## Suporte

Para reportar bug ou solicitar mudança de contrato, abra issue no repositório do serviço correspondente:

- OS → [`api-ordem-servico`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/issues)
- Cadastros → [`api-cadastros`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros/issues)
- Pagamentos → [`api-pagamentos`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos/issues)
