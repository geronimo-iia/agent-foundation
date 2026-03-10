---
title: "Tool Execution Policy"
summary: "Autonomous agents execute tools with user authority. Security through transparency, logging, and user consent."
read_when:
  - Designing agent tool execution systems
  - Understanding agent security model
  - Implementing tool invocation and logging
status: active
last_updated: "2025-01-16"
---

# Tool Execution Policy

## 1. Core Principle

**Agents execute tools with user authority**. No restrictions, no sandboxing, no permission checks.

### Rationale

**Trust through transparency**: Autonomous agents act on behalf of users with user's authority. The user chose to run the agent and install skills (consent). Security comes from visibility and audit trails, not artificial restrictions.

**Simplicity first**: Complex permission systems and enforcement layers add overhead without guaranteeing security. Start simple, add restrictions only when deployment context requires them.

**User responsibility**: Users choose which skills to install and trust. The agent is an extension of the user, not a separate untrusted entity.

---

## 2. Execution Model

**Unrestricted execution**: Agent invokes tools directly with user privileges

**Characteristics**:

- No validation of tool calls
- No process isolation
- No capability restrictions
- Full access to user resources

**Security model**: Trust through transparency

- All tool executions logged
- Audit trail accessible to user
- User consent for skill installation

---

## 3. Logging Requirements

Implementations MUST log all tool executions:

**Required fields**:

- Timestamp
- Tool name and arguments
- Execution context (skill name, agent session)
- Result or error
- Exit code (for shell commands)

**Log retention**: Implementation-specific

**Audit trail**: MUST be accessible to user for review

---

## 4. User Consent

**Skill installation**: User explicitly installs skills

- Skills declare dependencies in `compatibility` field
- Skills may include `lifecycle.yaml` with install commands
- User approves all lifecycle commands individually

**Tool execution**: No per-execution approval required

- Agent executes tools as needed
- User trusts installed skills
- Logging provides audit trail

---

## 5. Design Principles

**Transparency over restriction**: Log everything, restrict nothing

**User responsibility**: User chooses which skills to install and trust

**Simplicity**: No complex permission systems or enforcement layers

**Auditability**: Complete record of agent actions

---

## 6. Future Considerations

Enforcement mechanisms (sandboxing, permission checks, isolation) are possible future enhancements but not part of this specification.

See agent-research for exploration of:
- MCP protocol for tool isolation
- WASM sandboxing for skills
- Permission-based execution models

---

## Related Specifications

- [Tool Definition](definition.md) — What tools are
- [Skill Authoring](../skills/authoring-guide.md) — How to write skills
- [Skill Lifecycle](../skills/lifecycle.md) — Skill installation and management
