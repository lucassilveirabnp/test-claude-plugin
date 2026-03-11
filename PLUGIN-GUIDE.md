# Guia de Skills e Plugins — Claude Code

> Baseado na [documentação oficial](https://code.claude.com/docs/en/plugins)

---

## O que é um plugin?

Um plugin é um repositório GitHub que estende o Claude Code com um ou mais dos seguintes componentes:

| Componente | O que faz | Localização |
|---|---|---|
| **Skills** | Comandos `/` com instruções para o Claude — o mais comum | `skills/<nome>/SKILL.md` |
| **Agents** | Subagentes especializados com system prompt próprio | `agents/<nome>.md` |
| **Hooks** | Automações disparadas por eventos (ex: após editar arquivo) | `hooks/hooks.json` |
| **MCPs** | Servidores que conectam o Claude a APIs e ferramentas externas | `.mcp.json` |

Um plugin pode ter qualquer combinação desses componentes no mesmo repositório.

---

## Estrutura do repositório

```
meu-plugin/
├── .claude-plugin/
│   ├── plugin.json        ← identidade do plugin (obrigatório)
│   └── marketplace.json   ← catálogo para distribuição (obrigatório para instalar via /plugin)
├── skills/                ← skills (opcional)
│   └── nome-da-skill/
│       ├── SKILL.md       ← arquivo principal, sempre este nome
│       ├── reference/     ← arquivos de suporte opcionais
│       └── scripts/       ← scripts opcionais
├── agents/                ← subagentes (opcional)
│   └── meu-agente.md
├── hooks/                 ← automações por evento (opcional)
│   └── hooks.json
├── .mcp.json              ← servidores MCP (opcional)
└── README.md
```

**Regras críticas:**
- Só `plugin.json` e `marketplace.json` ficam dentro de `.claude-plugin/` — agentes, skills e hooks ficam na raiz
- Skills ficam em `skills/<nome>/SKILL.md` — nunca `.md` na raiz, arquivo sempre se chama `SKILL.md`
- O nome da pasta vira o comando: `skills/fastapi/` → `/meu-plugin:fastapi`
- **Nunca use `"source": "./"` no marketplace.json** quando plugin e marketplace estão na mesma raiz — causa recursão infinita no cache

---

## plugin.json

```json
{
  "name": "meu-plugin",
  "description": "Descrição do que o plugin faz",
  "version": "1.0.0",
  "author": { "name": "seu-usuario" },
  "homepage": "https://github.com/seu-usuario/meu-plugin",
  "repository": "https://github.com/seu-usuario/meu-plugin",
  "license": "MIT",
  "keywords": ["palavra-chave"]
}
```

---

## marketplace.json

```json
{
  "name": "meu-plugin",
  "owner": { "name": "seu-usuario" },
  "plugins": [
    {
      "name": "meu-plugin",
      "source": {
        "source": "github",
        "repo": "seu-usuario/meu-plugin"
      },
      "description": "Descrição",
      "version": "1.0.0",
      "license": "MIT"
    }
  ]
}
```

> ⚠️ **Nunca use `"source": "./"` quando marketplace e plugin estão na mesma raiz** — causa recursão infinita no cache.

---

## Skills — SKILL.md

```markdown
---
name: nome-da-skill
description: O que faz e quando usar. O Claude usa isso para decidir se invoca automaticamente.
user-invocable: true
---

# Conteúdo da skill

Instruções que o Claude seguirá ao executar a skill.
Use $ARGUMENTS para capturar o que o usuário passa após o comando.
```

### Campos do frontmatter

| Campo | Descrição | Padrão |
|---|---|---|
| `name` | Nome do comando (usa o nome da pasta se omitido) | nome da pasta |
| `description` | O que faz — Claude usa para invocar automaticamente | obrigatório |
| `user-invocable` | Aparece no menu `/`? | `true` |
| `disable-model-invocation` | Impede o Claude de invocar automaticamente | `false` |
| `allowed-tools` | Ferramentas permitidas sem confirmação | — |
| `argument-hint` | Dica no autocomplete, ex: `[issue-number]` | — |
| `context: fork` | Executa em subagente isolado (sem histórico da conversa) | — |

### Arquivos de suporte

Skills podem ter arquivos extras na mesma pasta. Referencie-os no SKILL.md para o Claude saber quando carregar:

```
skills/minha-skill/
├── SKILL.md          ← visão geral e navegação (máx. 500 linhas)
├── reference/        ← docs detalhadas carregadas quando necessário
└── scripts/          ← scripts que o Claude pode executar
```

---

## Agents

Subagentes com comportamento especializado. O Claude pode delegá-los via `context: fork` em uma skill.

```markdown
<!-- agents/meu-agente.md -->
---
name: meu-agente
description: Quando usar este agente
---

System prompt e instruções do agente aqui.
```

---

## Hooks

Automações disparadas por eventos do Claude Code (ex: após editar arquivo, antes de executar comando).

```json
// hooks/hooks.json
{
  "hooks": {
    "PostToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [{ "type": "command", "command": "npm run lint:fix" }]
      }
    ]
  }
}
```

Eventos disponíveis: `PreToolUse`, `PostToolUse`, `Stop`, `SubagentStop`, `PreCompact`.

---

## MCPs (Model Context Protocol)

Conecta o Claude a APIs e ferramentas externas.

```json
// .mcp.json
{
  "mcpServers": {
    "meu-servidor": {
      "command": "node",
      "args": ["${CLAUDE_PLUGIN_ROOT}/servidor.js"]
    }
  }
}
```

> Use `${CLAUDE_PLUGIN_ROOT}` para referenciar arquivos do plugin — o caminho muda após instalação.

---

## Instalação

### Uma vez por máquina
```
/plugin marketplace add seu-usuario/meu-plugin
/plugin install meu-plugin@meu-plugin
```

### Via projeto (compartilhado com o time)

Adicione ao `.claude/settings.json` do repositório e commite:

```json
{
  "extraKnownMarketplaces": {
    "meu-plugin": {
      "source": {
        "source": "github",
        "repo": "seu-usuario/meu-plugin"
      }
    }
  }
}
```

Cada pessoa instala uma vez:
```
/plugin install meu-plugin@meu-plugin
```

---

## Uso

Skills de plugin são sempre prefixadas com o nome do plugin:

```
/meu-plugin:nome-da-skill
/meu-plugin:nome-da-skill argumento opcional
```

---

## Atualizar o plugin

Após subir mudanças no GitHub:
```
/plugin marketplace update meu-plugin
/reload-plugins
```

---

## Testar localmente (sem publicar)

```bash
claude --plugin-dir ./meu-plugin
```

---

## Boas práticas

- **Uma skill por responsabilidade** — skills focadas são mais fáceis de manter e invocar
- **Description clara** — o Claude usa para decidir quando invocar automaticamente; seja específico
- **`disable-model-invocation: true`** em skills com efeitos colaterais (deploy, commit, envio de mensagens)
- **Versione sempre** — atualize `version` no `plugin.json` a cada mudança; sem isso o Claude Code não detecta atualização
- **SKILL.md abaixo de 500 linhas** — conteúdo longo vai em arquivos de suporte referenciados no SKILL.md
