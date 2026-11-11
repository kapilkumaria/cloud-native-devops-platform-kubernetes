# Secure Kubernetes Secret Provisioning

## Purpose and Current Status

This repository references Kubernetes Secrets but does **not** store Secret values or include Secret manifests in its Kustomize resources.

> **November 10, 2026:** This document describes the required provisioning procedure for the updated manifests. Secret creation and complete workload startup in a newly rebuilt Project 1 EKS cluster have **not** yet been revalidated.

## Namespace

Application workloads run in `cloud-platform`. Create the namespace before provisioning credentials:

```bash
kubectl create namespace cloud-platform --dry-run=client -o yaml | kubectl apply -f -
```

Confirm the intended Kubernetes context before executing commands against a live cluster:

```bash
kubectl config current-context
```

## Required Secret Names and Keys

### `backend-secret`

The backend requires:

- `FIRST_SUPERUSER`
- `FIRST_SUPERUSER_PASSWORD`
- `POSTGRES_PASSWORD`
- `SECRET_KEY`

### `postgres-secret`

PostgreSQL requires:

- `POSTGRES_DB`
- `POSTGRES_USER`
- `POSTGRES_PASSWORD`

`POSTGRES_PASSWORD` must match across the two Secrets. Database name and username must agree with the application's non-secret PostgreSQL configuration. Use strong, unique credentials for each environment. Keep the same `SECRET_KEY` across backend replicas and restarts unless intentionally rotating it.

**Recheck deployment environment-variable references against actual manifests if the Secret schema changes.** Do not infer from this document that AWS Secrets Manager or External Secrets Operator is already installed.

## Recommended Production Approach

Use a centrally managed secret store such as **AWS Secrets Manager**, with a properly configured retrieval mechanism (for example, External Secrets Operator), IAM least privilege and documented rotation/recovery procedures. This is a recommendation, **not** a claim that this Project 1 environment already implements those integrations.

## Manual Provisioning for Controlled Testing

An authorized deployment operator can create Secrets using `kubectl create secret generic` with secure input handling. Avoid passing secret plaintext directly in command-line literals: these can be exposed in command history, logs or process listings.

One possible controlled procedure is to provision short-lived files with permissions restricted to the current user, then create Kubernetes Secrets using `--from-file=KEY=path`. Treat those files as sensitive, avoid placing them inside the Git repository, and remove them securely according to your organization's credential-handling standards.

**Illustrative template (paths are placeholders, not populated credentials):**

```bash
kubectl -n cloud-platform create secret generic postgres-secret \
  --from-file=POSTGRES_DB=/secure/location/postgres-db \
  --from-file=POSTGRES_USER=/secure/location/postgres-user \
  --from-file=POSTGRES_PASSWORD=/secure/location/postgres-password

kubectl -n cloud-platform create secret generic backend-secret \
  --from-file=FIRST_SUPERUSER=/secure/location/first-superuser \
  --from-file=FIRST_SUPERUSER_PASSWORD=/secure/location/first-superuser-password \
  --from-file=POSTGRES_PASSWORD=/secure/location/postgres-password \
  --from-file=SECRET_KEY=/secure/location/jwt-secret-key
```

Each `--from-file=KEY=path` reads the file contents as the Secret value; if your files include trailing newlines, those newlines become part of the value. Prepare and validate files accordingly. File-based provisioning creates Kubernetes Secret objects in the cluster, so apply appropriate Kubernetes RBAC and encryption-at-rest policies.

For re-runs, `kubectl create secret` fails when the Secret already exists. Use an approved idempotent provisioning workflow rather than embedding values in Git or displaying decoded Secrets in a terminal transcript.

## Non-Sensitive Verification

After provisioning:

```bash
kubectl get secret backend-secret postgres-secret -n cloud-platform
kubectl describe secret backend-secret -n cloud-platform
kubectl describe secret postgres-secret -n cloud-platform
```

`kubectl describe secret` displays names and sizes, not decoded values. Limit access to sensitive metadata as appropriate and **do not paste** Secret YAML or decoded values into public issues or documentation.

For deployment and troubleshooting, confirm each manifest's `secretKeyRef` references the correct Secret and key. A missing or mismatched key can prevent container startup.

## Deployment Order

1. Verify correct cluster/account/context and `cloud-platform` namespace.
2. Ensure EBS CSI and Traefik prerequisites are in place.
3. Provision `postgres-secret` and `backend-secret` securely.
4. Confirm Secret names and key references without exposing values.
5. Apply or reconcile the Kubernetes application manifests through the intended GitOps workflow.
6. Validate PostgreSQL PVC binding, backend startup and API routing.

The Kustomize configuration deliberately excludes Secret manifests. **Do not claim Kustomize creates these Secrets.**

## Credential Hygiene and Rotation

- Never commit real passwords, JWT signing keys, Secret manifests containing values or kubeconfig credentials to Git.
- Kubernetes Secret data is base64-encoded, **not inherently encrypted** by that encoding.
- Use least-privilege RBAC, appropriate secret encryption and audit controls.
- Removing leaked credentials from a repository does not revoke them; rotate any still-valid historical credentials.
- Never reuse example passwords from older historical test commands in real environments.
