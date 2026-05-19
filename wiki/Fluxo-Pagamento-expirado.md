# Fluxo — Pagamento expirado

> **Rótulo:** Explicação
> **TL;DR:** Cliente não paga dentro do prazo do MP. Pagamento expira por timeout. OS volta para `ManutencaoFinalizada`.
> **Suíte E2E:** `tests/suites/05__pagamento_expirado.robot`
> **Última revisão:** 2026-05-18

## Cenário

Cliente recebe link mas não paga. O Mercado Pago expira a preferência. O polling agendado da SAGA de Pagamentos detecta status `cancelled`/`expired` e marca `Expirado`. OS é notificada e volta para `ManutencaoFinalizada` (mesmo comportamento do `Recusado`).

## Sequência

```mermaid
sequenceDiagram
  participant Pag as Pagamentos
  participant MP as Mercado Pago
  participant OS

  Note over Pag,MP: Pagamento criado mas não pago
  loop polling a cada 2 min
    Pag->>MP: GET /v1/payments/search?external_reference=...
    MP-->>Pag: status=pending
    Pag->>Pag: incrementa contador
  end
  Note over MP: timeout (definido no MP)
  Pag->>MP: GET /v1/payments/search
  MP-->>Pag: status=cancelled (expirado)
  Pag->>Pag: Pagamento → Expirado
  Pag-->>OS: pagamento.recusado.v1
  OS->>OS: AguardandoPagamento → ManutencaoFinalizada
```

## Notas

- A SAGA de Pagamentos limita o número de verificações de polling (`LimiteVerificacoes`, default 30). Atingindo o limite sem confirmação, marca `Expirado`.
- Usamos o mesmo evento `pagamento.recusado.v1` para `Recusado` e `Expirado` — do ponto de vista da OS, ambos resultam em **reverter** o estado.

## Eventos publicados

1. `ordem-de-servico.aguardando-pagamento.v1`
2. `link-pagamento-gerado.v1`
3. `pagamento.recusado.v1` (com motivo `Expirado` no payload, se quiser distinguir em log)

## Veja também

- [SAGA com MassTransit](SAGA-com-MassTransit)
- [Fluxo — Pagamento cancelado e recriado](Fluxo-Pagamento-cancelado-recriado)
