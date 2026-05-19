# Conteúdo da Wiki

Esta pasta contém o **conteúdo canônico** da Wiki do Mecânica Hermes em arquivos Markdown versionados.

A Wiki nativa do GitHub (https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/wiki) é sincronizada a partir desta pasta. Manter o conteúdo aqui permite:

- **Code review** das mudanças via Pull Request (a Wiki nativa não suporta PRs).
- **Histórico unificado** ligado a commits/PRs do projeto.
- **CI/CD** para validar links quebrados, lint de Markdown e renderização de Mermaid antes de publicar.
- **Reuso** dos diagramas e tabelas pelos READMEs dos outros repositórios via permalinks.

## Convenções

- Nome de arquivo em **kebab-case sem acentos**: `catalogo-de-eventos.md`.
- O GitHub Wiki transforma `Catalogo-de-eventos.md` em página `Catalogo de eventos` — mantemos os nomes com a mesma transformação.
- `_Sidebar.md` e `_Footer.md` são reservados pelo GitHub Wiki e ficam visíveis em todas as páginas.
- Todo conteúdo segue o **plano editorial** em [`WIKI-PLAN.md`](../WIKI-PLAN.md) da raiz.

## Como sincronizar com a Wiki nativa (manual)

A Wiki do GitHub é um repositório Git separado em `<repo>.wiki.git`. Procedimento manual após merge na `main`:

```bash
git clone https://github.com/fiap-challenge-13soat/mecanica-hermes-docs.wiki.git
cd mecanica-hermes-docs.wiki
rm -f *.md
cp ../mecanica-hermes-docs/wiki/*.md ./
git add .
git commit -m "sync: wiki content from main"
git push origin master
```

No futuro este passo pode virar um workflow do GitHub Actions disparado por `push` em `main` (TODO).

## Estrutura

Ver [`_Sidebar.md`](./_Sidebar.md) para a árvore completa de páginas.
