# Guia de Skills e Plugins — Claude Code

> Baseado na [documentação oficial](https://code.claude.com/docs/en/skills)

---

## O que é um plugin?

Um plugin é um repositório GitHub com skills (comandos) para o Claude Code. Skills são arquivos `.md` com instruções que o Claude segue quando invocados.

---

## Estrutura obrigatória do repositório

```
meu-plugin/
├── .claude-plugin/
│   ├── plugin.json        ← identidade do plugin (obrigatório)
│   └── marketplace.json   ← catálogo para distribuição (obrigatório para instalar via /plugin)
├── skills/
│   ├── nome-da-skill/
│   │   └── SKILL.md       ← uma pasta por skill, arquivo sempre SKILL.md
│   └── outra-skill/
│       └── SKILL.md
└── README.md
```

**Regras críticas:**
- Só o `plugin.json` e `marketplace.json` ficam dentro de `.claude-plugin/`
- Skills ficam em `skills/<nome>/SKILL.md` — nunca `.md` na raiz
- O nome da pasta vira o comando: `skills/fastapi/` → `/meu-plugin:fastapi`

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

## SKILL.md — frontmatter e conteúdo

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
