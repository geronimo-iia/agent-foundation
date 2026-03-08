---
title: "Documentation Authoring"
summary: "How to author documentation files with machine-readable YAML frontmatter for Agent Zero."
read_when:
  - Writing or reviewing a documentation file
  - Adding frontmatter to an existing doc
  - Designing the doc discovery or context-loading system
  - Understanding cross-reference fields between artifact types
status: active
last_updated: "2025-01-16"
---

# How to Write Docs — Machine-Readable YAML Frontmatter

Doc frontmatter is consumed by the **documentation system and agent context loader** —
not the skill runtime. It answers: what is this doc, and when should it be loaded?

This convention is derived from OpenClaw's doc corpus (200+ files using `summary` + `read_when`
consistently) but is not formally documented there. This guide is the canonical reference
for Agent Zero.

---

## 1. Schema

> **Scope**: Frontmatter is required on all `.md` files that are part of the doc corpus — i.e., indexed by `agentctl` and loaded by the agent. The following files are excluded and must **not** have frontmatter: `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, `ARCHIVED.md`.

Core fields (all docs):

```yaml
---
title: "Short display title"
summary: "One-line description of what this doc covers."
read_when:
  - Condition under which this doc is relevant
  - Another specific trigger condition
---
```

Extended fields (plan/experiment docs only):

```yaml
---
title: "My Refactor Plan"
summary: "Plan to isolate X from Y with end-to-end deadlines."
read_when:
  - Reviewing the X refactor approach
status: draft          # draft | active | done
owner: my-team
last_updated: "2025-07-01"
---
```

### Field reference

| Field          | Required | Context           | Purpose                                      |
| -------------- | -------- | ----------------- | -------------------------------------------- |
| `title`        | Yes      | All docs          | Display name for nav and headings            |
| `summary`      | Yes      | All docs          | One-line scope description for discovery     |
| `read_when`    | Yes      | All docs          | Load conditions — when is this doc relevant? |
| `status`       | No       | Plans/experiments | `draft`, `active`, or `done`                 |
| `owner`        | No       | Plans/experiments | Team or person responsible                   |
| `last_updated`    | No       | Plans/experiments | ISO date of last meaningful update                    |
| `requires_skills`  | No       | Docs tied to skills   | Skill slugs that must be installed for this doc to be actionable |
| `superseded_by`    | No       | Reference docs        | Slug of the doc that replaces this one |
| `source_study`     | No       | Plan/architecture docs | Research study slug that informed this doc |
| `documents_skill`  | No       | Reference docs        | Skill slug this doc describes |
| `produced_by`      | No       | Reference docs        | Task slug that produced this doc |

> `description` appears in ~23 OpenClaw docs as an inconsistent duplicate of `summary`.
> Do not use it — use `summary` only.

---

## 2. The `summary` field

One sentence. Describes the doc's scope, not its title.

```yaml
# Bad — just restates the title
summary: "Skills authoring guide."

# Good — describes scope and audience signal
summary: "How to author SKILL.md files with machine-readable YAML frontmatter for Agent Zero."
```

---

## 3. The `read_when` field

Each entry is a **specific condition**, not a topic label. Write it as a sentence fragment
starting with a verb or noun that completes "Read this when...":

```yaml
# Bad — too vague, no actionable signal
read_when:
  - Skills
  - Development

# Good — specific, actionable conditions
read_when:
  - Writing or reviewing a SKILL.md file
  - Implementing load-time gating or keyword scoring
  - Debugging skill discovery or injection
```

Aim for 2–4 entries. More than 4 usually means the doc covers too many topics.

---

## 4. The `requires_skills` field

Use `requires_skills` when the doc is only actionable if specific skills are installed. The
agent can use this to flag missing skills or suggest installing them alongside loading the doc.

```yaml
requires_skills:
  - payment-api       # skill slug, not display name
  - pdf-processing
```

Two cases where this applies:

- **The doc describes how to use a skill** — a "Payment API guide" that walks through using
  the `payment-api` skill. Without the skill, the doc is informational only.
- **The doc references scripts or tools bundled in a skill** — loading the doc without the
  skill means the referenced commands won't work.

Do not use `requires_skills` for docs that merely *mention* a skill in passing. Only use it
when the skill's absence makes the doc non-actionable.

---

## 5. Complete example

```yaml
---
title: "WebSocket Manager"
summary: "Internal WebSocket session lifecycle, debug flags, and namespace routing."
read_when:
  - Debugging WebSocket connection or session issues
  - Changing namespace routing or broadcast logic
  - Reviewing A0_WS_DEBUG flag behavior
---
```

With skill dependency:

```yaml
---
title: "Payment API Guide"
summary: "How to authenticate, call endpoints, and handle errors for the Payment API."
read_when:
  - Implementing or debugging payment API calls
  - Reviewing payment error handling
requires_skills:
  - payment-api
---
```

---

## 6. Authoring checklist

- [ ] File is not `README.md`, `CHANGELOG.md`, `CONTRIBUTING.md`, or `ARCHIVED.md` (those must not have frontmatter)
- [ ] `title` is present and matches the H1 heading
- [ ] `summary` is one sentence describing scope, not restating the title
- [ ] `read_when` entries are specific, actionable conditions (2–4 entries)
- [ ] Plan/experiment docs include `status`, `owner`, `last_updated`
- [ ] `requires_skills` lists only skills whose absence makes the doc non-actionable
- [ ] No `description` field — use `summary` instead
