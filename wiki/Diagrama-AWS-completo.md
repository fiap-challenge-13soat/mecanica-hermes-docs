# Diagrama AWS completo

> **Rótulo:** Referência
> **TL;DR:** Todos os recursos AWS provisionados, em um único diagrama.
> **Última revisão:** 2026-05-18

## Diagrama

```mermaid
flowchart TB
  subgraph EXT[Internet]
    User([Usuário interno + cliente])
    MP[Mercado Pago]
    SMTP[SMTP externo]
  end

  subgraph AWS[AWS]
    subgraph Net[VPC mechermes-vpc]
      subgraph Pub[Subnets Públicas]
        NAT[NAT Gateway]
        NLB[NLB do EKS]
      end
      subgraph Priv[Subnets Privadas]
        EKS[EKS Cluster<br/>K8s 1.34]
        NG[Node Groups<br/>t3.small]
      end
      subgraph DBNet[Subnets de Banco]
        RDS[(RDS PostgreSQL 17.6<br/>db.t4g.micro)]
      end
      SG[Security Group<br/>mechermes-sg]
    end

    APIG[API Gateway HTTP]
    Cognito[Cognito User Pool<br/>+ App Client<br/>+ Resource Server]
    Lambda[Lambda<br/>CognitoTokenLambda]
    SM[Secrets Manager]
    S3[(S3<br/>tfstate)]
    DDB[(DynamoDB<br/>tf locks)]
    CW[CloudWatch Logs]
  end

  subgraph Saas[SaaS]
    NR[New Relic APM]
    DH[Docker Hub<br/>mechermes/*]
    GH[GitHub Packages<br/>SDK NuGet]
  end

  User -->|HTTPS| APIG
  APIG -->|VPC Link| NLB --> EKS
  APIG -->|invoke| Lambda
  APIG --> Cognito
  Lambda --> Cognito
  Lambda --> SM
  Lambda --> RDS
  Lambda --> SMTP

  EKS --> RDS
  EKS --> SMTP
  EKS --> MP
  MP -.webhook HMAC.-> APIG
  EKS -.OTLP.-> NR
  EKS -.pull image.-> DH
  EKS -.pull NuGet.-> GH

  Net --> NAT
  EKS --> NAT --> Internet

  S3 -.tfstate.-> Terraform[GitHub Actions]
  DDB -.lock.-> Terraform
  CW -.logs.-> Lambda
```

## Lista de recursos

### Rede

| Recurso | Quantidade | Onde é criado |
|---|---|---|
| VPC | 1 | `mecanica-hermes-infra/aws/` |
| Subnets públicas | 2 (multi-AZ) | `aws/` |
| Subnets privadas | 2 | `aws/` |
| Subnets de banco | 2 | `aws/` |
| NAT Gateway | 1 | `aws/` |
| Internet Gateway | 1 | `aws/` |
| Security Group `mechermes-sg` | 1 | `aws/` |

### Compute

| Recurso | Quantidade | Onde é criado |
|---|---|---|
| EKS Cluster | 1 por env | `aws/` |
| Node Group | 1 por env | `aws/` |
| Lambda CognitoToken | 1 por env | `mecanica-hermes-lambda` |

### Persistência

| Recurso | Quantidade | Onde é criado |
|---|---|---|
| RDS Postgres | 1 por env | `db/` |
| S3 bucket `mechermes-tf-state-aws` | 1 (global) | manual ou primeiro run |
| DynamoDB `mechermes-tf-locks` | 1 (global) | manual ou primeiro run |

### Identity & Secrets

| Recurso | Quantidade | Onde é criado |
|---|---|---|
| Cognito User Pool | 1 por env | `cognito/` |
| App Client (Client Credentials) | 1 por env | `cognito/` |
| Resource Server `mechermes` | 1 por env | `cognito/` |
| Cognito Domain | 1 por env | `cognito/` |
| Secret `mechermes-{env}-cognito-client-secret` | 1 por env | `lambda` deploy |
| Secret do RDS Master | 1 por env | `db/` (gerenciado pelo RDS) |

### Edge / Routing

| Recurso | Onde |
|---|---|
| API Gateway HTTP | `api-gateway/` |
| Cognito Authorizer | `api-gateway/` |
| VPC Link | `api-gateway/` |
| CloudWatch Log Group | `api-gateway/` |

## Veja também

- [Provisionamento (Terraform)](Provisionamento-Terraform)
- [Cluster Kubernetes](Cluster-Kubernetes)
- [API Gateway + VPC Link](API-Gateway-VPC-Link)
- [Cognito](Cognito-User-Pool)
