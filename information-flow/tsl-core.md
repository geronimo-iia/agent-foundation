---
title: "Taint Specification Language (TSL) Core"
summary: "Declarative YAML format for specifying taint policies - sources, sinks, and basic propagation rules"
read_when:
  - Defining taint policies declaratively for agent systems
  - Creating portable taint specifications across implementations
  - Implementing policy-as-code for information flow tracking
status: active
last_updated: "2025-01-16"
---

# Taint Specification Language (TSL) Core

## Overview

TSL Core provides a declarative YAML format for defining information flow tracking policies. Policies are specified externally and loaded at runtime, enabling policy-as-code and cross-implementation portability.

**Version**: 1.0  
**Format**: YAML  
**Schema**: [tsl-policy.json](../schemas/tsl-policy.json)

## Policy Structure

```mermaid
graph TB
    Policy["Taint Policy"]
    Policy --> Metadata["metadata"]
    Policy --> TaintKinds["taint_kinds"]
    Policy --> Sources["sources"]
    Policy --> Sinks["sinks"]
    
    TaintKinds --> K1["user_input"]
    TaintKinds --> K2["external_fetch"]
    TaintKinds --> K3["llm_generated"]
    TaintKinds --> K4["secret"]
    TaintKinds --> K5["pii"]
    
    Sources --> S1["user_message"]
    Sources --> S2["web_fetch"]
    Sources --> S3["secret_vault"]
    
    Sinks --> Si1["shell_execute"]
    Sinks --> Si2["network_send"]
    Sinks --> Si3["llm_prompt"]
    
    style Policy fill:#e3f2fd
    style TaintKinds fill:#f3e5f5
    style Sources fill:#e8f5e8
    style Sinks fill:#fce4ec
```

## Information Flow Example

```mermaid
flowchart LR
    subgraph "Sources (from policy)"
        U["👤 user_message<br/>taints: [user_input]"]
        W["🌐 web_fetch<br/>taints: [external_fetch]"]
        V["🔐 secret_vault<br/>taints: [secret]"]
    end
    
    subgraph "Data Flow"
        U --> D1["🏷️ {user_input}"]
        W --> D2["🏷️ {external_fetch}"]
        V --> D3["🏷️ {secret}"]
        
        D1 --> M["🔄 Merge<br/>{user_input, external_fetch}"]
        D2 --> M
    end
    
    subgraph "Sinks (from policy)"
        M --> S1{"🚫 shell_execute<br/>blocked: [user_input,<br/>external_fetch, llm_generated]"}
        M --> S2{"📡 network_send<br/>blocked: [secret, pii]"}
        D3 --> S3{"📝 llm_prompt<br/>blocked: [secret]"}
    end
    
    subgraph "Results"
        S1 --> R1["❌ BLOCKED<br/>Contains user_input"]
        S2 --> R2["✅ ALLOWED<br/>No blocked taints"]
        S3 --> R3["❌ BLOCKED<br/>Contains secret"]
    end
    
    style U fill:#e3f2fd
    style W fill:#e3f2fd
    style V fill:#e3f2fd
    style D1 fill:#f3e5f5
    style D2 fill:#f3e5f5
    style D3 fill:#f3e5f5
    style M fill:#f3e5f5
    style S1 fill:#fce4ec
    style S2 fill:#fce4ec
    style S3 fill:#fce4ec
    style R1 fill:#ffebee
    style R2 fill:#e8f5e8
    style R3 fill:#ffebee
```

This diagram shows how TSL policy definitions control information flow from sources to sinks.

## Core Components

### 1. Metadata

```yaml
metadata:
  name: "Agent System Taint Policy"
  version: "1.0.0"
  description: "Information flow tracking policy for multi-agent system"
  author: "Security Team"
```

### 2. Taint Kinds

Define taint categories with descriptions:

```yaml
taint_kinds:
  user_input:
    description: "Data from user messages or interface input"
    
  external_fetch:
    description: "Data from external APIs, web requests, or file reads"
    
  llm_generated:
    description: "Content produced by language model inference"
    
  secret:
    description: "Credentials, API keys, or sensitive configuration"
    
  pii:
    description: "Personally identifiable information"
```

**Rules**:

- Taint kinds are flat (no hierarchies)
- Each kind has a unique name and description
- Names use snake_case

### 3. Sources

Define where taint originates:

```yaml
sources:
  user_message:
    taints: [user_input]
    pattern: "user:*"
    description: "User messages in chat interface"
    
  web_fetch:
    taints: [external_fetch]
    pattern: "url:*"
    description: "HTTP requests to external URLs"
    
  secret_vault:
    taints: [secret]
    pattern: "vault:*"
    description: "Secrets retrieved from vault"
    
  llm_inference:
    taints: [llm_generated]
    pattern: "agent:*"
    description: "LLM model outputs"
```

**Fields**:

- `taints` - Array of taint kinds assigned to data from this source
- `pattern` - Glob pattern for source identifiers (optional)
- `description` - Human-readable description

### 4. Sinks

Define output boundaries and blocked taints:

```yaml
sinks:
  shell_execute:
    description: "Shell command execution"
    blocked_taints: [user_input, external_fetch, llm_generated]
    reason: "Prevent command injection attacks"
    
  network_send:
    description: "Network requests and responses"
    blocked_taints: [secret, pii]
    reason: "Prevent data exfiltration"
    
  llm_prompt:
    description: "Input to language model"
    blocked_taints: [secret]
    reason: "Prevent secret leakage to model"
    
  disk_log:
    description: "Disk logging"
    blocked_taints: [secret, pii]
    reason: "Prevent sensitive data in logs"
    
  memory_store:
    description: "In-memory data storage"
    blocked_taints: []
    reason: "All taints permitted in memory"
```

**Fields**:

- `description` - Human-readable description
- `blocked_taints` - Array of taint kinds blocked at this sink
- `reason` - Rationale for the policy

## Complete Example

```yaml
# taint-policy.yaml
version: "1.0"

metadata:
  name: "Agent System Taint Policy"
  version: "1.0.0"
  description: "Information flow tracking policy for multi-agent system"
  author: "Security Team"

taint_kinds:
  user_input:
    description: "Data from user messages or interface input"
  external_fetch:
    description: "Data from external APIs, web requests, or file reads"
  llm_generated:
    description: "Content produced by language model inference"
  secret:
    description: "Credentials, API keys, or sensitive configuration"
  pii:
    description: "Personally identifiable information"

sources:
  user_message:
    taints: [user_input]
    pattern: "user:*"
    description: "User messages in chat interface"
    
  web_fetch:
    taints: [external_fetch]
    pattern: "url:*"
    description: "HTTP requests to external URLs"
    
  secret_vault:
    taints: [secret]
    pattern: "vault:*"
    description: "Secrets retrieved from vault"
    
  llm_inference:
    taints: [llm_generated]
    pattern: "agent:*"
    description: "LLM model outputs"

sinks:
  shell_execute:
    description: "Shell command execution"
    blocked_taints: [user_input, external_fetch, llm_generated]
    reason: "Prevent command injection attacks"
    
  network_send:
    description: "Network requests and responses"
    blocked_taints: [secret, pii]
    reason: "Prevent data exfiltration"
    
  llm_prompt:
    description: "Input to language model"
    blocked_taints: [secret]
    reason: "Prevent secret leakage to model"
    
  disk_log:
    description: "Disk logging"
    blocked_taints: [secret, pii]
    reason: "Prevent sensitive data in logs"
```

## Propagation Model

TSL Core uses **explicit union-based propagation**:

```
taint(f(a, b)) = taint(a) ∪ taint(b)
```

When values are combined, their taint sets merge. This is the only propagation rule in TSL Core v1.0.

## Policy Loading

Implementations MUST:
1. Parse YAML according to the schema
2. Validate all referenced taint kinds exist
3. Validate source patterns are valid globs
4. Validate sink blocked_taints reference defined taint kinds

## Validation Rules

**Taint Kinds**:

- Names must be unique
- Names must use snake_case
- Description is required

**Sources**:

- Must reference defined taint kinds
- Pattern is optional but must be valid glob if present

**Sinks**:

- Must reference defined taint kinds in blocked_taints
- Empty blocked_taints array is valid (permits all taints)

## Benefits

- **Declarative** - Policies separate from code
- **Portable** - Same format across implementations
- **Versionable** - Policies in version control
- **Auditable** - Clear documentation of security policies
- **Flexible** - Runtime policy updates without code changes

## Limitations

TSL Core v1.0 does NOT support:

- Hierarchical taint relationships
- Automatic taint propagation
- Pattern-based untainting
- Transform operations
- Sensitivity levels
- Conditional policies

These features are experimental and need to be documented.

## Related Specifications

- [Information Flow Tracking](definition.md) - Core taint tracking concepts
- [Taint Supervisor](taint-supervisor.md) - Centralized taint management pattern
- [JSON Schemas](../schemas/README.md) - Validation schemas for taint data

## References

- YAML 1.2 Specification - [yaml.org/spec/1.2/spec.html](https://yaml.org/spec/1.2/spec.html)
- JSON Schema Draft 2020-12 - [json-schema.org](https://json-schema.org/)
