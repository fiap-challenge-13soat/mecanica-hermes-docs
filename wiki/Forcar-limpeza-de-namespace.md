# Forçar limpeza de namespace

> **Rótulo:** How-to
> **TL;DR:** Quando um namespace fica preso em `Terminating`, use o workflow `Force Cleanup - Stuck Namespace` para remover os finalizers.
> **Última revisão:** 2026-05-18

## Quando usar

Namespace `mecanica-hermes-hml` ou `mecanica-hermes-prd` aparece em `kubectl get ns` com status `Terminating` por mais de **5 minutos**.

Causa típica: algum recurso dentro do namespace tem `finalizer` que não foi removido (por exemplo, NLB que não conseguiu deletar por causa de Security Group em uso).

## Workflow

Repositório: `mecanica-hermes-k8s`.

1. **Actions** → `Force Cleanup - Stuck Namespace`.
2. Run workflow → input `environment` (`hml` ou `prd`).
3. Confirmação no campo de segurança.

O workflow:

1. `kubectl get ns <namespace> -o json` salva o spec.
2. Edita removendo `spec.finalizers`.
3. `kubectl replace --raw "/api/v1/namespaces/<namespace>/finalize" -f -` injeta o spec.
4. Kubernetes deleta o namespace em segundos.

## Manual (sem o workflow)

```bash
NAMESPACE=mecanica-hermes-hml
kubectl get ns $NAMESPACE -o json > tmp.json
# Editar tmp.json: remover "kubernetes" de spec.finalizers
kubectl replace --raw "/api/v1/namespaces/$NAMESPACE/finalize" -f tmp.json
```

## Quando NUNCA usar

- Em namespaces que **não são** do projeto (só use em `mecanica-hermes-*`).
- Sem antes investigar a causa raiz por **10 minutos**.
- Em produção, sem alinhar com o time — pode mascarar um problema persistente.

## Investigação prévia

```bash
kubectl get events -n $NAMESPACE --sort-by=.lastTimestamp
kubectl describe ns $NAMESPACE
# Procure recursos travados:
kubectl get all,svc,ingress,pvc -n $NAMESPACE
# E os finalizers em cada um:
kubectl get <recurso> -n $NAMESPACE -o jsonpath='{.metadata.finalizers}'
```

## Veja também

- [Cluster Kubernetes](Cluster-Kubernetes)
- [Pipelines GitHub Actions](Pipelines-GitHub-Actions)
