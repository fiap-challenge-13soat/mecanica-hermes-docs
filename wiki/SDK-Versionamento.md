# SDK — Versionamento

> **Rótulo:** Referência
> **TL;DR:** SemVer estrito. Versão única para os 6 pacotes. Bump MAJOR para qualquer mudança quebrante.
> **Última revisão:** 2026-05-18

## Versão única

As versões dos 6 pacotes são **sincronizadas** — todos sobem juntos. Definidas em `Directory.Build.props` na raiz:

```xml
<PropertyGroup>
  <Version>1.0.0</Version>
</PropertyGroup>
```

## SemVer estrito

| Mudança | Bump |
|---|---|
| Adicionar tipo público | MINOR |
| Adicionar `EntityName` novo (`.v1`) | MINOR |
| Adicionar membro a tipo público existente (interface/abstract class) | MAJOR (consumers existentes quebram) |
| Adicionar membro a record posicional (Contracts) | MAJOR (serialização quebra) |
| Renomear tipo público ou mover de namespace | MAJOR |
| Renomear `EntityName` ou remover record | MAJOR |
| Mudar comportamento default | MAJOR |
| Fix interno sem afetar API pública | PATCH |

## Pre-releases

Sufixo NuGet padrão:

- `1.0.0-preview1`
- `1.0.0-rc1`
- `1.0.0`

Para publicar preview:

```bash
just pack-version 1.0.0-preview2
```

## Workflow de release

1. Atualizar `Version` em `Directory.Build.props`.
2. Commit em `main` via PR.
3. Criar tag `v1.0.0`:

   ```bash
   git tag v1.0.0
   git push origin v1.0.0
   ```

4. Workflow `Publish to GitHub Packages` dispara automaticamente, publica os 6 `.nupkg` + `.snupkg`.
5. Atualizar `<PackageVersion>` em `Directory.Packages.props` dos consumers (3 PRs).

## Política de evolução de Contracts

Ver [Versionamento de contratos](Versionamento-de-contratos).

## Source Link

Habilitado em `Directory.Build.props`. Permite step-into no debugger do consumidor direto no código do SDK (`.snupkg` contém símbolos + reference ao GitHub).

## Veja também

- [Versionamento de contratos](Versionamento-de-contratos)
- [SDK — Como consumir](SDK-Como-consumir)
