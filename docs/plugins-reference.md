# Plugins Reference — Cache e Resolução de Arquivos

> Fonte: https://code.claude.com/docs/en/plugins-reference

## Como o cache funciona

Plugins instalados via marketplace são **copiados** para o cache local do usuário (`~/.claude/plugins/cache`), nunca usados in-place.

```
~/.claude/plugins/cache/
└── <marketplace-name>/
    └── <plugin-name>/
        └── <version>/
            └── [conteúdo do plugin]
```

**Duas formas de especificar plugins:**
- `claude --plugin-dir ./caminho` — sessão temporária, não vai para o cache
- Via marketplace (instalado) — copiado para o cache, persiste entre sessões

---

## Limitações de path traversal

Plugins instalados **não podem referenciar arquivos fora do seu diretório**. Paths como `../shared-utils` não funcionam após instalação porque arquivos externos não são copiados para o cache.

### Solução: symlinks

Symlinks dentro do diretório do plugin **são honrados** durante a cópia para o cache:

```bash
# Dentro do diretório do plugin
ln -s /caminho/para/shared-utils ./shared-utils
```

O conteúdo linkado será copiado para o cache junto com o plugin.

---

## Plugin manifest schema (plugin.json)

| Campo | Tipo | Obrigatório | Descrição |
|-------|------|-------------|-----------|
| `name` | string | Sim | Identificador único (kebab-case) |
| `version` | string | Recomendado | Semver. Usado para detectar updates e invalidar cache |
| `description` | string | Não | Descrição breve |
| `author` | object | Não | `name` obrigatório, `email` opcional |
| `homepage` | string | Não | URL de documentação |
| `repository` | string | Não | URL do repositório |
| `license` | string | Não | Identificador SPDX |
| `keywords` | array | Não | Tags para descoberta |

---

## Strict mode

O campo `strict` no `marketplace.json` (por plugin entry) controla se `plugin.json` é a autoridade para definições de componentes.

- `strict: true` (padrão): `plugin.json` define quais componentes estão ativos
- `strict: false`: todos os componentes encontrados nos diretórios padrão são carregados

---

## Diagrama: fluxo de instalação via marketplace

```
extraKnownMarketplaces → [GitHub repo ou URL]
         ↓
  Claude Code clona/busca o marketplace
         ↓
  Lê marketplace.json
         ↓
  Para cada plugin com source:
    - "./caminho" → copia do repo já clonado (sem segundo clone)
    - "github"    → clona repo separado do plugin
    - "npm"       → instala via npm
         ↓
  Copia para ~/.claude/plugins/cache/<marketplace>/<plugin>/<version>/
         ↓
  Plugin disponível nas próximas sessões
```

⚠️ **Armadilha de recursão**: Se `marketplace.json` está dentro do plugin repo e usa `source: github` apontando para o mesmo repo:
1. Claude Code clona o repo para ler o marketplace
2. Encontra `source: github` → clona o repo NOVAMENTE dentro do cache
3. O segundo clone também tem `marketplace.json` → clona de novo
4. Loop infinito → MAX_PATH no Windows → EFAULT

**Solução**: Usar `source: "./"` no marketplace.json quando o plugin e o marketplace estão no mesmo repo.

---

## Anatomia do cache com source: "./"

```
~/.claude/plugins/cache/
└── test-claude-plugin/          ← nome do marketplace
    └── test-claude-plugin/      ← nome do plugin
        └── 1.1.0/               ← versão
            ├── .claude-plugin/
            ├── skills/
            └── ...              ← cópia do repo (sem recursão)
```

Com `source: github` no mesmo repo, aparece:
```
            └── 1.1.0/
                └── test-claude-plugin/  ← segundo clone (recursão!)
                    └── 1.1.0/
                        └── test-claude-plugin/  ← infinito
```

---

## extraKnownMarketplaces

Configurado em `.claude/settings.json` do projeto ou global. Adiciona marketplaces conhecidos sem precisar rodar `/plugin marketplace add`.

Para que **paths relativos** (`source: "./"`) funcionem, o marketplace deve ser adicionado **via GitHub** (não via URL direta ao JSON):

```json
{
  "extraKnownMarketplaces": [
    {
      "source": "github",
      "repo": "owner/plugin-repo"
    }
  ]
}
```

Se usar URL direta (ex: Gist raw URL), paths relativos no marketplace.json NÃO funcionam.
