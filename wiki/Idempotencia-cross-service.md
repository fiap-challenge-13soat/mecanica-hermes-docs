# Idempotência cross-service

> **Rótulo:** Explicação
> **TL;DR:** Consumers de eventos `.v1` **verificam estado-fonte** antes de despachar à SAGA. Eventos duplicados ou tardios são ignorados com métrica.
> **Última revisão:** 2026-05-18

## Por que precisamos

Mensagens de RabbitMQ são **at-least-once** — podem ser entregues mais de uma vez por:

- **Retry/redelivery** do MassTransit após falha.
- **Webhook re-enviado** pelo Mercado Pago após nosso ack.
- **Operador clicando 2 vezes** no webhook de orçamento.
- **Falha entre publish e ack** — o broker não sabe que processamos.

Se o consumer **avançar a OS** sem checar, podemos passar de `AguardandoAprovacao` para `EmExecucao` duas vezes — e da segunda, a transição falha (estado errado) e a SAGA marca falha, gerando ruído em DLQ.

## Padrão adotado

Os **4 consumers de integração** na OS (`Application/OrdensDeServico/Consumers/Integration/`) seguem:

```csharp
public async Task Consume(ConsumeContext<OrcamentoAprovadoPeloClienteEvent> context)
{
    var os = await _reader.GetByIdAsync(context.Message.OrdemDeServicoId);
    if (os is null) return; // OS removida

    if (os.Status.Nome != "AguardandoAprovacao")
    {
        _metrics.EventoIntegracaoIgnorado(
            evento: "orcamento.aprovado-pelo-cliente.v1",
            statusAtual: os.Status.Nome);
        return; // idempotência: skip silencioso
    }

    await _sender.Send(new AprovarOrcamentoCommand(os.Id));
}
```

## Tabela de verificações

| Evento consumido | Estado-fonte exigido | Ação se OS já saiu dele |
|---|---|---|
| `orcamento.aprovado-pelo-cliente.v1` | `AguardandoAprovacao` | skip + métrica |
| `orcamento.rejeitado-pelo-cliente.v1` | `AguardandoAprovacao` | skip + métrica |
| `pagamento.confirmado.v1` | `AguardandoPagamento` | skip + métrica |
| `pagamento.recusado.v1` | `AguardandoPagamento` | skip + métrica |

## Dedup de webhooks

Além do estado-fonte, webhooks públicos também fazem **dedup por event-id**:

- **Cadastros** — tabela `webhook_events`, coluna `webhook_event_id` PRIMARY KEY. Se INSERT falhar com violação, retorna 200 OK (idempotente).
- **Pagamentos** — coleção `mp_webhook_events`, dedup por `mp_event_id` que vem no payload do MP.

## Métrica

Counter `ordemdeservico.evento_integracao_ignorado` (meter `MecanicaHermes.Business`) com tags:

- `evento` — nome do `.v1`.
- `status_atual` — estado real da OS quando o evento chegou.

Utilidade: detectar **fluxo invertido** (ex.: pagamento confirmado chegando antes da OS chegar em `AguardandoPagamento` — algo errado na cronologia).

## Quando **não** é idempotência

Algumas operações não devem ser idempotentes:

- **Adicionar produto** — cliente quer mesmo 2 unidades, não deduplicamos.
- **Avançar etapa** — a SAGA garante uma chamada por vez, não queremos suprimir.

Nesses casos, dependemos da SAGA + transação do banco para evitar duplicidade.

## Veja também

- [SAGA com MassTransit](SAGA-com-MassTransit)
- [Webhooks assinados (HMAC)](Webhooks-assinados-HMAC)
- [Catálogo de eventos](Catalogo-de-eventos)
