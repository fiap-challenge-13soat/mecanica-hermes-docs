# SDK — Como contribuir

> **Rótulo:** How-to
> **TL;DR:** Branch + PR. CI valida formato + smoke tests. Tag publica.
> **Última revisão:** 2026-05-18

## Setup

```bash
git clone https://github.com/fiap-challenge-13soat/mecanica-hermes-api-sdk.git
cd mecanica-hermes-api-sdk
just build
just test
```

## Workflow

1. Branch a partir de `main`: `git checkout -b feature/<descricao>`.
2. Implemente + adicione/atualize testes.
3. Rode local:

   ```bash
   just build
   just test         # 22 smoke tests
   just format-check # 4 checagens idênticas ao CI
   just pack         # gera artifacts
   ```

4. PR para `main` com:
   - Descrição clara da mudança.
   - Se quebrar API: explicitar e propor caminho de migração.
   - Atualização de `docs/` se aplicável.
5. CI valida; reviewer aprova; merge.
6. Quando estiver pronto pra release, **outro PR** atualiza `Version` em `Directory.Build.props` e cria tag.

## Regras

- **Nunca commit em `main` direto** — sempre via PR.
- **Smoke tests** são vinculantes — mexer em `Contracts` exige atualizar `IntegrationEventNamesTests` se renomear (mas renomear é MAJOR, lembre).
- **Não adicionar deps externas** sem revisar. Cada pacote tem seu círculo de dependências definido.
- **Não mover código dos repos consumers para o SDK** sem antes alinhar com os 3 repos — a migração do código deles é outro PR.

## Veja também

- [SDK — Versionamento](SDK-Versionamento)
- [Repo: mecanica-hermes-api-sdk](Repo-mecanica-hermes-api-sdk)
