# SDK — Como consumir

> **Rótulo:** How-to
> **TL;DR:** Configurar `nuget.config` com GitHub Packages e PAT classic.
> **Última revisão:** 2026-05-18

## `nuget.config` na raiz do repositório consumidor

```xml
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="github-mechermes" value="https://nuget.pkg.github.com/fiap-challenge-13soat/index.json" />
  </packageSources>
  <packageSourceCredentials>
    <github-mechermes>
      <add key="Username" value="%GITHUB_USERNAME%" />
      <add key="ClearTextPassword" value="%GITHUB_TOKEN%" />
    </github-mechermes>
  </packageSourceCredentials>
</configuration>
```

## Variáveis de ambiente locais

```bash
export GITHUB_USERNAME=seu-handle
export GITHUB_TOKEN=ghp_xxx  # PAT classic com read:packages + SSO autorizado pra fiap-challenge-13soat
```

### Limitação conhecida do GitHub

**Fine-grained PATs não funcionam** com `nuget.pkg.github.com` — gera 401 independente das permissões marcadas. **Use PAT classic**.

## No CI

No `.github/workflows/*.yml`:

```yaml
permissions:
  contents: read
  packages: read

steps:
  - uses: actions/checkout@v4
  - run: dotnet restore
    env:
      GITHUB_USERNAME: ${{ github.actor }}
      GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

O `GITHUB_TOKEN` automático do Actions tem `packages: read` quando declarado.

## No Dockerfile

O restore acontece dentro do build da imagem. Para não gravar o token em layers/cache:

```dockerfile
ARG GITHUB_USERNAME
RUN --mount=type=secret,id=github_token \
    GITHUB_TOKEN=$(cat /run/secrets/github_token) \
    GITHUB_USERNAME=$GITHUB_USERNAME \
    dotnet restore
```

Dockerfiles dos 3 serviços já fazem isso. O workflow `docker-ci.yml` propaga `--secret id=github_token,env=GITHUB_TOKEN`.

## Adicionando referências

Nos `.csproj`:

```xml
<ItemGroup>
  <PackageReference Include="Mecanica.Hermes.Shared.Core" />
  <PackageReference Include="Mecanica.Hermes.Shared.Application" />
  <PackageReference Include="Mecanica.Hermes.Shared.AspNetCore" />
  <PackageReference Include="Mecanica.Hermes.Shared.Observability" />
  <PackageReference Include="Mecanica.Hermes.Contracts" />
</ItemGroup>
```

**Sem `Version=`** — versions são centralizadas em `Directory.Packages.props`:

```xml
<ItemGroup>
  <PackageVersion Include="Mecanica.Hermes.Shared.Core" Version="1.0.0" />
  <PackageVersion Include="Mecanica.Hermes.Shared.Application" Version="1.0.0" />
  <PackageVersion Include="Mecanica.Hermes.Shared.AspNetCore" Version="1.0.0" />
  <PackageVersion Include="Mecanica.Hermes.Shared.Observability" Version="1.0.0" />
  <PackageVersion Include="Mecanica.Hermes.Contracts" Version="1.0.0" />
</ItemGroup>
```

## Veja também

- [SDK — Visão dos 6 pacotes](SDK-Visao-dos-6-pacotes)
- [SDK — Versionamento](SDK-Versionamento)
