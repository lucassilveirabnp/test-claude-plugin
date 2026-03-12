# test-claude-plugin

Skills collection for Claude Code — Python API development and logging best practices.

## Skills

### `test-claude-plugin:fastapi`
Build Python APIs with FastAPI, Pydantic v2, and SQLAlchemy 2.0 async.

Covers: project structure (domain-based), JWT auth, Pydantic v2 validation, SQLAlchemy 2.0 async, 7 documented known issues, testing with pytest + httpx.

### `test-claude-plugin:logging-best-practices`
Logging best practices focused on wide events (canonical log lines) for powerful debugging and analytics.

### `test-claude-plugin:mcp-builder`
Guide for creating high-quality MCP (Model Context Protocol) servers in Python (FastMCP) or Node/TypeScript.

### `test-claude-plugin:skill-creator`
Guide for creating effective Claude Code skills — workflows, structure, and best practices.

---

## Installation

### Option 1 — One-time install (per machine)

Run inside Claude Code:

```
/plugin marketplace add lucassilveirabnp/test-claude-plugin
/plugin install test-claude-plugin@test-claude-plugin
```

### Option 2 — Via project settings (recommended for teams)

Add to your project's `.claude/settings.json` and commit to the repo. Anyone who clones the project already has the marketplace registered and just needs to run the install once:

```json
{
  "extraKnownMarketplaces": {
    "test-claude-plugin": {
      "source": {
        "source": "github",
        "repo": "lucassilveirabnp/test-claude-plugin"
      }
    }
  }
}
```

Then install inside Claude Code:

```
/plugin install test-claude-plugin@test-claude-plugin
```

> ⚠️ Do NOT add `enabledPlugins` to settings.json — it causes Claude Code to hang on startup while trying to auto-install.

### Updating

When a new version is released:

```
/plugin marketplace update test-claude-plugin
/reload-plugins
```

---

## Development / test locally

```bash
claude --plugin-dir ./plugins/test-claude-plugin
```

Reload changes during development:
```
/reload-plugins
```

---

## Plugin structure

```
test-claude-plugin/                    ← marketplace repo root
├── .claude-plugin/
│   └── marketplace.json               ← marketplace catalog
├── plugins/
│   └── test-claude-plugin/            ← plugin lives in subdirectory
│       ├── .claude-plugin/
│       │   └── plugin.json            ← plugin manifest
│       └── skills/
│           ├── fastapi/SKILL.md
│           ├── logging-best-practices/SKILL.md
│           ├── mcp-builder/SKILL.md
│           └── skill-creator/SKILL.md
└── docs/                              ← offline copies of official plugin docs
```

---

## Maintaining the plugin

When adding or modifying skills, bump the version in **both** files:

- `plugins/test-claude-plugin/.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`

Then push and run `/plugin marketplace update test-claude-plugin` + `/reload-plugins`.
