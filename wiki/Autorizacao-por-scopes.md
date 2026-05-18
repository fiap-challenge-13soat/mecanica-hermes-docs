# Autorização por scopes

> **Rótulo:** Referência
> **TL;DR:** Cada endpoint exige um **scope** do JWT. Políticas declarativas são aplicadas via `.RequireAuthorization(...)`.
> **Última revisão:** 2026-05-18

## Políticas disponíveis

Definidas em `AuthPolicies.cs` de cada API:

| Política | Scope exigido | Onde é aplicada |
|---|---|---|
| `OnlyAdminScope` | `mechermes/admin` | Endpoints de mutação de OS e Pagamentos |
| `OnlyClientScope` | `mechermes/client` | (raro) endpoints de auto-consulta do cliente |
| `AdminOrClientScope` | qualquer um | Endpoints de leitura comuns |
| `AdminOrM2MScope` (Cadastros) | `mechermes/admin` **ou** `servico-scope` | `GET /api/clientes/{id}` chamado por Pagamentos |
| `AllowServicoOrAdminScope` (Cadastros) | `servico-scope` **ou** `mechermes/admin` | Mesmo grupo, naming histórico |

## Mapeamento de endpoints (resumo)

### API Ordem de Serviço (`/api/ordens-de-servico`)

| Endpoint | Política |
|---|---|
| `POST /` | `OnlyAdminScope` |
| `GET /{id}` | `AdminOrClientScope` |
| `GET /` (listar) | `AdminOrClientScope` |
| `POST /{id}/produtos` | `OnlyAdminScope` |
| `PATCH /{id}/avancar` | `OnlyAdminScope` |
| `PATCH /{id}/cancelar` | `OnlyAdminScope` |

### API Cadastros (`/api/clientes`, `/api/produtos`)

| Endpoint | Política |
|---|---|
| `POST /clientes` | `OnlyAdminScope` |
| `GET /clientes/{id}` | `AdminOrM2MScope` (chamado por Pagamentos) |
| `POST /webhooks/orcamentos/{token}` | **público** + HMAC |

### API Pagamentos (`/api/pagamentos`)

| Endpoint | Política |
|---|---|
| `POST /` | `OnlyAdminScope` |
| `GET /{id}` | `AdminOrClientScope` |
| `POST /webhooks/mercadopago` | **público** + HMAC |

## Como aplicar

```csharp
endpoints.MapPost("/{id:guid}/produtos", AddProduto)
    .RequireAuthorization(AuthPolicies.OnlyAdminScope);
```

No Development, todas as políticas são **automaticamente satisfeitas** pelo `DevelopmentAuthenticationMiddleware`.

## Por que separar `admin` e `client`

- **admin** — atendente/operador. Pode tudo.
- **client** — cliente final autenticado via CPF. Só pode consultar a própria OS (filtro feito no handler, não no scope).

Em v1.1 a regra de cliente só ver a própria OS depende de um claim adicional `cpf` no token — a Lambda já inclui isso.

## Veja também

- [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT)
- [M2M Client Credentials](M2M-Client-Credentials)
- [API pública](API-publica)
