# Project instructions (inherited by the parent Pi session)

These rules govern how the parent agent should use the bundled subagents. They exist to prevent the retry-until-it-magically-works loop that burned a session debugging `minimax-m3` 404s.

## Delegation discipline

- **Probe before you diagnose, diagnose before you audit.** Before dispatching a heavy task on a model you haven't run this session, send a 1-token probe (`Reply with: OK <model> online`).
- **Retry a failed `subagent` call at most twice.** On the second identical failure, stop retrying and inspect the call envelope with `bash`/`read` — the input is wrong, not the universe.
- **Never retry identical calls.** Change exactly one variable per retry: the model, the tools, or the task weight. Zero-variable retries are forbidden.
- **Frontmatter model pins override runtime `model:` overrides.** If the frontmatter pins a broken model, *edit the file*; don't keep passing `model:` in the call — it will be ignored.
- **First run on a freshly-changed model = short task only.** Scalp up to the full audit only after the smoke test passes.

## Config invariants

- `context7` in `mcp.json` must NOT contain `"transport"` or `"lifecycle"` fields — they are non-standard and not read by Pi. Use `"command"`/`"args"` (stdio is implied) or `"type": "streamable-http"` for remote servers.
- Pi's MCP extension reads `<project-root>/mcp.json` and `~/.pi/agent/mcp.json` only — it does NOT read `.pi/mcp.json`. The bundle's `mcp.json` must be symlinked to the project root (`ln -s .pi/mcp.json mcp.json`).
- `tools` arrays in `.pi/settings.json` must be JSON arrays of strings, never comma-separated strings.
- Pi does NOT auto-load `.env` files. Env vars must be exported before launching `pi` (`set -a; . ./.env; set +a`, `direnv`, or inline `VAR=val pi`).

## Agent routing

- Project agents in `.pi/agents/` shadow builtins of the same runtime name (project wins). The bundled builtins `oracle`, `planner`, `researcher`, `context-builder`, `delegate` are available even without a local `.md` — dispatch them by name.
- Optional: finalize every code change with a reviewer subagent on the diff.