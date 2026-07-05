# .pi/ — Resource Bundle

This folder is the Pi resource bundle from the **pi-starter** template. It provides subagents, MCP servers, and settings. See the root [`../README.md`](../README.md) for full documentation.

## Contents

```
.pi/
├── INSTRUCTIONS.md ← Delegation rules & config invariants (inherited by parent session)
├── mcp.json        ← Project-local MCP servers (context7 via mcp-remote)
├── settings.json   ← Subagent model overrides
└── agents/
    ├── scout.md    ← Codebase recon (read-only, cheap model)
    ├── worker.md   ← General implementation agent
    └── reviewer.md ← Code review specialist (read-only)
```

## Quick reference

| Agent | Tools | Model |
|-------|-------|-------|
| scout | read, grep, find, ls | deepseek-v4-flash |
| worker | read, write, edit, bash, grep, find, ls, subagent | kimi-k2.7-code |
| reviewer | read, grep, find, ls, bash | glm-5.2 |

## To update this bundle

```bash
# If installed as submodule:
git submodule update --remote .pi
```

## context7 MCP

See `../README.md` for setup. TL;DR: add to `.env` in project root:

```
CONTEXT7_API_KEY=ctx7sk-xxxxxxxx
```

## To add your own agent

Create `.pi/agents/your-agent.md` with frontmatter. It's auto-discovered.
