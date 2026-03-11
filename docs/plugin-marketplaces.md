# Plugin Marketplaces — Referência Oficial

> Fonte: https://code.claude.com/docs/en/plugin-marketplaces

## O que é um marketplace

Um marketplace é um catálogo (`marketplace.json`) que lista plugins e onde encontrá-los. Permite que usuários instalem plugins via `/plugin install nome@marketplace`.

**Marketplace source vs Plugin source** — conceitos distintos:
- **Marketplace source**: onde buscar o `marketplace.json` em si (configurado em `extraKnownMarketplaces` ou via `/plugin marketplace add`)
- **Plugin source**: onde buscar cada plugin listado dentro do `marketplace.json` (campo `source` de cada entrada de plugin)

---

## Estrutura do marketplace.json

```json
{
  "name": "company-tools",
  "owner": {
    "name": "DevTools Team",
    "email": "devtools@example.com"
  },
  "metadata": {
    "description": "Descrição do marketplace",
    "pluginRoot": "./plugins"
  },
  "plugins": [
    {
      "name": "meu-plugin",
      "source": "./plugins/meu-plugin",
      "description": "Plugin local no mesmo repo",
      "version": "1.0.0"
    },
    {
      "name": "outro-plugin",
      "source": {
        "source": "github",
        "repo": "owner/outro-plugin-repo"
      },
      "description": "Plugin em repo separado"
    }
  ]
}
```

---

## Campos obrigatórios do marketplace

| Campo     | Tipo   | Descrição |
|-----------|--------|-----------|
| `name`    | string | Identificador (kebab-case). Público: aparece no `/plugin install` |
| `owner`   | object | `name` obrigatório, `email` opcional |
| `plugins` | array  | Lista de plugins disponíveis |

**Nomes reservados** (não podem ser usados): `claude-code-marketplace`, `claude-plugins-official`, `anthropic-marketplace`, `anthropic-plugins`, `agent-skills`, `life-sciences`.

---

## Campos opcionais (metadata)

| Campo                  | Descrição |
|------------------------|-----------|
| `metadata.description` | Descrição breve do marketplace |
| `metadata.version`     | Versão do marketplace |
| `metadata.pluginRoot`  | Diretório base para paths relativos (ex: `"./plugins"` permite escrever `"source": "formatter"` em vez de `"source": "./plugins/formatter"`) |

---

## Plugin sources (onde buscar cada plugin)

### ✅ Path relativo (recomendado para repo único)

```json
{ "name": "meu-plugin", "source": "./" }
```

- Aponta para um diretório local dentro do **repo do marketplace**
- **Deve começar com `./`**
- **SÓ funciona quando o marketplace é adicionado via Git (GitHub/GitLab/URL git)**
- Se adicionado via URL direta ao JSON, paths relativos NÃO funcionam

### GitHub

```json
{
  "name": "plugin",
  "source": {
    "source": "github",
    "repo": "owner/plugin-repo",
    "ref": "v2.0.0",
    "sha": "a1b2c3d4..."
  }
}
```

⚠️ **CUIDADO**: Se o marketplace.json está DENTRO do plugin repo e aponta `source: github` para o mesmo repo, causa **recursão infinita no cache** — o repo é clonado de novo dentro do clone anterior, infinitamente.

### URL git

```json
{
  "name": "plugin",
  "source": {
    "source": "url",
    "url": "https://gitlab.com/team/plugin.git",
    "ref": "main"
  }
}
```

### Git subdirectory (monorepo)

```json
{
  "name": "plugin",
  "source": {
    "source": "git-subdir",
    "url": "https://github.com/acme/monorepo.git",
    "path": "tools/claude-plugin",
    "ref": "v2.0.0"
  }
}
```

Usa sparse clone — baixa apenas o subdiretório.

### npm

```json
{
  "name": "plugin",
  "source": {
    "source": "npm",
    "package": "@acme/claude-plugin",
    "version": "2.1.0",
    "registry": "https://npm.example.com"
  }
}
```

### pip

```json
{
  "name": "plugin",
  "source": {
    "source": "pip",
    "package": "acme-claude-plugin",
    "version": "1.0.0"
  }
}
```

---

## Campos opcionais por plugin entry

| Campo        | Tipo           | Descrição |
|--------------|----------------|-----------|
| `description`| string         | Descrição breve |
| `version`    | string         | Versão do plugin |
| `author`     | object         | `name` obrigatório, `email` opcional |
| `homepage`   | string         | URL de documentação |
| `license`    | string         | Identificador SPDX (ex: MIT) |
| `keywords`   | array          | Tags para descoberta |
| `category`   | string         | Categoria |
| `strict`     | boolean        | Se `plugin.json` é autoridade para componentes (padrão: true) |
| `commands`   | string\|array  | Paths customizados para arquivos de commands |
| `agents`     | string\|array  | Paths para arquivos de agents |
| `hooks`      | string\|object | Config de hooks ou path para arquivo |
| `mcpServers` | string\|object | Config de MCP servers |

---

## Troubleshooting

**Plugins com paths relativos falham em marketplaces via URL**

Se `extraKnownMarketplaces` aponta para uma URL direta ao `marketplace.json` (ex: URL raw de Gist), paths relativos como `"source": "./"` não funcionam. Use sources do tipo `github`, `url`, `npm` ou `pip` em vez disso.

**Recursão infinita no cache**

Causa: `marketplace.json` dentro do plugin repo usa `source: github` apontando para o mesmo repo. Solução: usar `source: "./"` (path relativo) em vez de `source: github` para o próprio repo.
