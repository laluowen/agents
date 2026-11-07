# Global agent instructions

Apply in every project. Project `AGENTS.md` adds to this and wins on conflict.

## Response style

- Brief. Lead with the answer. No filler, pleasantries, preamble or recaps.
- Fragments OK. Short words (fix, not "implement a solution for").
- Code, commands, paths, URLs, errors: exact, never shortened.
- Full sentences only where step order or a warning needs them.
- Expand when I ask for explanation or detail.

## Skills

- **Go work** (new project, design, structure, review, tooling): load the `go` skill first.
- **Terraform/OpenTofu** (generate, review, refactor, migrate, CI pipelines): load the `terrashark` and `terraform` skills first.
- **Machine-facing text** (tool descriptions, error messages, prompts or instructions for other agents or subagents, status reports): load the `asd-ste100-skill` skill and write in its controlled English: one meaning per word, active voice, one instruction per sentence. Human-facing replies stay as in Response style.

## Working with me

- Take my setup claims as true. Don't re-test; if wrong, tell me.
- Ask before widening access: `gh` token scopes, repo/org settings, secrets. If a scope refresh is needed, tell me, run it, give me the one-time code to approve from my phone.
- Commit or push only when I ask. Secrets live in gitignored `.env`: never print or commit.
- Tools are managed by mise: check `mise ls` (`~/.config/mise/config.toml`) before saying something isn't installed.
- Versions. Global tools (`~/.config/mise/config.toml`): `latest` is fine, since I upgrade them interactively. Project `mise.toml`: never `latest`. Pin the exact version (use `mise ls-remote <tool>` for the newest tag) so CI and everyone else get the same tool, and let Renovate upgrade it. If a tool needs `rename_exe`, add it alongside the pin.
- Backend for mise tools: prefer `github:` or `gitlab:` over `aqua:`, until `packslip:` is widely used. Use the `aqua:` backend only when no `github:`/`gitlab:` source works.
- My checkouts: `~/Documents/git/laluowen/<repo>`.
- `~/Documents/git/misc/`: consumption-only clones. Plain, unauthenticated. No remotes, no pushing.
- Public agent skills go in `~/Documents/git/misc/skills/<name>`, with the skill directory symlinked into `~/.agents/skills/`.

## Git and GitHub

Normal git here uses 1Password SSH keys; every use needs a fingerprint touch. So **never `git push`, `git fetch` or `git pull` against an SSH `origin`, and never plain `git commit`** (it blocks waiting for me). Use the prompt-free path:

- **HTTPS via `gh`** is the default (`gh` is the credential helper for github.com). `gh repo clone` and HTTPS `git clone` just work, private repos included.
- **Remote `gh`** (HTTPS) is for agents' fetch/push. `origin` is SSH, for me only. If a repo lacks `gh`: `git remote add gh https://github.com/<owner>/<repo>.git`.
- **Upstreams stay on `origin`.** My branches, including `main`, track `origin/...`. Agents never change an upstream to `gh/...` (no `git branch -u gh/...`, no `--set-upstream` to `gh`): that makes `git pull` and `git status` use the agent path on my checkouts. Push with `git push gh <branch>` and leave tracking alone.
- **Cloning:** clone over HTTPS, rename that remote to `gh`, add SSH `origin` for the same repo.
- **Commits:** `AGENT_MODEL="<provider>/<model>" git agent-commit -m "type(scope): subject"`. Same args as `git commit`; signs with the agent key, adds a `Co-authored-by: OpenCode <version> (<model>) <noreply@anthropic.com>` trailer. If you're a different agent or the alias is missing, say so; don't fall back to plain `git commit` or mislabel the trailer.
- **Branches/PRs:** work on a branch, push to `gh`, `gh pr create`. I merge. Conventional Commits (`feat:`, `fix:`, `docs:`, `ci:`, `chore:`; `!` or `BREAKING CHANGE:` footer), since releases are generated from them.
- **Tokens:** pushes touching `.github/workflows/` need the `workflow` scope. Don't add scopes without asking.

### lalu-uk org

- PR into `main` required; no direct push.
- Releases are immutable: published assets and tag frozen, tags never reused. Bad release = new version. Pipelines: draft, attach all assets, publish.
- Actions pinned by commit SHA with a version comment (Renovate maintains them).
