---
title: "Skill Authoring"
summary: "How to author SKILL.md files with machine-readable YAML frontmatter."
read_when:
  - Writing or reviewing a SKILL.md file
  - Implementing load-time gating or keyword scoring
  - Designing the requires block or description field
status: active
last_updated: "2025-01-16"
---

# How to Write Skills — Machine-Readable YAML Frontmatter

The SKILL.md body is for the agent (natural language instructions).
The YAML frontmatter is for the **runtime** — discovery, gating, scoring, injection.
Every frontmatter field must be parseable and actionable by code.

---

## 1. Directory structure

```
my-skill/                  ← directory name must match `name` field exactly
├── SKILL.md               ← required
├── references/            ← optional: docs loaded on demand
└── assets/                ← optional: templates, WASM modules
    └── my-tools.wasm      ← optional: bundled MCP server
```

**No custom install scripts**: Skills are declarative only. System dependencies declared in `requires` block are checked at load time but not installed automatically.

**Bundled MCP servers**: Skills can include WASM-based MCP servers in `assets/*.wasm` for tool execution. See [MCP WASM specification](../tools/mcp-wasm.md).

---

## 2. Full frontmatter schema

```yaml
---
# ── Identity ─────────────────────────────────────────────────────────────────
name: my-skill
description: >
  What this skill does and when to use it. Must contain the vocabulary
  the agent will use when searching. Use when the user needs to [task].

# ── Discovery / scoring ───────────────────────────────────────────────────────
tags:
  - category
  - domain
triggers:
  - keyword1
  - keyword2

# ── Runtime gating ────────────────────────────────────────────────────────────
requires:
  bins:
    - ffmpeg
  env:
    - OPENAI_API_KEY
  mcp_servers:
    - my-tools        # bundled MCP server in assets/my-tools.wasm

# ── Lifecycle management ────────────────────────────────────────────────────
lifecycle:
  variables:
    SKILL_PATH: ${HOME}/.local/share/skills/${SKILL_NAME}
    VENV_PATH: ${SKILL_PATH}/.venv
    PYTHON_BIN: ${VENV_PATH}/bin/python
  install:
    - command: uv venv ${VENV_PATH} && uv pip install --python ${PYTHON_BIN} my-package
      description: Create venv and install my-package with uv
      platform: all
      requires_approval: true
  update:
    - command: uv pip install --python ${PYTHON_BIN} --upgrade my-package
      description: Update my-package to latest version
      platform: all
      requires_approval: true
  uninstall:
    - command: rm -rf ${VENV_PATH}
      description: Remove virtual environment
      platform: all
      requires_approval: true
    - command: rm -rf ~/.cache/my-package
      description: Remove cached data
      platform: all
      requires_approval: true

# ── Tool permissions ──────────────────────────────────────────────────────────
allowed-tools: Bash(ffmpeg:*) Read Write MCP(my-tools:*)

# ── Publishing metadata ───────────────────────────────────────────────────────
license: MIT
metadata:
  author: my-org
  version: "1.0"
  homepage: https://github.com/my-org/my-skill
---
```

### Field reference

| Field | Required | Machine-readable | Purpose |
|---|---|---|---|
| `name` | Yes | ✅ | Identity, must match directory name |
| `description` | Yes | ✅ (keyword scoring) | Discovery + activation signal |
| `tags` | No | ✅ list | Keyword scorer (+1/term) |
| `triggers` | No | ✅ list | Vocabulary checklist (see §4) |
| `requires` | No | ✅ nested map | Load-time gating (bins, env, mcp_servers) |
| `lifecycle` | No | ✅ nested map | Agent-assisted install/update/uninstall commands |
| `allowed-tools` | No | ✅ | Tool allowlist |
| `license` | No | ✅ | License identifier |
| `metadata` | No | ✅ key-value map | Author, version, homepage |

---

## 3. The `requires` block — load-time gating

Skills that fail the gate are never injected into the system prompt.
The agent never sees a skill it cannot use.

**This is declarative checking, not installation**: The system checks if dependencies exist but does not install them. Users must install dependencies manually or via their system's package manager.

```yaml
requires:
  bins:
    - ffmpeg          # must exist on PATH
    - jq
  env:
    - OPENAI_API_KEY  # must be set in environment
  mcp_servers:
    - python-tools    # bundled MCP server (assets/python-tools.wasm)
  config:
    - browser.enabled # must be truthy in agent config
```

**MCP server requirements**:
- `mcp_servers`: List of bundled MCP server names (matches `assets/<name>.wasm`)
- Skill gates if bundled WASM file is missing

See [MCP WASM specification](../tools/mcp-wasm.md) for server packaging.

---

## 4. The `lifecycle` block — agent-assisted management

Skills declare commands for installation, updates, and uninstallation in a single `lifecycle` block. The agent executes these with user approval.

```yaml
lifecycle:
  variables:
    SKILL_PATH: ${HOME}/.local/share/skills/${SKILL_NAME}
    VENV_PATH: ${SKILL_PATH}/.venv
    PYTHON_BIN: ${VENV_PATH}/bin/python
  install:
    - command: uv venv ${VENV_PATH} && uv pip install --python ${PYTHON_BIN} marker-pdf[full]
      description: Create venv and install marker-pdf with all dependencies
      platform: all
      requires_approval: true
    - command: brew install ffmpeg
      description: Install ffmpeg via Homebrew
      platform: macos
      requires_approval: true
  update:
    - command: uv pip install --python ${PYTHON_BIN} --upgrade marker-pdf
      description: Update marker-pdf to latest version
      platform: all
      requires_approval: true
  uninstall:
    - command: rm -rf ${VENV_PATH}
      description: Remove virtual environment
      platform: all
      requires_approval: true
    - command: rm -rf ~/Library/Caches/datalab
      description: Remove cached models (macOS)
      platform: macos
      requires_approval: true
```

**Variables**:
- `lifecycle.variables`: Define custom variables with default values
- Variables can reference other variables and built-in variables
- Evaluated in order of declaration

**Built-in variables**:
- `${SKILL_NAME}`: Skill name from frontmatter (e.g., `datalab-marker`)
- `${HOME}`: User home directory
- `${PLATFORM}`: Current platform (`linux`, `macos`, `windows`)

**Custom variables**: Defined in `lifecycle.variables` block with defaults

### Variable Evaluation

Variables are evaluated in this order:

1. **Built-in variables** are initialized first:
   ```python
   SKILL_NAME = "datalab-marker"  # from frontmatter
   HOME = "/Users/username"        # from system
   PLATFORM = "macos"              # from system
   ```

2. **Custom variables** are evaluated in declaration order:
   ```yaml
   variables:
     SKILL_PATH: ${HOME}/.local/share/skills/${SKILL_NAME}
     VENV_PATH: ${SKILL_PATH}/.venv
     PYTHON_BIN: ${VENV_PATH}/bin/python
   ```
   
   Evaluation:
   ```python
   # Step 1: SKILL_PATH
   SKILL_PATH = "${HOME}/.local/share/skills/${SKILL_NAME}"
   SKILL_PATH = "/Users/username/.local/share/skills/datalab-marker"
   
   # Step 2: VENV_PATH (can use SKILL_PATH)
   VENV_PATH = "${SKILL_PATH}/.venv"
   VENV_PATH = "/Users/username/.local/share/skills/datalab-marker/.venv"
   
   # Step 3: PYTHON_BIN (can use VENV_PATH)
   PYTHON_BIN = "${VENV_PATH}/bin/python"
   PYTHON_BIN = "/Users/username/.local/share/skills/datalab-marker/.venv/bin/python"
   ```

3. **Commands** are evaluated with all variables:
   ```yaml
   command: uv venv ${VENV_PATH} && uv pip install --python ${PYTHON_BIN} marker-pdf
   ```
   
   Result:
   ```bash
   uv venv /Users/username/.local/share/skills/datalab-marker/.venv && \
   uv pip install --python /Users/username/.local/share/skills/datalab-marker/.venv/bin/python marker-pdf
   ```

**Rules**:
- Variables are case-sensitive
- Forward references not allowed (variable must be defined before use)
- Circular references cause error
- Unknown variables left as-is (e.g., `${UNKNOWN}` stays `${UNKNOWN}`)

**Structure**:
- `lifecycle.install`: Commands to install missing dependencies
- `lifecycle.update`: Commands to update to newer versions
- `lifecycle.uninstall`: Commands to clean up dependencies and caches

**Command fields**:
- `command`: Shell command to execute
- `description`: Human-readable explanation
- `platform`: `all`, `linux`, `macos`, `windows`
- `requires_approval`: Always `true` (agent must ask user)

**Security**:
- All commands visible in SKILL.md (auditable)
- User must approve each command individually
- Agent explains what each command does
- No hidden or obfuscated commands

---

## 5. The `description` field

The description serves two roles:

1. **The model** — decides whether to activate the skill based on this text
2. **The keyword scorer** — tokenizes and scores against `name`, `description`, `tags`

```yaml
# Bad — vague, no activation signal, no searchable vocabulary
description: Helps with PDFs.

# Good — specific capability + explicit trigger condition + searchable terms
description: >
  Extract text and tables from PDF files, fill PDF forms, and merge multiple PDFs.
  Use when the user needs to work with PDF documents, extract content, fill forms,
  or combine files.
```

YAML block scalar (`>`) folds newlines into spaces — use it for long descriptions.

---

## 6. `tags` and `triggers`

Scorer weights:

```
name match:        +3 per term
description match: +2 per term
tag match:         +1 per term
```

`triggers` are a **vocabulary checklist** — every trigger term must appear in `description`
or `name`. If it doesn't, the description is missing that vocabulary.

```yaml
description: >
  Process and resize images, convert between formats (JPEG, PNG, WebP, AVIF),
  apply filters. Use when the user needs to edit, convert, or optimize images.
tags:
  - image
  - media
triggers:
  - image
  - photo
  - resize
  - convert
  - jpeg
  - png
  - webp
  - filter
  - optimize
```

---

## 7. Progressive disclosure

```
Tier 1 — Discovery  (~100 tokens):  name + description, loaded at startup for ALL skills
Tier 2 — Activation (<5000 tokens): full SKILL.md body, loaded when skill is selected
Tier 3 — Resources  (as needed):    references/, assets/ loaded on demand
```

Body structure:

```markdown
## When to use this skill
Repeat the activation conditions from the description. Be explicit.

## How to [primary task]
Step-by-step. Concrete. No filler.

## Edge cases
What can go wrong and how to handle it.

For the full reference, see [references/api.md](references/api.md).
```

Keep body under 500 lines. Move large reference tables and full API docs to `references/`.

---

## 8. Complete example

```markdown
---
name: pdf-processing
description: >
  Extract text and tables from PDF files, fill PDF forms, and merge multiple PDFs.
  Use when the user needs to work with PDF documents, extract content, fill forms,
  or combine files.
tags:
  - pdf
  - document
triggers:
  - pdf
  - extract
  - table
  - form
  - merge
requires:
  bins:
    - pdfplumber
    - pypdf
lifecycle:
  variables:
    SKILL_PATH: ${HOME}/.local/share/skills/${SKILL_NAME}
    VENV_PATH: ${SKILL_PATH}/.venv
    PYTHON_BIN: ${VENV_PATH}/bin/python
  install:
    - command: uv venv ${VENV_PATH} && uv pip install --python ${PYTHON_BIN} pdfplumber pypdf
      description: Create venv and install PDF processing libraries
      platform: all
      requires_approval: true
  update:
    - command: uv pip install --python ${PYTHON_BIN} --upgrade pdfplumber pypdf
      description: Update PDF libraries to latest versions
      platform: all
      requires_approval: true
  uninstall:
    - command: rm -rf ${VENV_PATH}
      description: Remove virtual environment
      platform: all
      requires_approval: true
allowed-tools: Bash(python:*) Read Write
license: MIT
metadata:
  author: my-org
  version: "1.0"
---

# PDF Processing

## When to use this skill

Use when the user mentions PDFs, needs to extract text or tables, fill a form,
or combine multiple PDF files.

## Extract text

1. Use pdfplumber to open the PDF file
2. Iterate through pages and extract text with `page.extract_text()`
3. For tables, use `page.extract_tables()`

Example: `python -c "import pdfplumber; print(pdfplumber.open('file.pdf').pages[0].extract_text())"`

## Merge PDFs

1. Use pypdf's PdfWriter class
2. Append each input PDF with `writer.append(path)`
3. Write combined output with `writer.write(output_path)`

Example: `python -c "from pypdf import PdfWriter; w=PdfWriter(); [w.append(f) for f in ['a.pdf','b.pdf']]; w.write('out.pdf')"`

## Edge cases

- Scanned PDFs: pdfplumber returns empty text — use OCR instead.
- Encrypted PDFs: pass `password=` to `pdfplumber.open()`.

For the full API reference, see [references/api.md](references/api.md).
```

---

## 9. Authoring checklist

- [ ] Directory name matches `name` exactly (lowercase, hyphens only)
- [ ] `description` answers *what* + *when* in ≤1024 chars
- [ ] Every `triggers` term appears in `description` or `name`
- [ ] `requires` lists all runtime dependencies (bins, env, mcp_servers)
- [ ] `lifecycle` block provided with install/update/uninstall commands (if applicable)
- [ ] `lifecycle.variables` defined for reusable paths
- [ ] Body is under 500 lines; large reference material in `references/`
- [ ] If bundling MCP servers, include `assets/*.wasm` and declare in `requires.mcp_servers`
