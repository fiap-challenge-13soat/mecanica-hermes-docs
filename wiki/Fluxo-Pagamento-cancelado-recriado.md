# Fluxo — Pagamento cancelado e recriado

> **Rótulo:** Explicação
> **TL;DR:** Primeiro pagamento é recusado (cartão negado). OS volta para `ManutencaoFinalizada`. Segundo pagamento aprovado conclui o fluxo.
> **Suíte E2E:** `tests/suites/02__pagamento_cancelado_recriado.robot`
> **Última revisão:** 2026-05-18

## Cenário

Cliente recebe link, tenta pagar com cartão A, MP recusa. OS reverte para `ManutencaoFinalizada` para que o atendente possa **gerar novo link de pagamento**. Cliente paga com cartão B, aprovado, fluxo termina.

## Sequência

```mermaid
sequenceDiagram
  participant OS
  participant Pag as Pagamentos
  participant MP as Mercado Pago
  participant Cli as Cliente

  OS->>Pag: ordem-de-servico.aguardando-pagamento.v1 (#1)
  Pag->>MP: cria preferência #1
  Cli->>MP: tenta pagar
  MP-->>Pag: webhook payment.rejected
  Pag-->>OS: pagamento.recusado.v1
  OS->>OS: AguardandoPagamento → ManutencaoFinalizada

  Note over OS: atendente avança de novo
  OS->>Pag: ordem-de-servico.aguardando-pagamento.v1 (#2)
  Pag->>MP: cria preferência #2
  Cli->>MP: paga (cartão diferente)
  MP-->>Pag: webhook payment.approved
  Pag-->>OS: pagamento.confirmado.v1
  OS->>OS: AguardandoPagamento → PagamentoConfirmado → Entregue
```

## Estados percorridos

| Etapa | OS | Pagamento #1 | Pagamento #2 |
|---|---|---|---|
| 1 | `AguardandoPagamento` | `AguardandoConfirmacao` | — |
| 2 | `ManutencaoFinalizada` | `Recusado` (terminal) | — |
| 3 | `AguardandoPagamento` | `Recusado` | `AguardandoConfirmacao` |
| 4 | `PagamentoConfirmado` → `Entregue` | `Recusado` | `Pago` |

## Eventos publicados

1. `ordem-de-servico.aguardando-pagamento.v1` (1ª vez)
2. `link-pagamento-gerado.v1`
3. `pagamento.recusado.v1`
4. `ordem-de-servico.aguardando-pagamento.v1` (2ª vez)
5. `link-pagamento-gerado.v1`
6. `pagamento.confirmado.v1`

## Resiliência

O consumer `pagamento.recusado.v1` na OS verifica que o estado-fonte é `AguardandoPagamento` antes de reverter (ver [Idempotência cross-service](Idempotencia-cross-service)). Isso evita reverter a OS que já saiu desse estado por outro motivo.

## Veja também

- [Fluxo — Caminho feliz](Fluxo-Caminho-feliz)
- [Idempotência cross-service](Idempotencia-cross-service)
