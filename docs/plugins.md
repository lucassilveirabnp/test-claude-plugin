# Plugins — Referência Oficial

> Fonte: https://code.claude.com/docs/en/plugins

## Standalone vs Plugin

| Abordagem | Nome das skills | Quando usar |
|-----------|----------------|-------------|
| **Standalone** (`.claude/`) | `/hello` | Fluxos pessoais, configurações específicas do projeto |
| **Plugin** (`.claude-plugin/plugin.json`) | `/plugin-name:hello` | Compartilhar com o time, distribuição, versionamento, reutilização entre projetos |

---

## Estrutura de diretórios

```
meu-plugin/
├── .claude-plugin/
│   └── plugin.json          # Manifesto obrigatório
├── skills/
│   └── minha-skill/
│       └── SKILL.md         # Agent Skills
├── commands/                # Slash commands (arquivos .md)
├── agents/                  # Definições de agents customizados
├── hooks/
│   └── hooks.json           # Event handlers
├── .mcp.json                # Configurações de MCP servers
├── .lsp.json                # LSP servers
└── settings.json            # Settings padrão do plugin
```

---

## plugin.json (manifesto)

```json
{
  "name": "meu-plugin",
  "description": "Descrição do plugin",
  "version": "1.0.0",
  "author": {
    "name": "Seu Nome",
    "email": "email@example.com"
  },
  "homepage": "https://github.com/user/meu-plugin",
  "repository": "https://github.com/user/meu-plugin",
  "license": "MIT",
  "keywords": ["fastapi", "python"]
}
```

---

## Skills (SKILL.md)

Skills ficam em `skills/<nome>/SKILL.md`. São invocadas pelo modelo (model-invoked), não pelo usuário diretamente.

```markdown
---
name: minha-skill
description: Descrição usada pelo modelo para saber quando invocar esta skill
---

# Conteúdo da skill

Instruções detalhadas para o modelo...
```

**Frontmatter fields:**

| Campo | Tipo | Descrição |
|-------|------|-----------|
| `name` | string | Identificador da skill |
| `description` | string | Usada pelo modelo para decidir quando invocar. Aparecer no `/skills` |
| `disable-model-invocation` | boolean | Se true, só pode ser invocada pelo usuário explicitamente |

---

## Hooks (hooks.json)

```json
{
  "hooks": [
    {
      "event": "PreToolUse",
      "matcher": "Bash",
      "command": "echo 'about to run bash'"
    }
  ]
}
```

---

## Comandos slash (commands/)

Arquivos `.md` na pasta `commands/` viram slash commands:

```markdown
---
description: Descrição do comando
---

Instruções do comando...
```

Invocado como `/plugin-name:comando`.

---

## Desenvolvimento local

```bash
# Testar localmente (sem instalar)
claude --plugin-dir ./meu-plugin

# Recarregar mudanças
/reload-plugins

# Múltiplos plugins
claude --plugin-dir ./plugin-um --plugin-dir ./plugin-dois

# Validar estrutura
claude plugin validate .
```

---

## Versionamento

O Claude Code usa a versão para detectar updates e invalidar cache. **Sempre bump a versão ao adicionar ou modificar skills/commands/hooks.**

Usa semantic versioning: `MAJOR.MINOR.PATCH`

---

## Debug

```bash
claude --debug
```

Checklist:
1. Componentes devem estar na **raiz do plugin**, não dentro de `.claude-plugin/`
2. `plugin.json` fica em `.claude-plugin/plugin.json`
3. Skills ficam em `skills/<nome>/SKILL.md`
4. Versão foi atualizada após mudanças?

---

## Distribuição

1. Adicionar `README.md` com documentação
2. Versionar com semantic versioning
3. Criar ou usar um marketplace
4. Testar com outros antes do release
5. Submeter ao marketplace oficial: `platform.claude.com/plugins/submit`
