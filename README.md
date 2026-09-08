# Hybrid Cloud Infrastructure

Infrastructure as Code and hybrid-cloud deployment patterns for enterprise workloads.

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                     Global Load Balancer                      │
└───────────────────────────┬─────────────────────────────────┘
                            │
            ┌───────────────┴───────────────┐
            │                               │
     ┌──────▼──────┐                 ┌──────▼──────┐
     │  AWS Region │                 │  Azure Region│
     │  (Primary)  │                 │  (DR)       │
     └─────────────┘                 └─────────────┘
            │                               │
     ┌──────▼──────┐                 ┌──────▼──────┐
     │   EKS/K8s   │                 │   AKS/K8s   │
     │  - App Pods │                 │  - App Pods │
     │  - Redis    │                 │  - Redis    │
     │  - Postgres │◄────Replication────►│  - Postgres │
     └─────────────┘                 └─────────────┘
```

## IaC Modules

### Terraform Structure

```
modules/
  networking/
    vpc.tf
    subnets.tf
    peering.tf
  compute/
    eks.tf
    aks.tf
  database/
    rds.tf
    postgresql.tf
  security/
    iam.tf
    kms.tf
```

### Key Patterns

#### 1. Multi-Region Failover

- Primary region serves all traffic
- DR region replicates data async
- DNS failover with health checks
- RTO: <5 minutes, RPO: <1 minute

#### 2. Secrets Management

- HashiCorp Vault for dynamic secrets
- AWS Secrets Manager / Azure Key Vault as fallback
- Never store secrets in IaC state files
- Use `terraform remote state` with encryption

#### 3. Network Isolation

- Private subnets for data layer
- NAT Gateway for outbound traffic
- Security groups with least privilege
- VPC peering for cross-cloud communication

## CI/CD Pipeline

```yaml
stages:
  - validate: terraform fmt && terraform validate
  - plan: terraform plan -out=tfplan
  - apply: terraform apply tfplan (manual approval)
  - smoke_test: verify endpoints
```

## Cost Optimization

- Use spot/preemptible instances for non-critical workloads
- Implement auto-scaling with predictive scaling
- Right-size instances based on CloudWatch/Monitor metrics
- Use reserved instances for baseline capacity

## References

- AWS Well-Architected Framework
- Azure Architecture Center
- Terraform AWS/Azure Providers
