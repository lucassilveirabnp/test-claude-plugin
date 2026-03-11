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

### Test locally
```bash
claude --plugin-dir ./test-claude-plugin
```

### Via marketplace (after setup)
See [Claude Code plugin marketplaces](https://code.claude.com/docs/en/plugin-marketplaces) for distribution options.

## Plugin structure

```
test-claude-plugin/
├── .claude-plugin/
│   └── plugin.json          # Plugin manifest
├── skills/
│   ├── fastapi/
│   │   └── SKILL.md         # FastAPI skill
│   └── logging-best-practices/
│       └── SKILL.md         # Logging skill
└── README.md
```
