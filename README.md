# Agent Foundation

Specifications and foundational patterns for multi-agent systems.

## Purpose

This repository contains stable specifications only — no implementation. It defines foundational patterns for:

- Multi-agent system architecture and protocols
- Agent identity, capabilities, and security models
- Skill format, lifecycle, and hub protocols
- Documentation standards and organization
- Information flow tracking and taint analysis
- Workspace bootstrap and setup procedures

## Structure

```
/
├── agents/         # Agent definitions, manifests, registry protocols
├── skills/         # Skill definitions, formats, hub protocols
├── docs/           # Documentation definitions, formats, hub protocols
├── tools/          # Tool definitions, MCP protocol, execution policies
├── information-flow/ # Information flow tracking and taint analysis
├── bootstrap/      # Workspace setup and bootstrap guides
├── rules/          # Workspace rules best practices and templates
├── schemas/        # JSON schemas for validation (see schemas/README.md)
```

## Getting Started

See [bootstrap/development-environment.md](bootstrap/development-environment.md) for the full setup guide.

To bootstrap your workspace, paste this prompt into Amazon Q, Cursor, Zed, or any AI-enabled IDE:

> Bootstrap my agent workspace following @agent-foundation/bootstrap/development-environment.md. Install agentctl, clone the required repositories, register the agent-skills hub, and install the agentctl skill.

### Essential reading

- [bootstrap/development-environment.md](bootstrap/development-environment.md) — full workspace setup guide
- [rules/best-practices.md](rules/best-practices.md) — how to write workspace rules for AI behavior
- [skills/authoring-guide.md](skills/authoring-guide.md) — how to write and publish skills
- [docs/authoring-guide.md](docs/authoring-guide.md) — documentation format and frontmatter standards
- [skills/hub.md](skills/hub.md) — hub protocol and index format

## Key Specifications

### Mature specifications & Tooling

- **[Skills](skills/definition.md)** — Skill format, lifecycle, hub protocols, authoring guide
- **[Documentation](docs/definition.md)** — Document format, hub protocols, organization standards
- **[Schemas](schemas/)** — JSON schemas for validation (see [schemas/README.md](schemas/README.md))
- **[Bootstrap](bootstrap/definition.md)** — Minimal setup procedures for agent workspaces
- **[Rules](rules/definition.md)** — Workspace rules best practices and templates for AI behavior enforcement

### Specification only

> These specifications are stable but have no tooling implementation yet. They may benefit from further refinement as implementation experience is gained.

- **[Agents](agents/definition.md)** — Agent definitions, manifest formats, registry protocols
- **[Tools](tools/definition.md)** — Tool definitions, MCP protocol, execution policies
- **[Information Flow Tracking](information-flow/definition.md)** — Taint tracking, TSL policy language, audit events

## Tooling

**[agentctl](https://github.com/geronimo-iia/agentctl)** is the CLI companion to this repository. It implements the hub protocols defined here:

- Validate and generate `index.json` for skill and doc hubs
- Manage hub registry (`hub add/list/remove/enable/disable/refresh`)
- Install, update, and remove skills (`skill install/list/update/remove`)
- Manage local config (`config init/show/get/set`)

```sh
brew tap geronimo-iia/agent && brew install agentctl
# or
cargo install agent-ctl
```

## License

[MIT](LICENSE.md)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).