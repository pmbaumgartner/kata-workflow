# Kata CLI Gotchas

Read this reference when scripting Kata output, using low-frequency mutations,
configuring optional interfaces, or diagnosing local operations. The installed
`kata quickstart` and command help remain normative.

## Compatibility And Scope

The skill examples were checked against Kata v0.14.3. Verify the binary with
`kata version --json`; inspect `name`, `version`, `kata_api_version`, and
`agent_format`.

Refs are short ULID-derived IDs such as `abc4`. Cross-project refs look like
`kata#abc4`; full ULIDs also resolve. Legacy numeric refs do not.

`--label` and `--no-label` are repeatable. `list`, `ready`, and `next`
support cross-project `--all`; `search` remains project-scoped, so pass
`--project <name>` outside the bound project.

Search is lexical without embeddings and automatically hybrid when
`[search.embeddings]` is configured. Explicit `--hybrid` and `--semantic`
fail without embeddings; keep lexical search and vary the query instead.

## Output And JSON

Do not parse `--agent` output in scripts; it is concise, not a stable machine
contract. Use `--json` and project with `jq`.

- Do not pipe full JSON to `head`; project the required field first.
- `list` and `ready` return `.issues[]`.
- `search` returns `.results[]`, each with `issue`, `score`, and
  `matched_in`.
- In `show --json`, `issue` holds scalar fields. `links`, `labels`, and
  `comments` are siblings. Read relationships from `.links[]`.

## Mutation Edge Cases

- `edit`, `close`, `reopen`, `claim`, `assign`, `unassign`, and
  `label add/rm` accept `--comment TEXT`.
- The mutation lands before its follow-up comment. If the comment fails, retry
  with `kata comment <ref> --body ...`.
- Add `needs-review` with `kata label add <ref> needs-review`; `kata edit
  --label` is invalid even if older help text suggests it.
- `--remove-parent <ref>` is strict: the ref must match the current parent.
  Other `--remove-*` flags are idempotent.
- Preview a move with `kata move <ref> <project> --dry-run --agent`. A move
  preserves history and links but assigns a new short ref in the target project.

## Optional Interfaces And Hooks

- Use `kata show <ref> --render` only in a human-facing terminal. Redirects,
  pipelines, `--agent`, and `--json` should remain unrendered.
- Use `kata tui [<ref>]` or `kata ui [<ref>]` for human supervision.
- Use `kata mcp serve` only when configuring a bound project as an MCP server.
- `kata init --with-codex-hooks` installs Codex CLI attention wiring. In
  v0.14.3 it installs the session-start half; keep end-of-session updates in the
  launcher until Codex exposes a stable session-end hook.

## Operations And Less-Common Closes

Use `kata daemon status --agent` for diagnosis and `kata daemon restart
--agent` only when a clean restart is needed.

For long-running sessions, resume polling with `kata events --after <cursor>
--agent` or stream with `kata events --tail --agent`. Use `kata digest
--since 24h --agent` for a human-scale handoff summary.

```bash
kata close abc4 --duplicate-of d4ex --message "Same Safari race condition." --agent
kata close abc4 --superseded-by d4ex --message "Replaced by broader scope." --agent
kata close abc4 --wontfix --message "Out of scope after contract review." --agent
kata close abc4 --audit-no-change --message "Reviewed schema; no change needed." \
  --evidence "no-change-audit:schema unchanged after review" \
  --reviewed internal/db/schema.sql --agent
```
