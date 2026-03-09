# texlab-lsp for Claude Code

Adds [texlab](https://github.com/latex-lsp/texlab) LaTeX language server support to Claude Code.

## Installation

### Step 1 — Install the texlab binary

```bash
brew install texlab       # macOS
apt install texlab         # Debian/Ubuntu
cargo install texlab       # any platform via Cargo
```

### Step 2 — Add the marketplace and install the plugin

Inside Claude Code, run:

```
/plugin marketplace add jottr/texlab-lsp-claude
/plugin install texlab-lsp@texlab-lsp
```

## Supported file types

| Extension | Language |
|-----------|----------|
| `.tex`    | LaTeX    |
| `.bib`    | BibTeX   |
| `.sty`    | LaTeX    |
| `.cls`    | LaTeX    |
