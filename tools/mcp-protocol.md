---
title: "Model Context Protocol (MCP)"
summary: "Anthropic's standard protocol for connecting AI models to tools, resources, and data sources through a client-server architecture."
read_when:
  - Implementing tool servers for agents
  - Integrating external tools and resources
  - Understanding MCP architecture and capabilities
  - Building MCP-compatible systems
status: active
last_updated: "2025-01-16"
---

# Model Context Protocol (MCP)

## 1. Overview

**Model Context Protocol (MCP)** is an open standard by Anthropic that enables AI models to securely connect to tools, resources, and data sources through a standardized client-server architecture.

**Purpose**: Decouple tool/resource providers from AI applications, enabling reusable integrations.

---

## 2. Architecture

### Client-Server Model

```
┌─────────────┐         ┌─────────────┐         ┌─────────────┐
│   AI Host   │ ◄─────► │ MCP Client  │ ◄─────► │ MCP Server  │
│  (Claude)   │         │             │         │   (Tools)   │
└─────────────┘         └─────────────┘         └─────────────┘
```

**MCP Host**: AI application (Claude Desktop, IDE, custom app)
**MCP Client**: Protocol implementation in host
**MCP Server**: Tool/resource provider process

**Communication**: JSON-RPC 2.0 over stdio, HTTP, or WebSocket

---

## 3. Core Capabilities

### Tools

**Definition**: Functions the model can invoke

**Schema**:
```json
{
  "name": "read_file",
  "description": "Read contents of a file",
  "inputSchema": {
    "type": "object",
    "properties": {
      "path": { "type": "string" }
    },
    "required": ["path"]
  }
}
```

**Invocation**: Model requests tool execution, server returns result

### Resources

**Definition**: Data sources the model can read

**Types**:
- Files
- Database records
- API responses
- Live data streams

**URI-based**: `file:///path/to/file`, `db://table/record`

### Prompts

**Definition**: Reusable prompt templates with parameters

**Use case**: Standardized workflows across applications

---

## 4. Protocol Messages

### Tool Discovery

**Request**: `tools/list`
**Response**: Array of tool definitions with schemas

### Tool Execution

**Request**: `tools/call`
```json
{
  "name": "read_file",
  "arguments": {
    "path": "/etc/hosts"
  }
}
```

**Response**: Tool result or error

### Resource Access

**Request**: `resources/read`
```json
{
  "uri": "file:///path/to/file"
}
```

**Response**: Resource contents

---

## 5. Security Model

MCP provides security capabilities through its architecture:

**Sandboxing**: Servers run in isolated processes
**Permissions**: Host can control which servers run
**User consent**: Host can require approval for tool invocations
**Audit logging**: Protocol supports logging all tool calls

**Note**: These are MCP capabilities. The [execution policy](execution-policy.md) for this specification uses unrestricted execution with logging only

---

## 6. MCP in Agent Systems

### Skills + MCP

**Skills**: Provide knowledge and instructions
**MCP Tools**: Provide executable capabilities

**Example**: Skill describes "how to analyze code", MCP tool provides "read_file" capability

### Hub Distribution

MCP servers can be distributed via skill hubs:
- Skill includes MCP server configuration
- Installation sets up MCP server
- Skill instructions reference MCP tools

---

## 7. Design Principles

**Separation of concerns**: Tools separate from AI logic
**Reusability**: One server, many applications
**Security**: Process isolation and permission controls
**Extensibility**: Easy to add new tools and resources
**Interoperability**: Standard protocol across vendors

---

## 8. Use Cases

**Development Tools**: Git, file operations, code execution
**Data Access**: Database queries, API integrations
**System Integration**: Cloud services (AWS, GitHub, Slack)
**Custom Workflows**: Domain-specific tools and resources

---

## References

- [MCP Specification](https://spec.modelcontextprotocol.io/) - Official protocol specification
- [MCP Documentation](https://modelcontextprotocol.io/) - Implementation guides
- [MCP GitHub](https://github.com/modelcontextprotocol) - Reference implementations and SDKs
- [Tool Definition](definition.md) - General tool concepts
