# Agent Foundation

Final specifications and foundational patterns for multi-agent systems.

## Purpose

This repository contains stable specifications only — no implementation. It defines foundational patterns for:

- Multi-agent system architecture and protocols
- Agent identity, capabilities, and security models
- Documentation standards and organization
- Workspace bootstrap and setup procedures

## Structure

```
/
├── agents/         # Agent definitions, manifests, registry protocols
├── skills/         # Skill definitions, formats, hub protocols
├── docs/           # Documentation definitions, formats, hub protocols
├── tools/          # Tool definitions, MCP protocol, execution policies
├── bootstrap/      # Workspace setup and bootstrap guides
├── schemas/        # JSON schemas for validation (see schemas/README.md)
```

## Contents

- **Agent Specifications** — Agent definitions, manifest formats, registry protocols
- **Skill Specifications** — Skill format, lifecycle, hub protocols, authoring standards
- **Documentation Specifications** — Document format, hub protocols, organization standards
- **Tool Specifications** — Tool definitions, MCP protocol, execution policies
- **Bootstrap Guides** — Minimal setup procedures for agent workspaces
- **JSON Schemas** — Validation schemas for all formats (see [schemas/README.md](schemas/README.md))
- **Stable Specifications** — Concrete schemas, contracts, and protocols