# Padrão de commits e PRs

> **Rótulo:** Referência
> **TL;DR:** Conventional Commits no resumo. Body com **porquê**, não **o quê**. PRs com descrição curta + test plan.
> **Última revisão:** 2026-05-18

## Conventional Commits

```
<tipo>(<escopo opcional>): <resumo no imperativo>

<body opcional com o porquê>

<footer opcional: BREAKING CHANGE, Refs, Closes>
```

| Tipo | Quando |
|---|---|
| `feat` | Nova funcionalidade |
| `fix` | Correção de bug |
| `refactor` | Reorganização sem mudar comportamento |
| `test` | Adição/modificação de teste sem mudar código de produção |
| `docs` | Apenas documentação |
| `chore` | Build, deps, configuração |
| `perf` | Melhoria de performance |
| `ci` | Pipelines |

### Exemplos

```
feat(os): adicionar comando AvancarEtapa

A OS agora pode avançar de Recebida para EmDiagnostico via comando
assíncrono. Necessário para suportar fluxo 01 da suíte E2E.
```

```
fix(pagamentos): corrigir dedup do webhook MP em concorrência

O ON CONFLICT DO NOTHING não estava ativo no índice por causa de
case-sensitivity. Adiciona migration recriando índice em lower(mp_event_id).
```

```
refactor(cadastros): extrair WebhookOrcamentoHandler para Application

Move lógica fora da Api (composition root) para Application (use case).
Não muda comportamento; resolve violação detectada por ArchitectureTests.
```

## Resumo do PR

Título:

```
feat(os): adicionar comando AvancarEtapa
```

Descrição (template):

```markdown
## Summary
- 1-3 bullets explicando o porquê
- Mencionar mudança arquitetural se houver

## Test plan
- [ ] Testes unitários novos cobrem casos X, Y, Z
- [ ] Suíte E2E 01 passa localmente
- [ ] Coverage local ≥ 80%
```

## Tamanho

- **Prefira PR pequeno**: < 400 linhas.
- Se grande, divida em commits logícos (Git Stash + revisão por commit).

## Veja também

- [Code review checklist](Code-review-checklist)
