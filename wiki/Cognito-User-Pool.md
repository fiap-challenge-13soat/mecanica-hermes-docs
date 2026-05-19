# Cognito

> **Rótulo:** Referência
> **TL;DR:** AWS Cognito User Pool emite JWTs via OAuth2 Client Credentials para todos os atores do sistema.
> **Última revisão:** 2026-05-18

## Provisionado pelo módulo `cognito/`

| Recurso | Nome |
|---|---|
| User Pool | `mechermes-<env>-userpool` |
| App Client | `mechermes-<env>-client` |
| Resource Server | `mechermes` |
| Domínio | `mechermes-<env>.auth.us-east-1.amazoncognito.com` |
| Secret (Secrets Manager) | `mechermes-<env>-cognito-client-secret` |

## Resource Server e Scopes

O Resource Server `mechermes` define os scopes que existem:

| Scope identifier | Nome amigável |
|---|---|
| `mechermes/admin` | Admin (atendente, operador) |
| `mechermes/client` | Cliente final (consulta a própria OS) |
| `servico-scope` | M2M entre serviços (Pagamentos → Cadastros) |

## App Client

Fluxo único habilitado: **Client Credentials**. Sem login interativo. Sem sign-up via Cognito.

O `client_id` e `client_secret` estão em **Secrets Manager** — nenhum serviço tem hardcoded.

## Como obter token

```bash
DOMAIN=mechermes-prd.auth.us-east-1.amazoncognito.com
curl -X POST "https://${DOMAIN}/oauth2/token" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -d "grant_type=client_credentials&client_id=...&client_secret=...&scope=mechermes/admin"
```

Resposta:

```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIs...",
  "expires_in": 3600,
  "token_type": "Bearer"
}
```

## Cognito como Authorizer

O API Gateway valida o JWT via JWKS público do Cognito (`/.well-known/jwks.json`). Sem chamada extra ao Cognito por request — JWKS é cacheado.

## Secrets armazenados

O secret `mechermes-<env>-cognito-client-secret` (auto-criado pelo deploy da Lambda) contém:

```json
{
  "client_id": "...",
  "client_secret": "...",
  "domain": "mechermes-prd.auth.us-east-1.amazoncognito.com",
  "scope": "mechermes/client",
  "db_password": "<senha-do-rds>"
}
```

A senha do RDS é incluída para a Lambda não precisar de dois secrets.

## Veja também

- [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT)
- [Autorização por scopes](Autorizacao-por-scopes)
- [Lambda CognitoToken](Lambda-CognitoToken)
- [M2M Client Credentials](M2M-Client-Credentials)
