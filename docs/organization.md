---
title: "Specification Organization"
summary: "Organizational principles for agent foundation specifications including domain grouping, naming conventions, and cross-reference standards."
read_when:
  - Deciding where to place a new specification document
  - Establishing naming conventions for specification files
  - Setting up cross-reference standards between specifications
status: active
last_updated: "2025-01-16"
---

# Specification Organization Standard

---

## 1. Goal

Provide a standard structure for organizing agent foundation specifications by topic.
Grouping by domain makes the corpus navigable: a new specification has an obvious home,
and a reader looking for "agent coordination specs" has one place to look.

---

## 2. Grouping principles

- **One folder per coherent domain** — a domain is a set of specifications that implementers would naturally look for together
- **Specification type follows domain** — schemas, protocols, and guides for the same domain belong together
- **Cross-cutting specifications get their own folder** — choose folder name based on the content goal, not a fixed "foundation" name
- **Every domain requires a definition** — each domain folder must contain a `definition.md` that defines what the domain covers

---

## 3. Domain placement workflow

When creating a new specification, follow this decision process:

1. **Does it fit an existing domain?** — Check if the specification naturally belongs with existing domain folders
2. **Is it cross-cutting?** — If it spans multiple domains, create or use an appropriately named folder based on the content goal
3. **Does it define a new domain?** — If it doesn't fit existing domains and isn't cross-cutting:
  
   - Create a new domain folder
   - Write `definition.md` first to define the domain scope, see [authoring-guide](authoring-guide.md)
   - Then add the specification
4. **When in doubt** — Start at root level and move to a domain folder when the domain becomes clear

---

## 4. Naming conventions

Filenames use kebab-case slugs with specification type suffix:

```
skills/definition.md               ✓
skills/format-specification.md     ✓
skills/hub-protocol.md            ✓
agents/coordination-protocol.md   ✓
organization/structure-standard.md ✓
```

*Note: For frontmatter requirements and document formatting standards, see [authoring-guide](authoring-guide.md).*

---

## 5. Cross-references

Cross-reference fields use slugs, not paths. Prose references use relative paths:

```markdown
See [hub-protocol](skills/hub-protocol.md) for the distribution contract.
```
