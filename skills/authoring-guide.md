---
title: "Skill Authoring"
summary: "How to author SKILL.md files with machine-readable YAML frontmatter for Agent Zero."
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
├── scripts/               ← optional: executable code
├── references/            ← optional: docs loaded on demand
└── assets/                ← optional: templates, data files
```

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
  python:
    - pdfplumber

# ── Tool permissions ──────────────────────────────────────────────────────────
allowed-tools: Bash(ffmpeg:*) Read Write

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
| `requires` | No | ✅ nested map | Load-time gating |
| `allowed-tools` | No | ✅ | Tool allowlist |
| `license` | No | ✅ | License identifier |
| `metadata` | No | ✅ key-value map | Author, version, homepage |

---

## 3. The `requires` block — load-time gating

Skills that fail the gate are never injected into the system prompt.
The agent never sees a skill it cannot use.

```yaml
requires:
  bins:
    - ffmpeg          # must exist on PATH
    - jq
  env:
    - OPENAI_API_KEY  # must be set in environment
  python:
    - pdfplumber      # must be importable
    - pypdf
  config:
    - browser.enabled # must be truthy in agent config
```

Runtime implementation in `list_skills()`:

```python
def _passes_gate(skill: Skill) -> bool:
    req = skill.requires  # parsed from frontmatter into Skill dataclass
    for bin_ in req.get("bins", []):
        if not shutil.which(bin_):
            return False
    for env_ in req.get("env", []):
        if not os.environ.get(env_):
            return False
    for pkg in req.get("python", []):
        if importlib.util.find_spec(pkg) is None:
            return False
    return True
```

---

## 4. The `description` field — dual purpose

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

## 5. `tags` and `triggers` — scoring vocabulary

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

## 6. Progressive disclosure — body and resources

```
Tier 1 — Discovery  (~100 tokens):  name + description, loaded at startup for ALL skills
Tier 2 — Activation (<5000 tokens): full SKILL.md body, loaded when skill is selected
Tier 3 — Resources  (as needed):    scripts/, references/, assets/ loaded on demand
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

## 7. Complete example

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
  python:
    - pdfplumber
    - pypdf
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

```python
import pdfplumber

with pdfplumber.open("input.pdf") as pdf:
    for page in pdf.pages:
        print(page.extract_text())
```

## Merge PDFs

```python
from pypdf import PdfWriter

writer = PdfWriter()
for path in ["a.pdf", "b.pdf"]:
    writer.append(path)
writer.write("merged.pdf")
```

## Edge cases

- Scanned PDFs: pdfplumber returns empty text — use OCR instead.
- Encrypted PDFs: pass `password=` to `pdfplumber.open()`.

For the full API reference, see [references/api.md](references/api.md).
```

---

## 8. Authoring checklist

- [ ] Directory name matches `name` exactly (lowercase, hyphens only)
- [ ] `description` answers *what* + *when* in ≤1024 chars
- [ ] Every `triggers` term appears in `description` or `name`
- [ ] `requires` lists all runtime dependencies (bins, env, python)
- [ ] Body is under 500 lines; large reference material in `references/`
- [ ] `scripts/` files are self-contained with clear error messages
