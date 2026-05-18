# Editar a Wiki

> **Rótulo:** How-to
> **TL;DR:** A Wiki vive como arquivos Markdown na pasta `wiki/` do repositório `mecanica-hermes-docs`. Mudanças via PR para `main`. Sincronização com a Wiki nativa do GitHub é manual (TODO automatizar).
> **Última revisão:** 2026-05-18

## Workflow

1. **Branch:** `git checkout -b docs/<descricao>` em `mecanica-hermes-docs`.
2. **Edite** os `.md` em `wiki/`.
3. **Pre-render** localmente (opcional): copie pra Wiki dev nativa ou abra no VS Code com extensão de Mermaid.
4. **PR** para `main`.
5. **Review** — outro membro do time.
6. **Merge.**
7. **Sync para Wiki nativa** (manual, por enquanto):

   ```bash
   git clone https://github.com/fiap-challenge-13soat/mecanica-hermes-docs.wiki.git
   cd mecanica-hermes-docs.wiki
   rm -f *.md
   cp ../mecanica-hermes-docs/wiki/*.md ./
   git add .
   git commit -m "sync: wiki content from main"
   git push origin master
   ```

## Convenções

Ver [plano editorial](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/WIKI-PLAN.md) para detalhes.

- **Idioma:** PT-BR.
- **Nome de arquivo:** kebab-case sem acentos. Ex.: `Catalogo-de-eventos.md`.
- **Cabeçalho padrão** no topo de toda página (Rótulo + TL;DR + Última revisão).
- **Mermaid** para diagramas.
- **Links internos:** `[Nome](Slug-da-pagina)` (sem `.md`).
- **Links externos:** URL completa.
- **Não duplique** conteúdo de README — resuma e linke.
- **Não exponha** credenciais ou valores reais.

## Templates

Ver seção 12 do [WIKI-PLAN.md](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/WIKI-PLAN.md#12-templates-de-páginas). Há templates para:

- Explicação
- Referência
- How-to
- Tutorial
- Página por repositório
- Página de fluxo de negócio

## Sidebar e Footer

- `_Sidebar.md` — menu lateral. Editar quando criar/remover página.
- `_Footer.md` — rodapé. Raramente muda.

O GitHub trata esses arquivos de forma especial — eles não aparecem como páginas no menu, mas são renderizados em toda página.

## Checklist antes de publicar

Ver [WIKI-PLAN.md § 13](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/WIKI-PLAN.md#13-apêndice--checklist-de-pronto-para-publicar).

## Veja também

- [Plano editorial (WIKI-PLAN.md)](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/blob/main/WIKI-PLAN.md)
- [Repo: mecanica-hermes-docs](Repo-mecanica-hermes-docs)
