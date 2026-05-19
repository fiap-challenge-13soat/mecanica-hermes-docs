# Runbook — Ordem de Serviço

> **Rótulo:** How-to
> **TL;DR:** Incidentes mais comuns na API Ordem de Serviço e como diagnosticá-los.
> **Última revisão:** 2026-05-18

## Inspeção básica

```bash
# Pods
kubectl get pods -n mecanica-hermes-prd -l app=api-ordem-servico

# Logs
kubectl logs -n mecanica-hermes-prd -l app=api-ordem-servico --tail 200

# Eventos
kubectl get events -n mecanica-hermes-prd --sort-by=.lastTimestamp
```

## Cenários

### OS travada em estado intermediário

**Sintoma:** OS criada há horas, ainda em `AguardandoAprovacao` ou `AguardandoPagamento`.

**Diagnóstico:**

1. Verifique se o evento esperado foi publicado:
   - `AguardandoAprovacao` espera `orcamento-aprovado-pelo-cliente.v1` ou `rejeitado`.
   - `AguardandoPagamento` espera `pagamento.confirmado.v1` ou `recusado.v1`.
2. Verifique no RabbitMQ Management (port-forward + UI) se há mensagem na fila de origem.
3. Verifique métrica `messaging.consume_faults` para esse `message_type`.
4. Verifique métrica `ordemdeservico.evento_integracao_ignorado` — pode estar sendo descartado.

**Mitigação:**

- Se o evento não foi publicado: investigue o serviço de origem (Cadastros ou Pagamentos).
- Se foi publicado mas não consumido: verifique se o consumer está rodando (pode estar em CrashLoop).
- Se foi consumido mas "ignorado": investigue por que a OS já não está no estado-fonte (pode ter sido cancelada).

### `dotnet ef` migration falhou no startup

**Sintoma:** Pod em CrashLoopBackOff, log com `Database migration failed`.

**Diagnóstico:**

- Logs do pod métricos no startup. EF Core loga a migration específica que falhou.
- Conexão ao RDS OK? (`kubectl exec ... -- psql -h $DB_HOST`).
- Existe migration não-idempotente (raro)?

**Mitigação:**

- **Não** rode `dotnet ef database update` direto em prod.
- Reverta para a imagem anterior, corrija a migration, redeploya.

### Outbox engargalando

**Sintoma:** latência de evento (publish → consume) crescendo de minutos.

**Diagnóstico:**

- Métrica `outbox.processor.batch_size` — alto e crescendo? Backlog.
- Métrica `outbox.messages.failed` — muito alto? Algo errado no broker.
- Verifique conexão com RabbitMQ (`kubectl exec ... -- nc -zv rabbitmq 5672`).

**Mitigação:**

- Restart do pod (background service reinicia).
- Aumentar replicas — cada pod processa em paralelo via `SELECT ... FOR UPDATE SKIP LOCKED`.

### SAGA com lock travado

**Sintoma:** comandos para uma `OrdemDeServicoId` específica não avançam.

**Diagnóstico:**

- Inspecione `saga_ordem_de_servico` no Mongo: `db.saga_ordem_de_servico.findOne({_id: "<OrdemId>"})`.
- Estado `Pending`/`InProgress` por > sentinel timeout? Bug.

**Mitigação:**

- **Não** delete o registro — isso permite duplicação.
- Aguarde o sentinel timeout liberar (alguns minutos).
- Se permanecer, abra issue no repo com o dump do documento.

## Diagnóstico geral

Ver [`docs/runbook.md`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico/blob/main/docs/runbook.md) no repositório para mais cenários.

## Veja também

- [Runbook macro](Runbook-macro)
- [Resposta a incidentes](Resposta-a-incidentes)
- [Outbox transacional](Outbox-transacional)
