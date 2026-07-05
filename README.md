# 🧩 Pi Starter

**Portable Pi agent resources — subagents, MCP servers, and settings — a repo you mount as the `.pi/` folder of any project.**

This repo root **is** the bundle: submodule or copy it to `<your-project>/.pi/` and everything lands exactly where Pi looks for it.

---

## What is this?

A reusable [Pi](https://earendil-works.github.io/pi/) resource bundle you clone, submodule, or copy into **any project** — research repo, web app, CLI tool, data pipeline — to instantly bootstrap:

| Resource | What it gives you |
|----------|------------------|
| **Subagents** | Focused child agents: scout (codebase recon), worker (implementation), reviewer (code audit) |
| **MCP servers** | Project-local tool integrations (e.g. `context7` knowledge server) |
| **Settings** | Model routing, thinking levels, tool budgets — per-project, overriding global config |

> **The idea:** Your project changes. Your Pi setup shouldn't have to. Keep it in its own git, pull updates without touching your project code.

---

## ✅ Prerequisites

On each machine that will use this template, install Pi and the subagents extension (one-time, global):

```bash
# 1. Pi itself
#    See https://github.com/earendil-works/pi for install options

# 2. pi-subagents — enables the `subagent` delegation tool
pi install npm:pi-subagents
```

After that, pull this `.pi/` bundle into any project on that machine and the agents + MCP servers are live. The `.pi/` folder defines **roles and config** — it does not install the `subagent` engine itself. That lives in `~/.pi/agent/npm/node_modules/pi-subagents` and is registered in `~/.pi/agent/settings.json`.

---

## 📦 Structure

```
pi-starter/                  ← This repo — mounts as .pi/ in your project
├── README.md               ← You are here
├── .gitignore              ← Ignores .pi-subagents/ artifacts, .env, logs, OS junk
├── .env.example            ← Template for CONTEXT7_API_KEY (copy to project root as .env)
├── INSTRUCTIONS.md         ← Delegation rules & config invariants (inherited by parent session)
├── mcp.json                ← Project-local MCP servers (context7 + yours)
├── settings.json           ← Subagent model routing & overrides
└── agents/
    ├── scout.md            ← Fast codebase recon (read-only, cheap model)
    ├── worker.md           ← General-purpose implementation agent
    └── reviewer.md         ← Code review specialist (read-only)
```

Once mounted at `.pi/`, your project sees `.pi/agents/`, `.pi/mcp.json`, `.pi/settings.json` — the paths Pi auto-discovers.

> **Note:** running subagents creates a `.pi-subagents/` folder (run transcripts and metadata) in the project root. It's regenerated runtime output — add it to your project's `.gitignore`.

---

## 🚀 Using this in a project

### Option 1: Git submodule (recommended — easy updates)

```bash
cd your-project
git submodule add https://github.com/rvibek/pi-starter.git .pi
git submodule init
```

Later, pull the latest template:
```bash
git submodule update --remote .pi
```

After cloning the main project on a new machine:
```bash
git submodule update --init
```

### Option 2: Standalone copy

```bash
cd your-project
git clone --depth 1 https://github.com/rvibek/pi-starter.git .pi
rm -rf .pi/.git
```

### Option 3: Fetch via tarball (no git)

```bash
cd your-project
mkdir -p .pi
curl -sL https://github.com/rvibek/pi-starter/tarball/main | tar -xz --strip=1 -C .pi
```

### After install (all options): expose mcp.json

Pi reads project MCP config from `<project-root>/mcp.json`, not from `.pi/`. Symlink it once:

```bash
ln -s .pi/mcp.json mcp.json
```

Agents and settings need no such step — Pi discovers `.pi/agents/` and `.pi/settings.json` directly.

---

## 🧠 Subagents

Subagents are focused child Pi sessions. They keep your main context clean.

| Agent | Model | Thinking | Tools | Best for |
|-------|-------|----------|-------|----------|
| **scout** | DeepSeek V4 Flash | off | read, grep, find, ls | "Find where auth is handled", "Map the API routes" |
| **worker** | Kimi K2.7 Code | medium | read, write, edit, bash, grep, find, ls, **subagent** | "Implement the login endpoint", "Refactor this module" |
| **reviewer** | GLM 5.2 | high | read, grep, find, ls, bash | "Review my last diff for security issues" |

> **Model provider:** this setup routes all agents through [OpenCode Zen](https://opencode.ai) models (`opencode-go/...` identifiers) — GLM, Kimi, DeepSeek, MiMo, MiniMax, Qwen. You need OpenCode Zen access configured in Pi for these to resolve. To use a different provider, swap the `model` / `fallbackModels` values in `.pi/settings.json` and the agent frontmatter.
>
> ⚠️ Zen models on the Anthropic-style `/v1/messages` endpoint (MiniMax, Qwen) have been seen to 404 in subagent child context — probe with a 1-token task before relying on them. The `/v1/chat/completions` family (GLM, Kimi, DeepSeek, MiMo) works.

### Usage inside Pi

```text
> Use scout to explore the project structure.
> Use worker to set up a Next.js app with Tailwind.
> Use reviewer to audit my last commit.
> Dispatch parallel reviewers: one for correctness, one for tests.
```

### Dogfood: review this template with itself

```text
> Use scout to map the repo, reviewer to audit .pi/mcp.json and .pi/settings.json, then brief me.
```

That's the loop: **scout finds → reviewer checks → you synthesise.** The same line works for any codebase you pull this bundle into — recon, audit, brief. It's also the cheapest way to make sure you (or your parent agent) actually delegate instead of reading 10 files solo.

### Customising agents

Override model, thinking, or tools **without editing agent files** — use `.pi/settings.json`:

```json
{
  "subagents": {
    "agentOverrides": {
      "scout": {
        "model": "opencode-go/deepseek-v4-flash",
        "thinking": "off"
      }
    }
  }
}
```

### Adding your own

Create a new `.md` file in `.pi/agents/` with frontmatter:

```markdown
---
name: oracle
description: Strategic advisor — second opinions on architecture
tools: read, grep, find, ls, bash
model: opencode-go/deepseek-v4-pro
thinking: high
systemPromptMode: replace
defaultContext: fork
---
You are an experienced architect. Challenge assumptions, suggest alternatives.
```

No registration needed — Pi auto-discovers `.md` files in that folder.

---

## 🔌 MCP Servers

Pi's MCP extension reads exactly two config files:

1. `~/.pi/agent/mcp.json` — global
2. **`<project-root>/mcp.json`** — project-local (note: project root, **not** `.pi/`)

The bundle ships `mcp.json`, but after mounting at `.pi/` it sits at `.pi/mcp.json` where Pi won't look. **One-time step after install** — symlink it to the project root:

```bash
cd your-project
ln -s .pi/mcp.json mcp.json
```

(Or copy it, if you want project-specific edits that don't track the template.) On first launch Pi asks to trust the project `mcp.json` — accept.

### context7 server

The template ships with `context7` — a remote knowledge/context MCP server accessed via [mcp-remote](https://www.npmjs.com/package/mcp-remote).

**No changes needed** — the config in `.pi/mcp.json` is ready to use. But Pi reads **`process.env` directly** — it does not auto-load `.env` files. So you must export the variable before launching Pi:

```bash
# Option A: copy the example into your project root, then export it in your shell
cp .pi/.env.example .env
# edit .env with your real key, then:
set -a; . ./.env; set +a
pi
```

```bash
# Option B: inline for a single session
CONTEXT7_API_KEY=ctx7sk-xxxxxxxx pi
```

```bash
# Option C: use direnv (recommended for repeated use)
# add `. .env` to .envrc, then `direnv allow`
```

The `${CONTEXT7_API_KEY}` variable in `mcp.json` is resolved by **mcp-remote** at runtime from the inherited child-process environment. No space after the colon in the header — `mcp-remote`'s arg parser chokes on spaces, which is also why the `${VAR}` pattern is used instead of shell-style substitution.

**How it works:**
- `npx -y mcp-remote` — downloads and runs the remote-MCP proxy on the fly
- `--header CONTEXT7_API_KEY:${CONTEXT7_API_KEY}` — passes auth as an HTTP header (variable resolved by mcp-remote from the child process environment)

To disable the server temporarily, add `"disabled": true` to its entry.

### Adding more servers

```json
{
  "mcpServers": {
    "my-db-tool": {
      "command": "python",
      "args": ["-m", "my_mcp_server"],
      "disabled": false
    }
  }
}
```

---

## ⚙️ Settings

`.pi/settings.json` overrides the global `~/.pi/agent/settings.json` **for that project only**.

| Override field | Type | Description |
|---------------|------|-------------|
| `model` | string | Model identifier |
| `thinking` | string | `off`, `minimal`, `low`, `medium`, `high`, `xhigh` |
| `fallbackModels` | string[] | Ordered fallback list |
| `tools` | string[] | JSON array of tool names (e.g. `["read", "grep", "bash"]`) — never a comma-separated string |
| `systemPromptMode` | string | `append` or `replace` |
| `defaultContext` | string | `fork` or `fresh` |
| `inheritProjectContext` | boolean | Child inherits project context? |
| `inheritSkills` | boolean | Child inherits loaded skills? |
| `agentScope` | string | `user` \| `project` \| `both` (default). Project wins on name collisions — your `.pi/agents/*.md` shadow builtins like `oracle`/`planner`/`researcher`. |
| `disabled` | boolean | Hide an agent from discovery and `list` output (reversible) |

---

## 🧪 What about skills?

Project-local skills go in `.pi/skills/`:

```
.pi/skills/
└── my-skill/
    ├── SKILL.md
    └── ...
```

Pi auto-discovers them. Use via `/skill:my-skill` in the editor. The template doesn't ship with any — add your own.

---

## 🌐 Git workflow

### Pushing this template to GitHub

```bash
# From the repo root (where this README lives)
git init
git add .
git commit -m "Initial Pi starter — subagents, MCP, settings"
git remote add origin https://github.com/rvibek/pi-starter.git
git push -u origin main
```

### Updating the template

```bash
# Make changes to agents, MCP, or settings
git add .
git commit -m "Add new agent, tweak models"
git push
```

All projects using `git submodule` can then:
```bash
cd your-project
git submodule update --remote .pi
```

---

## 🔧 Extending the template

Fork this repo, then (paths are repo-root here — they appear under `.pi/` once mounted in a project):

- **Add agents** — create `agents/<name>.md`
- **Add MCP servers** — edit `mcp.json`
- **Tweak models/thinking** — edit `settings.json`
- **Add project-local skills** — create `skills/<name>/SKILL.md`
- **Update documentation** — edit this `README.md`

Submit a PR if you think others would benefit!

---

## 📁 Full reference

```
pi-starter/                   ← Repo root == the bundle — mounts at .pi/ in your project
├── README.md                ← This file — visible on repo homepage and in .pi/
├── .gitignore               ← .pi-subagents/ artifacts, .env, logs, OS junk
├── .env.example             ← CONTEXT7_API_KEY template (copy to project root as .env)
├── INSTRUCTIONS.md          ← Delegation rules & config invariants
├── mcp.json                 ← Project-local MCP servers
├── settings.json            ← Subagent overrides
└── agents/
    ├── scout.md             ← Codebase recon agent
    ├── worker.md            ← Implementation agent
    └── reviewer.md          ← Code review agent
```

---

> Built for [Pi](https://earendil-works.github.io/pi/) · Uses [pi-subagents](https://github.com/nicobailon/pi-subagents) · Worker's minimal-code ladder adapted from [ponytail](https://github.com/DietrichGebert/ponytail)
