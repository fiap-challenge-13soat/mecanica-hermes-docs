# Result Pattern

> **Rótulo:** Explicação
> **TL;DR:** Como representamos sucesso e falha em todas as camadas sem lançar exceções para fluxo de controle.
> **Última revisão:** 2026-05-18

## Motivação

Lançar exceções para sinalizar "OS não encontrada" ou "cliente inativo" tem 3 problemas:

1. **Custo** — stack trace caro.
2. **Implicitude** — a assinatura do método não documenta as falhas possíveis.
3. **Mistura semântica** — confunde *erro de programação* (deveria virar 500) com *resultado de negócio* (deveria virar 400/404/409).

O **Result Pattern** torna a falha **explícita** no tipo de retorno.

## Anatomia

Dois tipos definidos no SDK (`Mecanica.Hermes.Shared.Core`):

```csharp
public record Result
{
    public bool IsSuccess { get; }
    public bool IsFailure => !IsSuccess;
    public HttpStatusCode StatusCode { get; }
    public IReadOnlyList<Error> Errors { get; }

    public static Result Success(HttpStatusCode statusCode = HttpStatusCode.OK);
    public static Result Accepted();
    public static Result Failure(HttpStatusCode statusCode, params Error[] errors);
    public static Result NotFound(string message);
    public static Result Conflict(string message);
    public static Result Validation(params Error[] errors);
}

public record Result<T> : Result
{
    public T? Value { get; }
}
```

## Uso típico

### No domínio

```csharp
public Result<OrdemDeServicoStatusBase> AvancarEtapa()
{
    if (this is StatusEntregue or StatusCancelada or StatusRejeitada)
        return Result.Failure<OrdemDeServicoStatusBase>(
            HttpStatusCode.Conflict,
            new Error("OS.estado-terminal", "OS já está em estado terminal.")
        );
    return Result.Success(ProximoEstado());
}
```

### Na Application

```csharp
public async Task<Result<OrdemDeServicoDto>> Handle(GetOrdemByIdQuery query, CancellationToken ct)
{
    var os = await _reader.GetByIdAsync(query.Id, ct);
    return os is null
        ? Result.NotFound<OrdemDeServicoDto>($"OS {query.Id} não encontrada.")
        : Result.Success(os.ToDto());
}
```

### Na Api

O `ResultPresenter` converte `Result<T>` em `IResult` HTTP:

```csharp
public static IResult Present<T>(this Result<T> result) => result.StatusCode switch
{
    HttpStatusCode.OK        => Results.Ok(result.Value),
    HttpStatusCode.Created   => Results.Created(...),
    HttpStatusCode.Accepted  => Results.Accepted(),
    HttpStatusCode.NotFound  => Results.NotFound(result.AsProblemDetails()),
    HttpStatusCode.Conflict  => Results.Conflict(result.AsProblemDetails()),
    _                        => Results.Problem(result.AsProblemDetails())
};
```

## RFC 7807 — Problem Details

Quando o resultado é falha, a resposta segue [RFC 7807](https://datatracker.ietf.org/doc/html/rfc7807):

```json
{
  "type": "about:blank",
  "title": "Conflict",
  "status": 409,
  "detail": "OS já está em estado terminal.",
  "errors": [
    { "code": "OS.estado-terminal", "message": "OS já está em estado terminal." }
  ]
}
```

## Quando ainda lançamos exceção

- **Erro de programação** (argumento null em construtor, violação de invariante interna) — lançar exception. Vira 500 pelo `GlobalExceptionHandlerMiddleware`.
- **Concorrência otimista** — `ConcurrencyConflictException` é lançada pelo EF Core e o consumer trata explicitamente (retry da SAGA cuida).

## Veja também

- [Clean Architecture + DDD + CQRS](Clean-Architecture-DDD-CQRS)
- [State Pattern](State-Pattern)
- [SDK — Visão dos 6 pacotes](SDK-Visao-dos-6-pacotes)
