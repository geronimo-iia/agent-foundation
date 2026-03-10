---
title: "What are Documents?"
summary: "Documents are markdown files with machine-readable frontmatter that provide knowledge to agents on demand."
read_when:
  - Understanding what documents are and how they work
  - Learning about the document format and structure
  - Implementing document support in an agent system
status: active
last_updated: "2025-01-16"
---

# What are Documents?

A **document** is a `.md` file with machine-readable YAML frontmatter that provides knowledge to agents on demand. Documents are loaded into agent context when relevant, unlike skills which are injected into system prompts.

## Document structure

```
my-document.md
```

Every document contains:

- **YAML frontmatter** — Metadata for discovery and loading
- **Markdown body** — The actual knowledge or specifications
- **Cross-references** — Links to related documents, skills, or tasks

## How documents work

Documents use **progressive disclosure** for knowledge management:

1. **Discovery**: Agents scan document frontmatter (`title`, `summary`, `read_when`) to understand what knowledge is available.

2. **Relevance matching**: When a task matches a document's `read_when` conditions, the agent identifies it as relevant.

3. **Loading**: The agent loads the full document content into context when needed, providing just-in-time knowledge.

This approach keeps agents efficient while giving them access to extensive knowledge on demand.

## The frontmatter format

Every document starts with YAML frontmatter containing discovery metadata:

```yaml
---
title: "Payment API Reference"
summary: "Endpoints, authentication, error codes, and rate limits for the Payment API."
read_when:
  - Implementing or debugging payment API calls
  - Reviewing error handling for payment flows
status: active
last_updated: "2025-01-16"
---

# Payment API Reference

The Payment API provides endpoints for...
```

Required frontmatter fields:

- `title`: Display name for the document
- `summary`: One-line description of what the document covers
- `read_when`: Specific conditions when this document is relevant

The Markdown body contains the actual knowledge and has no specific restrictions on structure or content.

This format has key advantages:

- **Discoverable**: Agents can quickly scan available knowledge without loading full content
- **Contextual**: `read_when` conditions make knowledge loading precise and relevant
- **Portable**: Documents are just markdown files, easy to edit, version, and share

## Related specifications

- [authoring-guide.md](authoring-guide.md) — Detailed authoring standards and best practices
- [hub.md](hub.md) — Multi-source documentation registry and distribution system
- [organization.md](organization.md) — Specification organization principles