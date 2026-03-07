---
title: "What are Skills?"
summary: "Skills are a lightweight, open format for extending AI agent capabilities with specialized knowledge and workflows."
read_when:
  - Understanding what skills are and how they work
  - Learning about the SKILL.md format and structure
  - Implementing skills support in an agent system
status: active
last_updated: "2025-07-14"
---

# What are Skills?

Skills package domain expertise, new capabilities, and repeatable workflows that agents can use. At its core, a skill is a folder containing a `SKILL.md` file with metadata and instructions that tell an agent how to perform a specific task.

```
my-skill/
├── SKILL.md          # Required: name, description, instructions
├── lifecycle.yaml    # Optional: install/update/uninstall commands
├── scripts/          # Optional: executable scripts
├── references/       # Optional: documentation
└── assets/           # Optional: templates, resources
```

## Key benefits

- **Self-documenting**: A skill author or user can read a `SKILL.md` and understand what it does, making skills easy to audit and improve
- **Interoperable**: Skills work across any agent that implements the Agent Skills specification
- **Extensible**: Skills can range in complexity from simple text instructions to bundled scripts, templates, and reference materials
- **Shareable**: Skills are portable and can be easily shared between projects and developers

## How skills work

Skills use **progressive disclosure** to manage context efficiently:

1. **Discovery**: At startup, agents load only the name and description of each available skill, just enough to know when it might be relevant.

2. **Activation**: When a task matches a skill's description, the agent reads the full `SKILL.md` instructions into context.

3. **Execution**: The agent follows the instructions, optionally loading referenced files or executing bundled code as needed.

This approach keeps agents fast while giving them access to more context on demand.

## The SKILL.md file

Every skill starts with a `SKILL.md` file containing YAML frontmatter and Markdown instructions:

```yaml
---
name: pdf-processing
description: Extract text and tables from PDF files, fill forms, merge documents.
---

# PDF Processing

## When to use this skill
Use this skill when the user needs to work with PDF files...

## How to extract text
1. Use pdfplumber for text extraction...

## How to fill forms
...
```

The following frontmatter is required at the top of `SKILL.md`:

- `name`: A short identifier
- `description`: When to use this skill

The Markdown body contains the actual instructions and has no specific restrictions on structure or content.

## Format compatibility

This specification is compatible with the [Agent Skills](https://agentskills.io) format, with extensions:

**Core compatibility** (shared with agentskills.io):
- SKILL.md file with YAML frontmatter
- Required fields: `name`, `description`
- Optional fields: `license`, `compatibility`, `metadata`
- Directory structure with optional `scripts/`, `references/`, `assets/`

**Extensions** (this specification):
- `lifecycle.yaml` file for agent-assisted install/update/uninstall
- Mode-specific skills (`skills-{mode}/` directories) — from [Kilo](https://kilo.ai/docs/customize/skills)
- Priority/override rules (project > global, mode-specific > generic) — from [Kilo](https://kilo.ai/docs/customize/skills)
- Variable substitution in lifecycle commands

**Authoring best practices** (incorporated from [mgechev/skills-best-practices](https://github.com/mgechev/skills-best-practices)):
- Negative triggers in descriptions to prevent false activations
- Third-person imperative tone in skill body
- No human-oriented prose in skill body
- Flat hierarchy rule (one level deep)
- Scripts stdout/stderr contract
- LLM validation workflow

Basic Agent Skills will work without modification. The extensions are optional and backward-compatible.

## Related specifications

- [authoring-guide.md](authoring-guide.md) — Detailed authoring standards and best practices
- [hub.md](hub.md) — Multi-source skill registry and distribution system