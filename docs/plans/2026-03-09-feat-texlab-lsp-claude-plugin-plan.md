---
title: "feat: Build and publish texlab LSP plugin for Claude Code"
type: feat
status: active
date: 2026-03-09
---

# feat: Build and publish texlab LSP plugin for Claude Code

## Overview

Create a standalone GitHub repository that functions as a Claude Code plugin marketplace, publishing a single `texlab-lsp` plugin that registers `texlab` as the LaTeX language server. Users add the repo as an `extraKnownMarketplaces` source in their `~/.claude/settings.json`, then install via `claude plugin install texlab-lsp@texlab-lsp`.

This mirrors exactly how the official `claude-plugins-official` marketplace works.

## Background

Claude Code's plugin system supports language servers via `lspServers` config embedded in a **marketplace manifest** (`.claude-plugin/marketplace.json`), not in individual plugin directories. Individual plugin dirs contain only `README.md` and `LICENSE`. The LSP config — command, args, extension→language mappings — lives at the marketplace level, keyed per plugin entry.

`texlab` is the de-facto LaTeX LSP (RFC-7553, maintained by the texlab team), providing:
- Real-time diagnostics
- Go-to-definition across `\input`/`\include`
- Hover for commands and environments
- Completion and formatting

## Repository Structure

```
texlab-lsp-claude/                       ← GitHub repo root
├── .claude-plugin/
│   └── marketplace.json                 ← marketplace manifest (lspServers lives here)
├── plugins/
│   └── texlab-lsp/
│       ├── README.md                    ← installation + binary setup instructions
│       └── LICENSE                      ← MIT or Apache-2.0
└── README.md                            ← top-level: what this repo is, how to add it
```

## Key Files

### `.claude-plugin/marketplace.json`

```json
{
  "$schema": "https://anthropic.com/claude-code/marketplace.schema.json",
  "name": "texlab-lsp",
  "description": "LaTeX language server (texlab) plugin for Claude Code",
  "owner": {
    "name": "<your-name>",
    "email": "<your-email>"
  },
  "plugins": [
    {
      "name": "texlab-lsp",
      "description": "LaTeX language server (texlab) for diagnostics and code intelligence in .tex, .bib, .sty, and .cls files",
      "version": "1.0.0",
      "author": {
        "name": "<your-name>"
      },
      "source": "./plugins/texlab-lsp",
      "category": "development",
      "strict": false,
      "lspServers": {
        "latex": {
          "command": "texlab",
          "extensionToLanguage": {
            ".tex": "latex",
            ".bib": "bibtex",
            ".sty": "latex",
            ".cls": "latex"
          }
        }
      }
    }
  ]
}
```

**Notes:**
- `lspServers` key `"latex"` is the language ID (matches what texlab reports)
- No `"args"` needed — texlab defaults to stdio when called without args
- `strict: false` allows the plugin to work without being Anthropic-signed

### `plugins/texlab-lsp/README.md`

Documents the plugin and binary installation:

```markdown
# texlab-lsp

LaTeX language server (texlab) for Claude Code, providing diagnostics,
go-to-definition, hover, and code intelligence for .tex, .bib, .sty, .cls files.

## Supported Extensions
`.tex`, `.bib`, `.sty`, `.cls`

## Prerequisites

Install the `texlab` binary before enabling this plugin:

### macOS
\`\`\`bash
brew install texlab
\`\`\`

### Linux (apt)
\`\`\`bash
apt install texlab
\`\`\`

### Cargo (cross-platform)
\`\`\`bash
cargo install texlab
\`\`\`

Verify installation:
\`\`\`bash
texlab --version
\`\`\`

## More Information
- [texlab GitHub](https://github.com/latex-lsp/texlab)
- [texlab documentation](https://github.com/latex-lsp/texlab/wiki)
```

### `README.md` (repo root)

User-facing installation guide:

```markdown
# texlab-lsp for Claude Code

Adds [texlab](https://github.com/latex-lsp/texlab) LaTeX language server support to Claude Code.

## Installation

### Step 1 — Install the texlab binary

\`\`\`bash
brew install texlab       # macOS
apt install texlab         # Debian/Ubuntu
cargo install texlab       # any platform via Cargo
\`\`\`

### Step 2 — Add this marketplace to Claude Code

Add to `~/.claude/settings.json`:

\`\`\`json
{
  "extraKnownMarketplaces": {
    "texlab-lsp": {
      "source": {
        "source": "github",
        "repo": "<github-username>/texlab-lsp-claude"
      }
    }
  }
}
\`\`\`

### Step 3 — Install the plugin

\`\`\`bash
claude plugin install texlab-lsp@texlab-lsp --scope user
\`\`\`

Or via the TUI: `/plugin > Discover > texlab-lsp`

## Supported file types

| Extension | Language |
|-----------|----------|
| `.tex`    | LaTeX    |
| `.bib`    | BibTeX   |
| `.sty`    | LaTeX    |
| `.cls`    | LaTeX    |
```

## Implementation Steps

### Phase 1 — Scaffold the repo

- [ ] Create GitHub repo `<username>/texlab-lsp-claude` (public)
- [ ] Create directory structure as above
- [ ] Write `.claude-plugin/marketplace.json` (see above)
- [ ] Write `plugins/texlab-lsp/README.md` with binary install instructions
- [ ] Add `LICENSE` (MIT recommended — matches official plugins)
- [ ] Write repo-level `README.md`

### Phase 2 — Validate locally before publishing

- [ ] Clone the repo locally, then validate:
  ```bash
  claude plugin validate /path/to/texlab-lsp-claude
  ```
  Expected: `✔ Validation passed` (with optional description warning)

- [ ] Add the marketplace locally to test end-to-end:
  ```json
  // ~/.claude/settings.json
  "extraKnownMarketplaces": {
    "texlab-lsp-dev": {
      "source": {
        "source": "directory",
        "path": "/path/to/texlab-lsp-claude"
      }
    }
  }
  ```
  Then: `claude plugin install texlab-lsp@texlab-lsp-dev --scope user`

- [ ] Open a `.tex` file in a Claude Code session and verify the LSP starts (check `/plugin` Errors tab for texlab errors)

- [ ] Confirm texlab is providing diagnostics (introduce a LaTeX error, confirm Claude sees it)

### Phase 3 — Publish

- [ ] Push to GitHub
- [ ] Update `extraKnownMarketplaces` to use `source: "github"` pointing to the published repo
- [ ] Re-test the full install flow from the GitHub source
- [ ] Tag a `v1.0.0` release

### Phase 4 — Optional: submit to official marketplace

The official `claude-plugins-official` marketplace accepts external plugin submissions via a Google Form linked in their README. Submitting `texlab-lsp` there would give users access via the standard `/plugin > Discover` flow without needing to add a custom marketplace.

- [ ] Review submission requirements at `claude-plugins-official` README
- [ ] Submit via the plugin directory submission form

## Acceptance Criteria

- [ ] `claude plugin validate` passes with no errors
- [ ] `claude plugin install texlab-lsp@texlab-lsp` installs successfully from the GitHub source
- [ ] A `.tex` file opened in Claude Code triggers texlab (visible in `/plugin` status)
- [ ] Claude receives and acts on LSP diagnostics after editing a `.tex` file
- [ ] BibTeX (`.bib`) files are also handled
- [ ] Plugin README documents all three binary install paths (brew, apt, cargo)

## Technical Notes

- **No `args` needed for texlab** — unlike some LSPs (`typescript-language-server --stdio`), texlab defaults to stdio transport when invoked without arguments. Omitting `args` is correct.
- **`strict: false`** — required for any plugin not signed by Anthropic. Users will see a trust prompt on first install, which is expected.
- **Language ID `"latex"`** — this is what texlab declares in its server capabilities. Using a different key would cause a mismatch. `"bibtex"` is not needed as a separate lspServer entry; texlab handles `.bib` via the same `latex` language server when mapped with `extensionToLanguage`.
- **Version bumps** — increment `"version"` in `marketplace.json` on each update; Claude Code uses this to detect when to prompt for plugin updates.

## References

- [Official claude-plugins-official marketplace](https://github.com/anthropics/claude-plugins-official) — reference implementation to mirror
- [texlab GitHub](https://github.com/latex-lsp/texlab) — binary source and LSP spec compliance notes
- [Claude Code plugin docs](https://code.claude.com/docs/en/plugins.md)
- [Claude Code plugins reference (lspServers schema)](https://code.claude.com/docs/en/plugins-reference.md)
