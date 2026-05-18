# Code review checklist

> **Rótulo:** Referência
> **TL;DR:** Itens a verificar antes de aprovar uma PR.
> **Última revisão:** 2026-05-18

## Geral

- [ ] Título segue [Conventional Commits](Padrao-de-commits-e-PRs).
- [ ] Descrição tem **Summary** e **Test plan**.
- [ ] CI verde (build, test, format, sonar, cobertura).
- [ ] Nenhuma credencial commitada.

## Domínio

- [ ] Novas regras de negócio estão no `Domain`, não na `Application` ou `Infrastructure`.
- [ ] Agregados retornam `Result<T>` em vez de lançar exception para erros de negócio.
- [ ] Se nova transição de estado: foi modelada como **classe de status**, não `if/switch`.
- [ ] Eventos de domínio levantados nas mudanças relevantes.
- [ ] Value Objects validados no construtor (sem entidade inválida).

## Application

- [ ] Comandos assíncronos seguem padrão `IAsyncCommand` + `AsyncCommandHandler`.
- [ ] Queries usam `IReader` (não `IRepository`) — garante read-only.
- [ ] Consumers de integração verificam **estado-fonte** antes de despachar à SAGA ([Idempotência cross-service](Idempotencia-cross-service)).
- [ ] FluentValidation para inputs HTTP.

## Infrastructure

- [ ] EF Core/Mongo entities ficam separadas de agregados (não reusar a mesma classe).
- [ ] Outbox usado para qualquer evento que precise sair do serviço.
- [ ] Mappers em Infrastructure (não em Application/Domain).

## API

- [ ] Endpoints retornam via `ResultPresenter`.
- [ ] Política de autorização explícita (`.RequireAuthorization(AuthPolicies.X)`).
- [ ] OpenAPI/Scalar atualizado (geralmente automático).
- [ ] Endpoints de mutação retornam **`202 Accepted`** quando assíncronos.

## Mensageria

- [ ] Eventos cross-service usam **contrato do SDK** (`Mecanica.Hermes.Contracts`), não classe local.
- [ ] Se evento for novo, foi adicionado ao SDK (PR separado).
- [ ] Versionamento `.v1` respeitado.
- [ ] Mudança de evento existente justificada e segue regra `.v2` em paralelo.

## Testes

- [ ] Unitários cobrem caminhos felizes E **erros de negócio** (Result.Failure).
- [ ] Architecture tests passam (regra de dependência).
- [ ] Se mexer em consumer: teste com `MassTransit.TestFramework`.
- [ ] Se mexer em endpoint: teste de integração com WebApplicationFactory.
- [ ] Cobertura **não cai** abaixo de 80%.

## Documentação

- [ ] README atualizado se houve mudança visível.
- [ ] `docs/` atualizado se houve mudança arquitetural.
- [ ] **Wiki** atualizada se a mudança invalida algo escrito ([Editar a Wiki](Editar-a-Wiki)).
- [ ] ADR criado se decisão arquitetural significativa.

## Performance / Segurança

- [ ] Não introduz N+1 query.
- [ ] Não expande superficie de ataque sem revisão.
- [ ] Webhooks públicos sempre com HMAC + dedup.

## Veja também

- [Padrão de commits e PRs](Padrao-de-commits-e-PRs)
- [Architecture Tests](Architecture-Tests)
