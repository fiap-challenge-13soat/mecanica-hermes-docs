# State Pattern

> **Rótulo:** Explicação
> **TL;DR:** Cada status do agregado é uma **classe própria** que conhece suas próprias transições e permissões. Substitui `switch (status)` espalhado.
> **Última revisão:** 2026-05-18

## Motivação

A Ordem de Serviço tem 8 estados + 2 terminais. Cada estado:

- Define quais operações são permitidas (ex.: `AdicionarProduto` só antes de `EmExecucao`).
- Sabe para qual próximo estado ir em `AvancarEtapa()`.
- Sabe se pode reverter, cancelar, etc.

Com `switch (status)` em cada método do agregado, novos estados exigem editar **N lugares** — e o compilador não avisa se você esqueceu de um.

Com **State Pattern**, novos estados são uma classe nova herdando da base. O agregado delega para o estado atual.

## Implementação na OS

```csharp
public abstract class OrdemDeServicoStatusBase
{
    public abstract string Nome { get; }
    public abstract bool PermiteEditarProdutos { get; }

    public abstract Result<OrdemDeServicoStatusBase> AvancarEtapa();
    public abstract Result<OrdemDeServicoStatusBase> ReverterEtapa();
    public virtual Result<OrdemDeServicoStatusBase> Cancelar()
        => Result.Success(new StatusCancelada()); // default: pode cancelar
}

public sealed class StatusRecebida : OrdemDeServicoStatusBase
{
    public override string Nome => "Recebida";
    public override bool PermiteEditarProdutos => true;
    public override Result<OrdemDeServicoStatusBase> AvancarEtapa()
        => Result.Success<OrdemDeServicoStatusBase>(new StatusEmDiagnostico());
    public override Result<OrdemDeServicoStatusBase> ReverterEtapa()
        => Result.Failure<OrdemDeServicoStatusBase>(
            HttpStatusCode.Conflict,
            new Error("OS.sem-anterior", "Recebida é o estado inicial."));
}
```

No agregado:

```csharp
public Result AvancarEtapa()
{
    var resultado = _status.AvancarEtapa();
    if (resultado.IsFailure) return resultado;

    var novoStatus = resultado.Value!;
    _status = novoStatus;
    _historico.Add(new OrdemDeServicoHistoricoStatus(novoStatus.Nome, DomainClock.UtcNow));
    RaiseDomainEvent(new OrdemDeServicoStatusAlteradoEvent(Id, novoStatus.Nome));
    return Result.Success();
}
```

## Estados da OS

Ver diagrama completo em [Domínio de negócio](Dominio-de-negocio#ciclo-de-vida-da-ordem-de-serviço).

| Estado | `PermiteEditarProdutos` | `AvancarEtapa` |
|---|---|---|
| `Recebida` | ✅ | `EmDiagnostico` |
| `EmDiagnostico` | ✅ | `AguardandoAprovacao` |
| `AguardandoAprovacao` | ❌ | (via consumer) `EmExecucao` ou `Rejeitada` |
| `EmExecucao` | ❌ | `ManutencaoFinalizada` |
| `ManutencaoFinalizada` | ❌ | `AguardandoPagamento` |
| `AguardandoPagamento` | ❌ | (via consumer) `PagamentoConfirmado` ou volta para `ManutencaoFinalizada` |
| `PagamentoConfirmado` | ❌ | `Entregue` |
| `Entregue` | ❌ | falha (estado terminal) |
| `Rejeitada` | ❌ | falha (terminal) |
| `Cancelada` | ❌ | falha (terminal) |

## Estados de Pagamento

A API de Pagamentos tem **state machine própria**:

```text
PendenteGeracao
    │ GerarLink (chama MP)
    ▼
LinkGerado
    │ MarcarEmailEnviado
    ▼
AguardandoConfirmacao
    │ Confirmar → Pago      (terminal)
    │ Recusar  → Recusado  (terminal)
    │ Expirar  → Expirado  (terminal)
```

## Vantagens

- Compilador força a definir todas as operações ao adicionar um estado.
- Cada estado é testável isoladamente.
- Histórico de transições vem de graça (`OrdemDeServicoHistoricoStatus` adicionado a cada mudança).

## Veja também

- [Domínio de negócio](Dominio-de-negocio)
- [Result Pattern](Result-Pattern)
