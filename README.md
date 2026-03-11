# test-claude-plugin

Skills collection for Claude Code with Python API development and logging best practices.

## Skills

### `/test-claude-plugin:fastapi`
Build Python APIs with FastAPI, Pydantic v2, and SQLAlchemy 2.0 async.

Covers:
- Project structure (domain-based)
- JWT authentication
- Pydantic v2 validation patterns
- SQLAlchemy 2.0 async setup
- 7 documented known issues and their fixes
- Testing with pytest + httpx

### `/test-claude-plugin:logging-best-practices`
Logging best practices focused on wide events (canonical log lines) for powerful debugging and analytics.

## Installation

### Option 1 — Manual (once per machine)

```shell
/plugin marketplace add lucassilveirabnp/test-claude-plugin
/plugin install test-claude-plugin@test-claude-plugin
```

### Option 2 — Via project settings (shared with team)

Add to your project's `.claude/settings.json`:

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

Then install once inside Claude Code:

```shell
/plugin install test-claude-plugin@test-claude-plugin
```

### Option 3 — Auto-enable for the whole team

Add both `extraKnownMarketplaces` and `enabledPlugins` to `.claude/settings.json` and commit it to the repo. Everyone who opens the project will be prompted to install automatically:

```json
{
  "extraKnownMarketplaces": {
    "test-claude-plugin": {
      "source": {
        "source": "github",
        "repo": "lucassilveirabnp/test-claude-plugin"
      }
    }
  },
  "enabledPlugins": {
    "test-claude-plugin@test-claude-plugin": true
  }
}
```

## Test locally (development)

```bash
claude --plugin-dir ./test-claude-plugin
```

## Plugin structure

```
test-claude-plugin/
├── .claude-plugin/
│   ├── plugin.json        # Plugin manifest
│   └── marketplace.json   # Marketplace catalog
├── skills/
│   ├── fastapi/
│   │   └── SKILL.md       # /test-claude-plugin:fastapi
│   └── logging-best-practices/
│       └── SKILL.md       # /test-claude-plugin:logging-best-practices
└── README.md
```
