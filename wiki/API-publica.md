# API pública

> **Rótulo:** Referência
> **TL;DR:** Lista consolidada dos endpoints HTTP das 3 APIs. Para schema interativo, abra o Scalar de cada serviço.
> **Última revisão:** 2026-05-18

## Scalar (OpenAPI interativo)

Em dev local:

- OS → http://localhost:5016/scalar (ou `:8081/scalar` via Compose)
- Cadastros → http://localhost:8082/scalar (Compose)
- Pagamentos → http://localhost:5017/scalar (ou `:8083/scalar` via Compose)

Em produção: bloqueado externamente; acesse via `kubectl port-forward`.

## Endpoints — API Ordem de Serviço

Base: `/api/ordens-de-servico`

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `POST` | `/` | admin | Criar OS (202) |
| `GET` | `/{id}` | client/admin | Buscar OS |
| `GET` | `/` | client/admin | Listar OS (paginado) |
| `GET` | `/status/{status}` | client/admin | Listar OS por status |
| `POST` | `/{id}/produtos` | admin | Adicionar produto (202) |
| `DELETE` | `/{id}/produtos/{produtoId}` | admin | Remover produto (202) |
| `POST` | `/{id}/servicos` | admin | Adicionar serviço (202) |
| `DELETE` | `/{id}/servicos/{servicoId}` | admin | Remover serviço (202) |
| `PATCH` | `/{id}/avancar` | admin | Avançar etapa (202) |
| `PATCH` | `/{id}/reverter` | admin | Reverter etapa (202) |
| `PATCH` | `/{id}/cancelar` | admin | Cancelar (202) |

## Endpoints — API Cadastros

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `POST` | `/api/clientes` | admin | Criar cliente |
| `GET` | `/api/clientes/{id}` | admin / M2M | Buscar cliente |
| `GET` | `/api/clientes` | admin | Listar |
| `PATCH` | `/api/clientes/{id}` | admin | Atualizar |
| `DELETE` | `/api/clientes/{id}` | admin | Soft delete |
| `POST` | `/api/produtos` | admin | Criar produto |
| `GET` | `/api/produtos/{id}` | admin | Buscar produto |
| `POST` | `/api/webhooks/orcamentos/{token}` | público + HMAC | Aprovar/rejeitar orçamento |

## Endpoints — API Pagamentos

| Método | Rota | Auth | Descrição |
|---|---|---|---|
| `POST` | `/api/pagamentos` | admin | Solicitar geração (202) |
| `GET` | `/api/pagamentos/{id}` | client/admin | Buscar pagamento |
| `GET` | `/api/pagamentos/{id}/status` | admin | Forçar consulta MP (debug) |
| `POST` | `/api/webhooks/mercadopago` | público + HMAC | Callback do Mercado Pago |

## Endpoints — Lambda CognitoToken

Rota via API Gateway: `POST /oauth2/token` (não-autenticado).

Ver [Lambda CognitoToken](Lambda-CognitoToken).

## Health-checks

Ver [Endpoints de health-check](Endpoints-de-health-check).

## Semântica assíncrona

**Endpoints de mutação retornam `202 Accepted`** — o trabalho é enfileirado, o cliente deve polar para ver o resultado. Ver [Para integradores externos](Para-integradores-externos).

## Veja também

- [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT)
- [Autorização por scopes](Autorizacao-por-scopes)
- [Catálogo de portas locais](Catalogo-de-portas-locais)
