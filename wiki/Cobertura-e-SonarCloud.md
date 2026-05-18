# Cobertura e SonarCloud

> **Rótulo:** Referência
> **TL;DR:** Cobertura mínima 80% enforced no CI; SonarCloud valida quality gate, smells, vulnerabilities e duplicação.
> **Última revisão:** 2026-05-18

## Cobertura

### Ferramenta

- **Coverlet** — instrumenta o código .NET em runtime durante os testes.
- Saída: `coverage.opencover.xml`.
- Configuração: `coverlet.runsettings` (uma por repo).

### Comando padrão

```bash
dotnet test --settings coverlet.runsettings \
  --results-directory .temp/coverage \
  --collect:"XPlat Code Coverage"
```

### Relatório HTML

```bash
reportgenerator \
  -reports:.temp/coverage/**/coverage.opencover.xml \
  -targetdir:.temp/coverage/report \
  -reporttypes:TextSummary\;Html
```

Abra `.temp/coverage/report/index.html` no navegador.

### Limite

**80%** de cobertura de linhas. O CI **falha** se cair abaixo.

```yaml
- name: Test with coverage
  run: dotnet test --settings coverlet.runsettings
- name: Check threshold
  run: |
    coverage=$(awk -F\" '/line-rate/ {print $2}' .temp/coverage/coverage.cobertura.xml)
    if (( $(echo "$coverage < 0.80" | bc -l) )); then exit 1; fi
```

## SonarCloud

### Projetos

| Repositório | Project key |
|---|---|
| `api-ordem-servico` | `fiap-challenge-13soat_mecanica-hermes-api-ordem-servico` |
| `api-cadastros` | `fiap-challenge-13soat_mecanica-hermes-cadastros` |
| `api-pagamentos` | `fiap-challenge-13soat_mecanica-hermes-api-pagamentos` |
| `api-sdk` | `fiap-challenge-13soat_mecanica-hermes-shared-sdk` |

### Quality Gate (default)

O SonarCloud aplica o Quality Gate "Sonar way" em cada PR. Checa:

- 0 bugs new code.
- 0 vulnerabilities new code.
- ≤ 3% duplicação.
- Cobertura ≥ 80% no new code.

### Workflow

Cada repo de serviço tem `.github/workflows/sonarqube-ci.yml`:

- Trigger: push `main`, PR para `main`.
- Roda `dotnet test` + `dotnet sonarscanner begin/end`.
- Resultado no PR como check + comentamentos inline.

## Veja também

- [Estratégia de testes](Estrategia-de-testes)
- [Pipelines GitHub Actions](Pipelines-GitHub-Actions)
