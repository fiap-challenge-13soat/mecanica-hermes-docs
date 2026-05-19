# Fluxo — Caminho feliz

> **Rótulo:** Explicação
> **TL;DR:** Da chegada do veículo à entrega após pagamento aprovado, sem desvios.
> **Suíte E2E:** `tests/suites/01__caminho_feliz.robot`
> **Última revisão:** 2026-05-18

## Cenário

Cliente leva o carro à oficina. Atendente abre OS. Mecânico monta orçamento. Cliente aprova por webhook. Manutenção concluída. Cliente paga pelo Mercado Pago. Carro é entregue.

## Sequência

```mermaid
sequenceDiagram
  participant At as Atendente
  participant OS
  participant Cad as Cadastros
  participant Pag as Pagamentos
  participant MP as Mercado Pago
  participant Cli as Cliente
  participant SMTP

  At->>OS: POST /ordens-de-servico (cliente, veículo)
  OS-->>At: 202 Accepted
  At->>OS: PATCH /{id}/avancar (Recebida → EmDiagnostico)
  At->>OS: POST /{id}/produtos (adiciona peças)
  At->>OS: POST /{id}/servicos (adiciona mão de obra)
  At->>OS: PATCH /{id}/avancar (EmDiagnostico → AguardandoAprovacao)
  OS-->>Cad: ordem-de-servico.aguardando-aprovacao.v1
  Cad->>Cad: cria webhook_link
  Cad->>SMTP: e-mail com link
  SMTP-->>Cli: e-mail
  Cli->>Cad: POST /webhooks/orcamentos/{token} (aprovado)
  Cad-->>OS: orcamento-aprovado-pelo-cliente.v1
  OS->>OS: AguardandoAprovacao → EmExecucao

  At->>OS: PATCH /{id}/avancar (EmExecucao → ManutencaoFinalizada)
  At->>OS: PATCH /{id}/avancar (ManutencaoFinalizada → AguardandoPagamento)
  OS-->>Pag: ordem-de-servico.aguardando-pagamento.v1
  Pag->>MP: POST /checkout/preferences
  MP-->>Pag: preference_id + init_point
  Pag-->>OS: link-pagamento-gerado.v1
  Pag-->>Cad: link-pagamento-gerado.v1
  Cad->>SMTP: e-mail com link MP
  SMTP-->>Cli: e-mail

  Cli->>MP: paga
  MP-->>Pag: POST /webhooks/mercadopago (HMAC)
  Pag->>MP: GET /v1/payments/{id} (confirma status)
  Pag-->>OS: pagamento.confirmado.v1
  OS->>OS: AguardandoPagamento → PagamentoConfirmado
  At->>OS: PATCH /{id}/avancar (PagamentoConfirmado → Entregue)
```

## Estados percorridos

| Etapa | OS | Pagamento |
|---|---|---|
| 1 | `Recebida` | — |
| 2 | `EmDiagnostico` | — |
| 3 | `AguardandoAprovacao` | — |
| 4 | `EmExecucao` | — |
| 5 | `ManutencaoFinalizada` | — |
| 6 | `AguardandoPagamento` | `PendenteGeracao` → `LinkGerado` → `AguardandoConfirmacao` |
| 7 | `PagamentoConfirmado` | `Pago` |
| 8 | `Entregue` | `Pago` (terminal) |

## Eventos publicados

1. `ordem-de-servico.aguardando-aprovacao.v1` (OS → Cadastros)
2. `orcamento-aprovado-pelo-cliente.v1` (Cadastros → OS)
3. `ordem-de-servico.aguardando-pagamento.v1` (OS → Pagamentos)
4. `link-pagamento-gerado.v1` (Pagamentos → OS + Cadastros)
5. `pagamento.confirmado.v1` (Pagamentos → OS)

## Veja também

- [Catálogo de eventos](Catalogo-de-eventos)
- [Domínio de negócio](Dominio-de-negocio)
- [Fluxo — Pagamento cancelado e recriado](Fluxo-Pagamento-cancelado-recriado) — a variação mais comum
