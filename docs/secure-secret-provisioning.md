# Secure Kubernetes Secret Provisioning

This repository does not store Kubernetes Secret values.

## Namespace

Application workloads run in the `cloud-platform` namespace.

Create the namespace before provisioning credentials:

    kubectl create namespace cloud-platform --dry-run=client -o yaml | kubectl apply -f -

## Required Secrets

Provision these Kubernetes Secrets securely before applying the application:

### backend-secret

Required keys:

- FIRST_SUPERUSER
- FIRST_SUPERUSER_PASSWORD
- POSTGRES_PASSWORD
- SECRET_KEY

### postgres-secret

Required keys:

- POSTGRES_DB
- POSTGRES_USER
- POSTGRES_PASSWORD

The `POSTGRES_PASSWORD` value must be the same in both Secrets.
Use the same PostgreSQL username and database settings expected by the
application configuration.

Generate unique, strong credentials for each deployment environment.

Do not commit passwords, JWT signing keys, Secret YAML containing
credentials, or Kubernetes kubeconfig files to Git.

## Secret Provisioning

Use an approved secrets-management mechanism or provision the Secrets
using `kubectl create secret generic` with values supplied securely
from your deployment environment.

Avoid entering secrets directly into shell command arguments because
they can appear in shell history or process listings.

For production deployments, prefer centralized secret management
such as AWS Secrets Manager with External Secrets Operator.

## Deployment Order

1. Create namespace `cloud-platform`.
2. Provision `postgres-secret` and `backend-secret`.
3. Confirm required Secret keys exist.
4. Apply the Kustomize deployment.
5. Verify the PostgreSQL and backend workloads.

The Kustomize configuration deliberately does not include Secret
manifests. Workloads reference pre-existing Kubernetes Secrets.

Removing previously committed credentials does not revoke them.
Rotate any historical credentials still valid in accessible systems.
