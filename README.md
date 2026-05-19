# mecanica-hermes-docs

Repositório de documentação central do ecossistema **Mecânica Hermes** (Tech Challenge 13SOAT — FIAP).

## O que tem aqui

- **[`WIKI-PLAN.md`](./WIKI-PLAN.md)** — Plano editorial completo da Wiki: audiências, estrutura (sidebar), detalhamento de páginas, mapeamento de fontes, convenções, roadmap em 6 fases e governança.
- **[`wiki/`](./wiki/)** — Conteúdo canônico da Wiki em arquivos Markdown versionados. Sincronizado para a Wiki nativa do GitHub (ver [`wiki/README.md`](./wiki/README.md)).

## Por que o conteúdo está versionado neste repo

A Wiki nativa do GitHub é um repositório Git separado (`<repo>.wiki.git`), mas **não suporta Pull Requests**. Versionando o conteúdo aqui obtemos:

- Code review das mudanças antes de publicar.
- Histórico unificado ligado aos commits/PRs do projeto.
- CI/CD futuro para validar links, lint de Markdown e renderização de Mermaid.
- Reuso dos diagramas pelos READMEs dos outros repositórios via permalinks.

## Wiki online

> 📖 **https://github.com/fiap-challenge-13soat/mecanica-hermes-docs/wiki**

## Os 9 repositórios do ecossistema

| Repositório | Papel |
|---|---|
| [`mecanica-hermes-infra`](https://github.com/fiap-challenge-13soat/mecanica-hermes-infra) | Infraestrutura AWS via Terraform (5 módulos) |
| [`mecanica-hermes-k8s`](https://github.com/fiap-challenge-13soat/mecanica-hermes-k8s) | Deploy Kubernetes via Kustomize |
| [`mecanica-hermes-lambda`](https://github.com/fiap-challenge-13soat/mecanica-hermes-lambda) | Lambda CognitoToken (login do cliente por CPF) |
| [`mecanica-hermes-api-ordem-servico`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-ordem-servico) | API da Ordem de Serviço (orquestrador) |
| [`mecanica-hermes-api-cadastros`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-cadastros) | API de Clientes e Produtos |
| [`mecanica-hermes-api-pagamentos`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-pagamentos) | API de Pagamentos (Mercado Pago) |
| [`mecanica-hermes-api-sdk`](https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk) | SDK compartilhado (6 pacotes NuGet) |
| [`mecanica-hermes-tests-e2e`](https://github.com/fiap-challenge-13soat/mecanica-hermes-tests-e2e) | Suíte E2E Robot Framework BDD |
| [`mecanica-hermes-docs`](https://github.com/fiap-challenge-13soat/mecanica-hermes-docs) | **Este repositório** — Wiki |

## Como contribuir

Ver [`wiki/Editar-a-Wiki.md`](./wiki/Editar-a-Wiki.md).

## Licença

MIT — ver [`LICENSE`](./LICENSE).
