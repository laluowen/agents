---
name: opencode-project-cleanup
description: Clean up OpenCode projects and sessions in the local SQLite DB. Use when listing, exporting, or deleting OpenCode projects, pruning old sessions, or reclaiming space from ~/.local/share/opencode/opencode.db.
---

# OpenCode project cleanup

Follow these steps in order. Do not skip the backup or the confirmation.

## Rules

- Get user confirmation before any delete. Show the list first.
- Do not stop the OpenCode service. Use the online backup, which is safe while the DB is open.
- Back up before every write.
- Export sessions before deleting them when the user asks for a backup.
- Run deletes in one transaction with `pragma foreign_keys=ON`. The `sqlite3` CLI leaves foreign keys off by default, so cascades do not run without this pragma.
- Never edit the DB by hand. Never use `cp` on the DB while it is open.

## Find the DB

```sh
opencode debug paths db
```

Default: `~/.local/share/opencode/opencode.db`. `OPENCODE_DB` can override it.

## Tables

- `project`: one row per project. `worktree` is the path.
- `session_v2`: live sessions. Use this table for counts and listings.
- `session`: legacy sessions. Some rows exist in both tables, and some only in one. Check both.
- `session_message`, `session_part`: children of `session_v2`. They cascade on delete.

Counts from `session` alone are wrong. Always count `session_v2`, and check `session` too.

## 1. Back up

```sh
sqlite3 "$DB" ".backup '$TMPDIR/opencode/opencode-backup-<label>.db'"
sqlite3 "$TMPDIR/opencode/opencode-backup-<label>.db" "pragma integrity_check;"
```

Expect `ok`. Tell the user the backup path. `$TMPDIR` is a temp dir, so suggest a durable copy for keepers.

## 2. List projects with session counts

```sh
sqlite3 -header -column "$DB" "select p.id, p.worktree, (select count(*) from session_v2 v where v.project_id=p.id) as sessions from project p order by p.time_active;"
```

Show a numbered list. Mark archived projects (`time_archived` not null). Show each project's sessions as well when the user asks for detail.

## 3. Export sessions (optional)

Export a project's sessions before deleting it:

```sh
opencode session export <session-id> > <dir>/<name>.json
```

- Name each file from the session title: lowercase, non-alphanumeric to `-`, trimmed, max 80 characters. If two names collide, add `-2`, `-3`.
- Use `--sanitize` for sessions with secrets, prod data, or key material. Ask the user first.
- Validate each file with `jq -e '.info.id' <file>`.
- Put files in `~/opencode-sessions/<project-name>/`.

## 4. Delete

Get the list of project IDs from the user. Then run one transaction:

```sh
sqlite3 "$DB" "pragma foreign_keys=ON; begin; delete from project where id in ('<id1>','<id2>'); select 'deleted', changes(); commit;"
```

Deleting a project cascades to its sessions, messages, parts, and project directories. The folder on disk is not touched.

## 5. Verify

```sh
sqlite3 "$DB" "select 'projects', count(*) from project; select 'v2', count(*) from session_v2; select 'v1', count(*) from session; select 'orphan messages', count(*) from session_message where session_id not in (select id from session_v2); pragma integrity_check;"
```

Expect:
- Project count dropped by the number deleted.
- `orphan messages` is 0.
- `integrity_check` returns `ok`.

Report before and after counts.

## Restore

Copy the rows back from the backup. Do not replace the live DB while the service is running.

## Known limits

- Do not delete the active session. Deleting it breaks the running session with a foreign key error (see anomalyco/opencode#37975).
- Sessions with `time_archived` set are still deleted with their project.
- `ocsm` (neuronalism/opencode-session-manager-cli) does export-then-delete and Markdown output. It was tested on OpenCode v1.18, so check it against v2 before use.
