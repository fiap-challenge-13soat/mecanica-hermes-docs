# Repo: `mecanica-hermes-docs`

> **Rótulo:** Referência
> **Papel em uma frase:** Repositório desta Wiki. Contém o plano editorial e os arquivos Markdown que abastecem o GitHub Wiki, sincronizados via GitHub Actions.
> **Stack:** Markdown, GitHub Wiki, Mermaid, GitHub Actions
> **Categoria:** Documentação
> **Última revisão:** 2026-05-19

## Resumo

Repositório que você está lendo agora. A Wiki nativa do GitHub é sincronizada **automaticamente** a partir da pasta [`wiki/`](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/tree/main/wiki) deste repo a cada merge em `main`, via o workflow [`Sync Wiki`](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/.github/workflows/sync-wiki.yml).

O **plano editorial** está em [`WIKI-PLAN.md`](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/WIKI-PLAN.md) na raiz.

## Como se relaciona com o resto

- **Documenta:** os outros 8 repositórios.
- **Não tem código executável** além do workflow de sync.

## Pontos-chave

- Conteúdo em **PT-BR**.
- Diagramas em **Mermaid** (consistente com READMEs).
- Estrutura inspirada em **arc42 + C4 + Diátaxis**.
- Cada página começa com cabeçalho padrão (Rótulo, TL;DR, Última revisão).
- Mudanças via PR para `main`, com revisão — após merge, o workflow `Sync Wiki` republica os arquivos em `<repo>.wiki.git`.

## Workflows GitHub Actions

| Workflow | O que faz |
|---|---|
| `Sync Wiki` | A cada push em `main` que toque `wiki/**`, clona o repo `*.wiki.git`, remove `*.md` antigos, copia os novos e commita com a mensagem `sync: wiki from main @ <sha>` |

## Como contribuir

Ver [Editar a Wiki](Editar-a-Wiki).

## Veja também

- [Home](Home)
- [Plano editorial (WIKI-PLAN.md)](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/WIKI-PLAN.md)
