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
├── schemas/        # JSON schemas for validation (see schemas/README.md)
```

## Key Specifications

- **[Skills](skills/)** — Skill format, lifecycle, hub protocols, authoring guide
- **[Information Flow Tracking](information-flow/)** — Taint tracking, TSL policy language, audit events
- **[Documentation](docs/)** — Document format, hub protocols, organization standards
- **[Agents](agents/)** — Agent definitions, manifest formats, registry protocols
- **[Tools](tools/)** — Tool definitions, MCP protocol, execution policies
- **[Bootstrap](bootstrap/)** — Minimal setup procedures for agent workspaces
- **[Schemas](schemas/)** — JSON schemas for validation (see [schemas/README.md](schemas/README.md))

## Maturity

| Domain           | Spec     | Tooling                                                                            |
| ---------------- | -------- | ---------------------------------------------------------------------------------- |
| Skills           | ✅ stable | ✅ [agentctl](https://github.com/geronimo-iia/agentctl) validates + generates index |
| Docs             | ✅ stable | ✅ [agentctl](https://github.com/geronimo-iia/agentctl) validates + generates index |
| Schemas          | ✅ stable | ✅ used by agentctl                                                                 |
| Bootstrap        | ✅ stable | —                                                                                  |
| Agents           | ✅ stable | ⬜ no implementation yet                                                            |
| Tools / MCP      | ✅ stable | ⬜ no implementation yet                                                            |
| Information Flow | ✅ stable | ⬜ no implementation yet                                                            |

## License

[MIT](LICENSE)

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md).