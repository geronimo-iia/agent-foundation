---
title: "What are Tools?"
summary: "Tools are executable capabilities that agents can invoke to interact with systems, APIs, and external resources beyond text generation."
read_when:
  - Understanding what tools are and how they work
  - Designing tool systems for agents
  - Implementing tool execution and security
  - Establishing tool specifications and contracts
status: active
last_updated: "2025-01-16"
---

# What are Tools?

> **Specification only**: This specification is stable but has no tooling implementation yet. It may benefit from further refinement as implementation experience is gained.

Tools are executable capabilities that extend agent functionality beyond text generation. They enable agents to interact with systems, APIs, files, databases, and external resources through well-defined interfaces.

## Core Concept

A tool is a **named function** with:

- **Input schema**: Parameters the tool accepts
- **Output schema**: Data the tool returns
- **Execution logic**: Code that performs the action
- **Security constraints**: Permissions and access controls

## Tool Categories

- **System Tools**: File operations, shell commands, process management
- **Network Tools**: HTTP requests, API calls, webhooks
- **Data Tools**: Database queries, data transformations, search
- **Integration Tools**: Third-party service interactions (GitHub, AWS, etc.)

## Tool vs Skill

- **Tools**: Executable functions agents invoke
- **Skills**: Knowledge and instructions agents follow

Tools provide **capabilities**, skills provide **knowledge**.

## Design Principles

- **Explicit invocation**: Agents explicitly call tools by name
- **Type safety**: Strong input/output schemas
- **Sandboxing**: Execution isolation and permission controls
- **Observability**: Logging and monitoring of tool usage
- **Idempotency**: Safe to retry when possible

## Related Specifications

This topic will cover:

- Tool specification format
- Tool execution model
- Security and permissions
- Tool discovery and registration
- Error handling and retries
