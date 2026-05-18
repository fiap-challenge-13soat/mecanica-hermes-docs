# Modelo de ameaças

> **Rótulo:** Explicação
> **TL;DR:** Mapeamento honesto das principais ameaças ao sistema, suas mitigações atuais e débitos de segurança conhecidos.
> **Última revisão:** 2026-05-18

## Escopo

Projeto acadêmico (Tech Challenge). Modelo de ameaças **proporcional** ao escopo — cobrimos vetores externos diretos. Defesa em profundidade ainda é incipiente para muitos cenários.

## Vetores principais

### 1. Acesso não-autorizado às APIs

**Risco:** atacante chama endpoints sem autenticação.

**Mitigação:**

- API Gateway com Cognito Authorizer rejeita JWT inválido antes de chegar ao EKS.
- APIs internas também validam JWT (defesa em profundidade).
- Em `Development`/`Testing`, o bypass está ativo — **não usar em produção**.

**Débito:** API interna no EKS aceitaria conexão direta de outro pod no cluster sem JWT válido se ASPNETCORE_ENVIRONMENT estiver mal configurado. NetworkPolicy do K8s mitiga, mas não está hoje.

### 2. Token JWT roubado

**Risco:** atacante captura JWT válido (man-in-the-middle, log vazado, browser dev tools).

**Mitigação:**

- TLS obrigatório em produção (API Gateway).
- Não logamos tokens.
- `expires_in` de 1h limita janela.

**Débito:** sem revogação ativa de tokens (Cognito User Pool tem `revoke` mas não usamos).

### 3. Webhook forjado

**Risco:** atacante envia POST falso para `/api/webhooks/...` simulando aprovação de orçamento ou confirmação de pagamento.

**Mitigação:**

- HMAC-SHA256 em ambos os webhooks ([Webhooks assinados (HMAC)](Webhooks-assinados-HMAC)).
- `FixedTimeEquals` evita timing attacks.
- Token de orçamento expira em 7 dias.
- Pagamento **não confia só no webhook do MP** — sempre faz `GET /v1/payments/{id}` para confirmar.

**Débito:** secret está em env var; rotation manual.

### 4. Replay de webhook

**Risco:** atacante captura webhook válido e reenvia.

**Mitigação:**

- Dedup por `webhook_event_id` (Cadastros) e `mp_event_id` (Pagamentos).
- HMAC inclui payload — alterar dado força recomputar hash.

### 5. SQL Injection / NoSQL Injection

**Risco:** input não sanitizado.

**Mitigação:**

- EF Core (parametrized queries) e MongoDB.Driver (sem string concatenation).
- FluentValidation no boundary (DTOs).

### 6. Secret leakage no Git

**Risco:** developer commita `.env` com credencial.

**Mitigação:**

- `.gitignore` lista `launchSettings.json`.
- Secrets em AWS Secrets Manager (em prod).
- GitHub secret scanning ativo no repo (alerts).

**Débito:** sem pre-commit hook bloqueante (TODO).

### 7. Container compromise

**Risco:** imagem com vulnerabilidade conhecida.

**Mitigação:**

- Imagens base oficiais (`mcr.microsoft.com/dotnet/aspnet:10`, `mongo:7`, `postgres:16`).
- Dockerfile multi-stage — imagem final não tem SDK.

**Débito:** sem scan automático de CVEs (TODO).

### 8. RDS exposto

**Risco:** banco acessível da internet.

**Mitigação:**

- RDS criado **sem public access** (`publicly_accessible = false`).
- Security Group permite só EKS + Lambda.

### 9. DLQ envenenada

**Risco:** mensagem malicosa cai em DLQ e alguém replay sem revisar.

**Mitigação:**

- Runbook de incidentes exige inspeção antes de replay.
- Esquemas de eventos validados em `IntegrationEventNamesTests`.

### 10. DoS

**Risco:** atacante envia muitas requests.

**Mitigação:**

- API Gateway tem rate limiting padrão.
- HPA escala pods em CPU.

**Débito:** sem WAF; sem rate limit específico por scope/IP.

## Débitos conhecidos (resumo)

- NetworkPolicy do Kubernetes.
- Rotation automática de webhook secrets.
- Scan de CVE nas imagens.
- WAF + rate limit específico.
- Revogação ativa de JWT.

## Veja também

- [Autenticação Cognito + JWT](Autenticacao-Cognito-JWT)
- [Webhooks assinados (HMAC)](Webhooks-assinados-HMAC)
- [Cluster Kubernetes](Cluster-Kubernetes)
