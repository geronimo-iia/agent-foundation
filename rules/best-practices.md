---
title: "Rule Creation Best Practices"
summary: "Software engineering principles and patterns for creating maintainable workspace rules."
read_when:
  - Creating new workspace rules
  - Refactoring existing rule architecture
  - Designing rule systems for AI behavior enforcement
  - Applying software engineering principles to rule design
status: active
last_updated: "2025-01-16"
---

# Rule Creation Best Practices

## Core Principles

### Single Responsibility Principle (SRP)

- One rule = one concern
- Each rule has a single reason to change
- Avoid mixing unrelated requirements

### DRY (Don't Repeat Yourself)

- Reference specifications instead of duplicating content
- Maintain single source of truth for each requirement
- Rules enforce, specifications educate

### Separation of Concerns

- **Rules**: Enforce behavior and point to specifications
- **Specifications**: Contain detailed knowledge and examples
- **Prompts**: Provide task-specific assistance

## Rule Structure Pattern

```markdown
# Rule Name

## Requirement
Brief statement of what must be done.

## Reference
Link to detailed specification.

## Enforcement
Why this rule exists/consequences.
```

## Best Practices

### 1. Enforcement-Focused Language
- Use imperative: "MUST", "always use", "required"
- Make expectations clear and non-negotiable
- Focus on behavior, not explanation

### 2. Minimal Content
- Keep rules as short as possible while being clear
- Remove examples if they duplicate main specifications
- Prioritize navigation over education

### 3. Reference Architecture
- Point to authoritative specifications
- Use relative paths when possible
- Avoid embedding detailed content

### 4. Maintenance-Friendly Design
- Avoid version-specific details
- Design for easy updates when specs change
- Clear scope definition

## Anti-Patterns

- ❌ **Long explanatory content** (belongs in specifications)
- ❌ **Duplicated examples** (creates sync issues)
- ❌ **Vague requirements** ("should consider")
- ❌ **Multiple unrelated rules** in one file
- ❌ **Embedded specifications** (violates DRY)

## Rule vs Prompt Decision Matrix

| Use Rules When                        | Use Prompts When                |
| ------------------------------------- | ------------------------------- |
| Behavior must be enforced             | Task-specific assistance needed |
| Applies to all workspace interactions | User chooses when to apply      |
| Standards and constraints             | Templates and examples          |
| Automatic compliance required         | Optional guidance desired       |

## Software Engineering Parallels

Rules are essentially **configuration as code** for AI behavior:

- **Interface Design**: Clean API for rule consumption
- **Maintainability**: Low coupling, high cohesion
- **Configuration Management**: Version-controlled standards
- **Documentation Architecture**: Hierarchical information design