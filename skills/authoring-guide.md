---
title: "Skill Authoring"
summary: "How to author SKILL.md files with machine-readable YAML frontmatter."
read_when:
  - Writing or reviewing a SKILL.md file
  - Implementing load-time gating or keyword scoring
  - Designing the requires block or description field
status: active
last_updated: "2025-07-14"
---

# How to Write Skills — Machine-Readable YAML Frontmatter

The SKILL.md body is for the agent (natural language instructions).
The YAML frontmatter is for the **runtime** — discovery, gating, scoring, injection.
Every frontmatter field must be parseable and actionable by code.

---

## 1. Directory structure

```
my-skill/                  ← directory name must match `name` field exactly
├── SKILL.md               ← required: name, description, instructions
├── lifecycle.yaml         ← optional: install/update/uninstall commands
├── scripts/               ← optional: executable scripts
├── references/            ← optional: docs loaded on demand
└── assets/                ← optional: templates, resources
```

**Flat hierarchy rule**: Skills must be exactly one level deep — `my-skill/SKILL.md`. Nested skill directories (e.g., `my-skill/sub-skill/SKILL.md`) are not supported and will not be discovered.

---

## 1a. Skill locations and priority

### Skill directories

Skills are loaded from multiple locations:

```
~/.agent/skills/                    # Global skills (all modes)
~/.agent/skills-code/               # Global code mode only
~/.agent/skills-architect/          # Global architect mode only

<project>/.agent/skills/            # Project skills (all modes)
<project>/.agent/skills-code/       # Project code mode only
<project>/.agent/skills-architect/  # Project architect mode only
```

**Mode-specific directories**: Use `skills-{mode-slug}/` pattern where `{mode-slug}` matches the agent mode identifier (e.g., `code`, `architect`, `debug`, `test`).

### Priority rules

When multiple skills share the same `name`, resolution follows these rules:

1. **Project skills override global skills** — A skill in `<project>/.agent/skills/` takes precedence over `~/.agent/skills/`
2. **Mode-specific skills override generic skills** — A skill in `skills-code/` overrides the same skill in `skills/` when in code mode

Combined priority (highest to lowest):

```
1. <project>/.agent/skills-{mode}/my-skill/     # Project + mode-specific
2. <project>/.agent/skills/my-skill/            # Project + generic
3. ~/.agent/skills-{mode}/my-skill/             # Global + mode-specific
4. ~/.agent/skills/my-skill/                    # Global + generic
```

**Use cases**:

- Define global skills for personal use across all projects
- Override them per-project when project-specific behavior is needed
- Customize skills per mode (e.g., different code review rules for code vs architect mode)

---

## 2. Full frontmatter schema

```yaml
---
# ── Identity ─────────────────────────────────────────────────────────────────
name: my-skill
description: >
  What this skill does and when to use it. Write descriptions that match
  how users phrase requests. Use when the user needs to [task].

# ── Compatibility ────────────────────────────────────────────────────────────
compatibility: Requires ffmpeg, OPENAI_API_KEY environment variable, network access

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
| `description` | Yes | ✅ | Discovery + activation signal |
| `compatibility` | No | ❌ | Human-readable environment requirements (1-500 chars) |
| `license` | No | ✅ | License identifier |
| `metadata` | No | ✅ key-value map | Author, version, homepage |

---

## 3. The `compatibility` field

The `compatibility` field describes environment requirements in human-readable text. The agent reads this to determine if the skill can be used.

```yaml
compatibility: Requires ffmpeg, OPENAI_API_KEY environment variable, macOS 12+, network access
```

**Guidelines**:

- Keep it 1-500 characters
- List required binaries, environment variables, OS requirements, network access, etc.
- Be specific but concise
- The agent will read this and use `lifecycle.install` to fix missing dependencies

**Examples**:

```yaml
# Good - specific and actionable
compatibility: Requires ffmpeg, pdfplumber Python package, 2GB disk space

# Good - OS and network requirements
compatibility: macOS 12+, requires network access for model downloads

# Too vague
compatibility: Needs some tools installed
```

---

## 4. The `lifecycle.yaml` file — agent-assisted management

Skills can include an optional `lifecycle.yaml` file with install/update/uninstall commands. This file is only loaded when the user runs lifecycle operations, keeping SKILL.md lightweight.

```yaml
variables:
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

- `${SKILL_NAME}`: Skill name from frontmatter (e.g., `marker-pdf`)
- `${SKILL_PATH}`: Absolute path to skill installation directory (set by agent)
- `${HOME}`: User home directory
- `${PLATFORM}`: Current platform (`linux`, `macos`, `windows`)

**Custom variables**: Defined in `lifecycle.variables` block

### Variable Evaluation

Variables are evaluated in this order:

1. **Built-in variables** are initialized first:
   ```python
   SKILL_NAME = "marker-pdf"       # from frontmatter
   SKILL_PATH = "~/.agent/skills/marker-pdf"  # from agent
   HOME = "/Users/username"        # from system
   PLATFORM = "macos"              # from system
   ```

2. **Custom variables** are evaluated in declaration order:
   ```yaml
   variables:
     VENV_PATH: ${SKILL_PATH}/.venv
     PYTHON_BIN: ${VENV_PATH}/bin/python
   ```
   
   Evaluation:
   ```python
   # Step 1: VENV_PATH (uses built-in SKILL_PATH)
   VENV_PATH = "${SKILL_PATH}/.venv"
   VENV_PATH = "~/.agent/skills/marker-pdf/.venv"
   
   # Step 2: PYTHON_BIN (can use VENV_PATH)
   PYTHON_BIN = "${VENV_PATH}/bin/python"
   PYTHON_BIN = "~/.agent/skills/marker-pdf/.venv/bin/python"
   ```

3. **Commands** are evaluated with all variables:
   ```yaml
   command: uv venv ${VENV_PATH} && uv pip install --python ${PYTHON_BIN} marker-pdf
   ```
   
   Result:
   ```bash
   uv venv ~/.agent/skills/marker-pdf/.venv && \
   uv pip install --python ~/.agent/skills/marker-pdf/.venv/bin/python marker-pdf
   ```

**Rules**:

- Variables are case-sensitive
- Forward references not allowed (variable must be defined before use)
- Circular references cause error
- Unknown variables left as-is (e.g., `${UNKNOWN}` stays `${UNKNOWN}`)

**Structure**:

- `variables`: Define custom variables with default values
- `install`: Commands to install missing dependencies
- `update`: Commands to update to newer versions
- `uninstall`: Commands to clean up dependencies and caches

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

The description serves two purposes:

1. **Discovery**: Helps users and agents understand what the skill does
2. **Activation**: The agent decides whether to use the skill based on this text

```yaml
# Bad — vague, no activation signal, no negative triggers
description: Helps with PDFs.

# Good — specific capability, positive triggers, negative triggers
description: >
  Extract text and tables from PDF files, fill PDF forms, and merge multiple PDFs.
  Use when the user needs to work with PDF documents, extract content, fill forms,
  or combine files. Do not use for image files, Word documents, or plain text files.
```

**Writing tips**:

- Write descriptions that match how users phrase requests
- Be specific about capabilities and when to use the skill
- Include **negative triggers**: explicitly state what the skill does NOT handle ("Do not use for…") — this prevents false activations
- Explicit invocation always works (e.g., "use the pdf-processing skill")
- Use YAML block scalar (`>`) to fold newlines into spaces for long descriptions

---

## 6. Skill body tone

The SKILL.md body is instructions for the agent, not documentation for humans.

- **Use third-person imperative**: Write steps as direct commands — "Extract the text", "Return the result", "Call the API"
- **No human-oriented prose**: Omit introductions, explanations of why, and background context — the agent needs what to do, not why it exists
- **No human docs in body**: Do not include usage examples written for human readers, README-style overviews, or marketing copy — move those to `references/` if needed

```markdown
# Bad — human-oriented prose
This skill helps you work with PDFs. PDFs are a common format and...
You can use this skill to extract text by following these steps:

# Good — third-person imperative
## Extract text
1. Open the PDF with pdfplumber.
2. Iterate pages and call `page.extract_text()`.
3. Return the concatenated result.
```

---

## 7. Scripts contract

Scripts in `scripts/` must follow this stdout/stderr contract so the agent can parse results reliably:

- **stdout**: Machine-readable output only — the result the agent will consume (JSON, plain value, file path)
- **stderr**: Human-readable status, progress messages, warnings, and errors
- **Exit codes**: `0` for success, non-zero for failure

```bash
#!/usr/bin/env bash
# Good — result on stdout, status on stderr
echo "Processing..." >&2
result=$(do_work "$1")
if [ $? -ne 0 ]; then
  echo "Failed to process $1" >&2
  exit 1
fi
echo "$result"   # stdout: agent reads this
```

Never mix status messages into stdout — the agent treats all stdout as the result.

---

## 8. Progressive disclosure

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

## 9. LLM validation workflow

Before publishing a skill, validate it with an LLM using this four-phase workflow:

**Phase 1 — Activation test**: Give the LLM only the `name` and `description`. Ask: "Would you activate this skill for the request: [sample request]?" Verify it activates on positive triggers and does NOT activate on negative trigger phrases.

**Phase 2 — Instruction clarity**: Give the LLM the full SKILL.md body. Ask it to follow the instructions for a sample task. If it asks clarifying questions or misinterprets steps, the instructions need revision.

**Phase 3 — Negative trigger coverage**: Give the LLM a set of requests that should NOT activate the skill. Confirm it correctly declines each one based on the negative triggers in the description.

**Phase 4 — Edge case handling**: Give the LLM the edge cases section and ask it to handle each scenario. Verify the instructions produce correct behavior.

A skill passes validation when all four phases complete without ambiguity or incorrect activation.

---

## 10. Complete example

```markdown
---
name: pdf-processing
description: >
  Extract text and tables from PDF files, fill PDF forms, and merge multiple PDFs.
  Use when the user needs to work with PDF documents, extract content, fill forms,
  or combine files.
compatibility: Requires pdfplumber and pypdf Python packages
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

**lifecycle.yaml**:

```yaml
variables:
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
```

---

## 11. Authoring checklist

- [ ] Directory name matches `name` exactly (lowercase, hyphens only)
- [ ] Skill placed in correct directory (`skills/` for generic, `skills-{mode}/` for mode-specific)
- [ ] `description` answers *what* + *when* in ≤1024 chars, matches how users phrase requests
- [ ] `description` includes negative triggers ("Do not use for…") to prevent false activations
- [ ] Skill body uses third-person imperative tone; no human-oriented prose or README-style content
- [ ] Scripts follow stdout/stderr contract: stdout is machine-readable result only, stderr is status/errors
- [ ] Skill validated with LLM using the four-phase workflow (activation, clarity, negative triggers, edge cases)
- [ ] `compatibility` lists environment requirements in 1-500 chars (if applicable)
- [ ] `lifecycle.yaml` provided with install/update/uninstall commands (if applicable)
- [ ] `lifecycle.yaml` variables defined for reusable paths
- [ ] Body is under 500 lines; large reference material in `references/`
- [ ] Skill directory is exactly one level deep (flat hierarchy — no nested skill dirs)
- [ ] If creating mode-specific variant, verify it overrides correctly

---

## Sources

- [agentskills.io](https://agentskills.io) — Base Agent Skills format specification
- [mgechev/skills-best-practices](https://github.com/mgechev/skills-best-practices) — Negative triggers, third-person imperative tone, no-human-docs rule, flat hierarchy rule, scripts stdout/stderr contract, LLM validation workflow
