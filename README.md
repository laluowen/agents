# agents

Personal agent config, tracked here and symlinked into `~/.agents/`.

## Contents

- `AGENTS.md`: global agent instructions. Symlinked to `~/.agents/AGENTS.md`.
- `skills/`: agent skills. Each subdirectory is one skill.
  - `skills/go/`: Go project conventions.
  - `skills/terraform/`: Terraform/OpenTofu preferences.

## Setup

```sh
ln -s ~/Documents/git/laluowen/agents/AGENTS.md ~/.agents/AGENTS.md
ln -s ~/Documents/git/laluowen/agents/skills/<skill> ~/.agents/skills/<skill>
```
