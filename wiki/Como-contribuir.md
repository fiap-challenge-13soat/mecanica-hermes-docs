# Como contribuir

> **Rótulo:** How-to
> **TL;DR:** Branch + PR para `main`. CI corre testes, cobertura, format e Sonar. Reviewer aprova. Merge.
> **Última revisão:** 2026-05-18

## Setup básico

1. Clone o repositório relevante.
2. Instale pré-requisitos (ver [Subir um único serviço](Subir-um-unico-servico) ou [Stack completa via Docker Compose](Stack-completa-via-Docker-Compose)).
3. Confirme acesso ao SDK em GitHub Packages — ver [SDK — Como consumir](SDK-Como-consumir).

## Workflow

```bash
git checkout main && git pull
git checkout -b feature/<descricao-curta>
# ... implementar ...
just test
just format-check
git add . && git commit -m "feat: ..."
git push origin feature/<descricao-curta>
# Abrir PR no GitHub para `main`
```

## Regras de ouro

1. **Nunca push direto em `main`** — sempre via PR.
2. **Cobertura ≥ 80%** — o CI falha se cair.
3. **Format check** verde — rode `just format` localmente antes do PR.
4. **Sonar quality gate** verde no PR.
5. **Suíte E2E** verde no PR (no repo `tests-e2e`) ou apenas em deploy (depende do escopo).
6. **ADR para mudanças arquiteturais** — ver [Índice de ADRs](Indice-de-ADRs).
7. **Atualizar Wiki** se a mudança invalida algo escrito — ver [Editar a Wiki](Editar-a-Wiki).

## Próximos passos

- [Padrão de commits e PRs](Padrao-de-commits-e-PRs)
- [Branching e ambientes](Branching-e-ambientes)
- [Code review checklist](Code-review-checklist)
- [Editar a Wiki](Editar-a-Wiki)
