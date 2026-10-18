# FlowForge infrastructure

This repository will contain FlowForge's Terraform/Azure infrastructure and
infrastructure delivery workflows. It is a foundation in progress, not a
deployable environment.

## Repository boundaries

- [flowforge](https://github.com/lu4mic/flowforge) owns the Java backend, workflow
  state and execution rules, application tests, runtime configuration contract
  and application image build/publishing.
- `flowforge-infra` owns infrastructure definitions and the future resource,
  identity, networking and deployment wiring that consumes that application
  contract. Application behavior belongs in the application repository.

Both repositories share the [FlowForge execution roadmap](https://github.com/lu4mic/flowforge/issues/12).
Use its dependency order and each issue's prerequisites; being in the same wave
does not remove an explicit dependency.

## Current contents and next steps

[.gitignore](.gitignore) excludes local Terraform state, saved plans, private
inputs, overrides and common credential files. Provider lockfiles and placeholder
examples remain versionable. Ignore rules are not secret scanning and do not
protect files already tracked in Git.

There are no Terraform roots/modules, version constraints, backend configuration
or GitHub Actions workflows yet. Nothing here currently provisions or deploys
resources, and no Terraform validation or Azure environment is claimed.

Start with [#1 — Repository bootstrap](https://github.com/lu4mic/flowforge-infra/issues/1).
Its remaining work includes the minimal layout, deliberate Terraform/provider
constraints, naming conventions and placeholder configuration. Keep one
ephemeral demo environment rather than duplicating environments prematurely.
Then follow [#2 — Structure and remote state](https://github.com/lu4mic/flowforge-infra/issues/2)
and [#13 — Validation CI](https://github.com/lu4mic/flowforge-infra/issues/13)
when their prerequisites are ready. Do not add empty modules to imply progress.

## Security and cost boundary

Static code/documentation review must not require an Azure account, real
credentials or paid services. Never commit state, saved plans, real input values,
private keys or secrets. Use placeholders in examples and review staged diffs;
ignored local files are not a secure secret store.

Automatic CI is intended for validation, not cloud provisioning. Infrastructure
apply, application deployment and destroy are future manual, protected actions.
No PR or push should provision resources automatically. A workflow being manual
does not by itself authorize an account operation: explicit owner authorization
is still required. Current automated work has no authorization to access Azure
accounts or run Terraform apply, destroy, import or state operations.

The planned demo is ephemeral: record the demonstration, then perform the
owner-authorized teardown and [cleanup verification](https://github.com/lu4mic/flowforge-infra/issues/17).
Remote-state storage and any retained backups/logs must be included in that
future cleanup inventory; teardown evidence is not available yet.

## Authentication design — not configured

No Azure identity, federated trust, role assignment or authenticated workflow is
configured by this repository. The intended boundaries are:

| Work | Authentication boundary |
| --- | --- |
| PR/push static validation | No Azure login, OIDC token permission or remote-state access |
| Future Azure-backed plan | Separately reviewed least-privilege access; account access is not currently authorized |
| Future apply, deployment and destroy | Manual protected jobs with owner authorization; do not reuse broad write permissions for planning |

Future GitHub jobs should use OIDC federation rather than stored client secrets.
Grant `id-token: write` only to an authorized job that needs a token; it enables
token requests, not Azure resource permissions. Restrict trust to the intended
repository and approved ref/environment, and review Azure role scopes separately.
Sharing a GitHub Project does not give these two repositories shared cloud trust.

Do not copy a legacy name-only `sub` example into a future trust policy. GitHub
documents immutable owner/repository IDs in default subjects for repositories
created after July 15, 2026. Before setup, confirm each repository's actual subject
format, selected protected environment, issuer and audience. Do not emit or log
tokens just to populate this document. See the official
[OIDC reference](https://docs.github.com/en/actions/reference/security/oidc) and
[Azure OIDC guidance](https://docs.github.com/en/actions/how-tos/secure-your-work/security-harden-deployments/oidc-in-azure).

Local account authentication also requires separate owner authorization. Missing
OIDC configuration must not trigger a fallback to a developer's Azure CLI session
or a client secret. Backend access/locking permissions and concrete identity
choices remain in [#2 — Structure and remote state](https://github.com/lu4mic/flowforge-infra/issues/2)
and [#7 — Managed identity/RBAC](https://github.com/lu4mic/flowforge-infra/issues/7);
these notes are not login or provisioning instructions.

## Checks available now

From a checkout, Git-only review needs no Terraform installation or cloud login:

```bash
git diff --check
git diff --cached --check
git status --short
git diff --cached
```

Check new files as well as tracked changes before committing. Terraform
format/validation commands and their tool versions will be documented alongside
the actual configuration; there is no Terraform configuration to validate yet.
