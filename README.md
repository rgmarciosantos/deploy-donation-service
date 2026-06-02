# Deploy Donation Service

Repositório GitOps com os manifestos Kubernetes do **Donation Service** da plataforma SolidaryTech.

## Visão Geral

Este repositório é monitorado pelo **ArgoCD**. A tag da imagem Docker em `manifests/deployment.yaml` é atualizada **automaticamente** pelo pipeline de CI do repositório [`donation-service`](https://github.com/brianmonteiro54/donation-service) a cada push na branch `main`. Não edite a tag manualmente.

Stack: **Go + PostgreSQL (RDS) + SQS** · porta `8082` · namespace `solidarytech-donation`.

## Manifestos

| Arquivo | Descrição |
|---|---|
| `deployment.yaml` | Deployment (2 réplicas, rolling update, probes) + initContainer que monta a `DATABASE_URL` |
| `service.yaml` | Service ClusterIP na porta 8082 |
| `ingress.yaml` | Ingress nginx, rota `/donations` |
| `externalsecret.yaml` | ExternalSecrets: credenciais do RDS + config da app (DB host/port/name, SQS URL) |
| `secretstore.yaml` | SecretStore apontando para o AWS Secrets Manager (us-east-1) |
| `init-db.yaml` | ConfigMap + Job (sync hook wave 1) que cria a tabela `donations` |

## Pré-requisitos (uma vez)

1. **Infra aplicada** (`terraform apply`): cria o ECR `solidarytech/donation-service`, o RDS `donation_db`, a fila SQS, os namespaces e o secret `aws-credentials` em cada namespace.
2. **Secret de config no AWS Secrets Manager** chamado `solidarytech/donation-service` com as chaves:
   - `DONATION_DB_HOST` → endpoint do RDS donation (`terraform output rds_endpoints`)
   - `DONATION_DB_PORT` → `5432`
   - `DONATION_DB_NAME` → `donation_db`
   - `DONATION_SQS_URL` → URL da fila SQS (opcional)
3. **UUID do secret do RDS**: em `externalsecret.yaml`, troque `rds!db-REPLACE_WITH_DONATION_DB_SECRET_UUID` pelo nome real (Console AWS → Secrets Manager → filtre por `rds!db-`).
4. **App no ArgoCD** apontando para este repositório (`path: manifests`, namespace `solidarytech-donation`).

> **Acesso público:** `http://solidarytech.pt/donation` (o ingress reescreve o path singular para a rota real `/donations` da aplicação). Aponte o DNS de `solidarytech.pt` para o IP/Hostname do Load Balancer do ingress-nginx.
