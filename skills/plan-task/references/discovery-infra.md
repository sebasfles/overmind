# Discovery track: infrastructure

Use as a checklist. Ask only what the docs and code do not answer.
Infra changes are often part of a feature task (a new queue, a bucket, a secret); load this track alongside the platform track when that happens.

## Fit

- Which environments: local, dev, staging, prod. In which order, and what gates promotion.
- IaC or console: everything through the repo's IaC (Terraform or equivalent); manual changes are out.
- Which stack or module of the IaC owns the change.

## Resources

- New or modified resources: compute, storage, queues, databases, DNS, certificates, CDN.
- Sizing and scaling: expected load, limits, autoscaling.
- Lifecycle: retention, backups, deletion protection.

## Configuration and secrets

- New env vars and where each environment gets its value.
- Secrets: created in the secrets manager, never in the repo or in tfvars committed to git.
- Rotation and who can read them.

## Access and network

- IAM or RBAC: which principals gain which permissions; least privilege.
- Network exposure: public or private, security groups, VPN, allowed origins.
- Service-to-service auth.

## Delivery

- CI/CD changes: new pipelines, steps, deploy targets, required checks.
- Migration or rollout plan: order of operations, feature flag, rollback path.
- Downtime expected or not.

## Operations and cost

- Logging, metrics, alerts, dashboards for the new resource.
- Cost estimate and what drives it.
- Anything that touches `prod` is called out explicitly in Risks and confirmed with Sebastian.
