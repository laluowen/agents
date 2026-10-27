---
name: terraform
description: Owen's Terraform/OpenTofu preferences. Use whenever writing, reviewing or refactoring Terraform/OpenTofu. Currently covers module policy (public registry, git or local modules).
---

## Modules

Default: raw provider resources. Public modules are often over-broad, lag the provider, and couple state addresses to their internals.

- Never add a public module for a single resource or a simple group (S3 bucket, security group, IAM role, log group). Write the provider resources.
- A public module may be proposed only for wiring that is easy to get wrong by hand (e.g. VPC routing/NAT, EKS with IAM/OIDC/add-ons), and only if it is from HashiCorp or a verified publisher with a recent release.
- Before adding any public module: say why, name the module and version, and ask me. Do not add it unprompted. Pin an exact version.
- My own or internal modules: fine to use and extend.
- Module already in the repo: follow its pattern. Don't swap it for raw resources or another module without asking.
- Removing a module changes resource addresses: use `moved` blocks and say so.
