---
name: go
description: Owen's Go project conventions. Use when creating, designing, structuring, reviewing or substantially editing any Go project or package, or setting up Go tooling (lint, release, tests).
---

Golden standard: `~/Documents/git/laluowen/Propsy`. Read its `AGENTS.md` and `PLAN.md` first when designing a new project, not memory. `~/Documents/git/laluowen/ChipIn` is the older sibling.

- Small, boring, dependency-light. Pure Go, `CGO_ENABLED=0`, static binary, distroless image.
- `cmd/<app>` is wiring only. Logic in `internal/`: config, transport-agnostic handler, stores, one adapter per transport.
- Interfaces small, defined where used (`handler.SlackAPI`, `store.Repository`). Logic never imports a concrete client. Hand-written fakes, no mocking frameworks.
- Several implementations (stores): one contract test runs against all. Share pure logic, don't duplicate.
- Config from env vars, `.env` fallback. Fail at startup with a clear message. Never silently discard data.
- Tests use no live services: `httptest`, in-memory store, emulator that skips itself when its env var is unset.
- Before finishing: `go test -race ./...` and `golangci-lint run`.
- Tooling: `golangci-lint` v2 (gofumpt, gci), GoReleaser, release-please, `.env.example`. Keep `PLAN.md` (design, decisions) and `AGENTS.md` (agent notes) current with the code.
