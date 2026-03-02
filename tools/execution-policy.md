---
title: "Tool Execution Policy"
summary: "Execution policy for autonomous agents: unrestricted by default with optional enforcement modes for bounded deployments."
read_when:
  - Designing task execution systems
  - Implementing agent tool invocation
  - Establishing security policies for agent operations
  - Understanding tool execution constraints
status: active
last_updated: "2025-01-16"
---

# Tool Execution Policy

## 1. Core Principle

**Agents execute tools with user authority**. Enforcement is optional and deployment-specific.

---

## 2. Design Philosophy

**Trust through transparency**: Autonomous agents act on behalf of users. Security comes from visibility, audit trails, and user consent.

**Optional enforcement**: Implementations MAY provide enforcement mechanisms. Deployments choose their security model.

**No mandatory restrictions**: Specifications do not require MCP protocol or sandboxing.

---

## 3. Execution Models

### Unrestricted Execution

**Model**: Agent executes tools directly with user privileges

**Characteristics**:
- No validation of tool calls
- No process isolation
- No capability restrictions
- Full access to user resources

**Security**: Transparency and audit logging

**Suitable for**:
- Single-user desktop environments
- Trusted skill sources
- Users accepting full risk

### Enforced Execution

**Model**: Agent validates tool calls against declared permissions

**Characteristics**:
- Tool calls validated against skill `allowed-tools` declaration
- Unauthorized calls denied or logged
- Optional process isolation (MCP, WASM, containers)
- Explicit capability grants

**Security**: Least privilege and isolation

**Suitable for**:
- Multi-user environments
- Untrusted skill sources
- Regulated deployments

---

## 4. Tool Declaration

Skills declare required tools in `allowed-tools` field:

```yaml
allowed-tools: Bash(ffmpeg:*) Read(*.pdf) Write(output/*) MCP(python-tools:*)
```

**Syntax**:
- `Bash(command:args)`: Shell command with argument pattern
- `Read(pattern)`: File read with glob pattern
- `Write(pattern)`: File write with glob pattern  
- `MCP(server:tool)`: MCP server and tool name
- `*`: Wildcard matching

**Purpose**: Informational for unrestricted mode, enforced in restricted mode

---

## 5. Enforcement Specification

### Enforcement Modes

Implementations MAY support enforcement modes:

**unrestricted**: No validation, all tools allowed
**enforce**: Validate against `allowed-tools`, deny violations
**audit**: Validate against `allowed-tools`, log violations but allow

### Validation Rules

When enforcement enabled:

**Tool call allowed if**:
1. Skill declares tool in `allowed-tools`, OR
2. Agent manifest grants capability, OR  
3. User grants permission interactively

**Tool call denied if**:
1. None of the above conditions met

**Denied calls**:
- MUST be logged with violation details
- MUST return error to agent
- MAY prompt user for permission

---

## 6. Isolation Mechanisms

Implementations MAY provide isolation:

**MCP Protocol**: Tools execute in separate MCP server processes
**WASM Sandboxing**: Skills bundle WASM modules with WASI capabilities
**OS Sandboxing**: Platform-specific isolation (AppArmor, SELinux, macOS sandbox)
**Container Isolation**: Docker/Podman with resource limits

**Specification does not mandate isolation mechanism**.

---

## 7. Logging Requirements

Implementations MUST log tool executions:

**Required fields**:
- Timestamp
- Tool name and arguments
- Execution context (skill, agent, task)
- Result or error
- Enforcement decision (if applicable)

**Log retention**: Implementation-specific

**Audit trail**: MUST be accessible to user

---

## 8. Design Principles

**Default to freedom**: Unrestricted execution is valid and supported
**User choice**: Deployments choose their security model
**Transparency first**: Logging before enforcement
**Declarative policy**: Skills declare needs, runtime decides enforcement
**Fail open**: Enforcement failures SHOULD log and allow, not break agent
**Backward compatibility**: Enforcement is additive, not breaking

---

## Related Specifications

- [MCP Protocol](mcp-protocol.md) - Optional tool execution protocol
- [Tool Definition](definition.md) - What tools are
- [Skill Authoring](../skills/authoring-guide.md) - How to declare allowed-tools
- [Agent Manifest](../agents/agent-manifest-specification.md) - Agent capabilities
