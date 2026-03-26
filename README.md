# claude-skills

Claude Code skills and plugins for Magik development.

## Contents

| Path | What it does |
|---|---|
| `skills/magik/` | Magik skill — teaches Claude Code Magik syntax, idioms, and best practices |
| `plugins/magik-lsp/` | magik-lsp plugin — wires the Magik Language Server into Claude Code's LSP |

---

## Prerequisites

- [Claude Code](https://github.com/anthropics/claude-code) installed
- Java 17 or later (JDK) — required for the language server

---

## 1. Install the Magik Skill

The skill gives Claude deep knowledge of Magik: syntax, naming conventions, collections, error handling, anti-patterns, and more. It is auto-triggered whenever you work on Magik or Smallworld code.

```bash
# Clone this repo (or your fork)
git clone https://github.com/robin-putters/claude-skills.git

# Copy the skill into Claude's user skills directory
cp -r claude-skills/skills/magik ~/.claude/skills/magik
```

Claude Code will pick up the new skill automatically — no restart needed.

---

## 2. Install the magik-lsp Plugin

The plugin configures the [magik-tools language server](https://github.com/StevenLooman/magik-tools) so Claude Code gets live diagnostics, hover info, and completions for `.magik` files.

### Step 1 — Download the language server JAR

Go to the [magik-tools releases page](https://github.com/StevenLooman/magik-tools/releases) and download the latest `magik-language-server-*.jar`.

Extract / place it in a permanent location, for example:

```bash
mkdir -p ~/.local/share/magik-lsp
mv magik-language-server-*.jar ~/.local/share/magik-lsp/
```

### Step 2 — Add the LSP configuration to Claude Code settings

Open (or create) `~/.claude/settings.json` and add an `lsp` section. Replace the JAR filename with the version you downloaded:

```json
{
  "lsp": {
    "magik": {
      "command": "java",
      "args": [
        "-jar",
        "/Users/YOU/.local/share/magik-lsp/magik-language-server-0.11.0.jar",
        "--debug"
      ],
      "extensionToLanguage": {
        ".magik": "magik",
        ".def": "sw-product-def"
      },
      "settings": {
        "magik": {
          "productDirs": [],
          "lint": {
            "overrideConfigFile": null
          },
          "typing": {
            "typeDatabasePaths": [],
            "showTypingInlayHints": false,
            "showArgumentInlayHints": false,
            "enableChecks": false,
            "indexGlobalUsages": true,
            "indexMethodUsages": false,
            "indexSlotUsages": true,
            "indexConditionUsages": true,
            "cacheIndexedDefinitions": true
          },
          "formatting": {
            "indentChar": "space",
            "indentWidth": 2,
            "insertFinalNewline": true,
            "trimTrailingWhitespace": true,
            "trimFinalNewlines": true,
            "indentStrategy": "null"
          }
        }
      }
    }
  }
}
```

If `settings.json` already contains other configuration, merge the `"lsp"` key into the existing JSON object rather than replacing the file.

### Step 3 — Point `productDirs` at your Smallworld installation (optional but recommended)

For type checking and symbol resolution to work, add your Smallworld product directories:

```json
"productDirs": ["/opt/smallworld/core", "/path/to/your/product"]
```

---

## Updating

```bash
cd claude-skills
git pull
cp -r skills/magik ~/.claude/skills/magik
```

The LSP plugin is just configuration — update the JAR path in `settings.json` when you upgrade magik-tools.

---

## More Information

- [magik-tools (StevenLooman)](https://github.com/StevenLooman/magik-tools)
- [Claude Code documentation](https://docs.anthropic.com/en/docs/claude-code)
