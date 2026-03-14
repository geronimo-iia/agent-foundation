---
title: "Workspace Rule Implementation"
summary: "Implementation patterns for workspace-level AI behavior rules, portable across Amazon Q, Cursor, Copilot, and other AI systems."
read_when:
  - Implementing workspace rules in new projects
  - Setting up AI behavior enforcement systems
  - Adapting rule patterns to different workspaces or AI systems
  - Understanding rule implementation architecture
status: active
last_updated: "2026-07-15"
---

# Workspace Rule Implementation

## Implementation Architecture

### Directory Structure
```
workspace-root/
└── .amazonq/rules/
    ├── semantic-commits.md
    ├── documentation-standards.md
    ├── project-organization.md
    └── workspace-maintenance.md
```

### Rule Categories

**Enforcement Rules**
- `semantic-commits.md` - Commit message format requirements
- `documentation-standards.md` - Frontmatter and authoring requirements

**Organization Rules**
- `project-organization.md` - Workspace structure and project purposes
- `workspace-maintenance.md` - Rules for maintaining rule consistency

## Core Patterns

### Reference Pattern
- Point to specifications, don't duplicate
- Use imperative language ("must", "always")
- Include enforcement statement

### Organization Pattern  
- Project structure diagram
- Quick reference guide
- Clear project purposes and rules

### Maintenance Pattern
- Explicit update requirements
- Verification steps
- Examples of triggering changes

## Implementation

1. Create `.amazonq/rules/` directory
2. Copy rule files from this workspace
3. Update paths to match target workspace
4. Adapt project structure to domain needs

## Rule Templates

Separate template files for easy copying:

- [semantic-commits.md](templates/semantic-commits.md) - Commit format enforcement
- [documentation-standards.md](templates/documentation-standards.md) - Frontmatter requirements
- [project-organization.md](templates/project-organization.md) - Workspace structure
- [workspace-maintenance.md](templates/workspace-maintenance.md) - Rule maintenance
- [minimal-code.md](templates/minimal-code.md) - Minimal code implementation standards

## Integration

- Rules → reference specifications
- Specifications → contain detailed knowledge  
- Tooling → implements standards
- Processes → documented in engineering specs

## Adapting for Other AI Systems

The four-rule structure and content patterns are portable across AI systems. Only the rules directory location changes:

| AI System | Rules Directory |
|---|---|
| Amazon Q | `.amazonq/rules/*.md` |
| Cursor | `.cursor/rules/*.md` |
| GitHub Copilot | `.github/copilot-instructions.md` |
| Claude (Anthropic) | `CLAUDE.md` (single file) |
| OpenAI Codex | `AGENTS.md` (single file) |

For single-file systems (Claude, Codex), concatenate the four rule files into one, preserving the section structure.

**What transfers unchanged**: rule categories, reference pattern, enforcement language, maintenance pattern.

**What needs adapting**: directory path in `documentation-standards.md` and `semantic-commits.md` references, project structure diagram in `project-organization.md`.