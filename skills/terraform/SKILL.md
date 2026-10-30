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

## Applying

- Never run `apply` (including `-auto-approve`, `-target`, and `apply` on a saved plan) or `destroy`. I run applies myself. Give me the exact command and what to expect.
- Same for state-changing commands: `state rm`, `state mv`, `untaint`, `taint`, `import`, `force-unlock`. Ask first.
- Read-only commands are fine: `fmt`, `validate`, `plan` (without `-out` applied), `show`, `state list`/`show`, `output`, `providers`.
