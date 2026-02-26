---
title: "Skills Knowledge Comparison"
summary: "Comparative study of skills and knowledge systems between OpenClaw and Agent Zero, covering format, retrieval, and design principles."
read_when:
  - Comparing skill system designs between different agent frameworks
  - Understanding vector vs keyword retrieval approaches for skills
  - Evaluating load-time gating and injection models
status: active
last_updated: "2025-01-16"
---

# Skills & Knowledge Systems — OpenClaw vs Agent Zero

Comparative study focused on format, vector/semantic retrieval, and design principles.

---

## 1. Format comparison

### Shared baseline: SKILL.md + YAML frontmatter

Both systems use the [AgentSkills](https://agentskills.io) convention: a directory containing a `SKILL.md` with YAML frontmatter followed by Markdown instructions.

| Aspect                 | Agent Zero                                                                                       | OpenClaw                                                                                                                    |
| ---------------------- | ------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- |
| Required fields        | `name`, `description`                                                                            | `name`, `description`                                                                                                       |
| Optional fields        | `version`, `author`, `tags`, `triggers`, `allowed-tools`, `license`, `compatibility`, `metadata` | `homepage`, `user-invocable`, `disable-model-invocation`, `command-dispatch`, `command-tool`, `metadata` (single-line JSON) |
| Metadata format        | Nested YAML dict                                                                                 | **Single-line JSON object** (parser constraint)                                                                             |
| Gating / load filters  | None at load time                                                                                | `metadata.openclaw.requires.{bins, env, config}`, `os`, `always`                                                            |
| Installer hints        | None                                                                                             | `metadata.openclaw.install` (brew/node/go/uv/download)                                                                      |
| YAML parser            | PyYAML with fallback minimal parser                                                              | Single-line constraint (embedded agent limitation)                                                                          |
| Slash command dispatch | No                                                                                               | `command-dispatch: tool` bypasses model entirely                                                                            |

**Key difference**: OpenClaw gates skills at load time based on environment (binaries on PATH, env vars, config flags). Agent Zero loads all discoverable skills unconditionally — filtering happens only at search/retrieval time.

---

## 2. Skill locations and precedence

### Agent Zero

```
skills/                          # bundled defaults
usr/skills/                      # user global
usr/agents/<profile>/skills/     # user profile-scoped
agents/<profile>/skills/         # built-in profile-scoped
usr/projects/*/.a0proj/skills/   # project-level
usr/projects/*/.a0proj/agents/*/skills/  # project + profile
```

Resolution: first root in `get_skill_roots()` that contains a matching `SKILL.md` wins (deduplication by normalized name, earlier root wins).

### OpenClaw

```
bundled skills (npm/app)         # lowest
~/.openclaw/skills               # managed/local
<workspace>/skills               # highest
skills.load.extraDirs            # additional, lowest
```

Plugin skills participate in the same precedence. Per-agent workspace means skills can be truly isolated per agent in multi-agent setups.

**Key difference**: OpenClaw has a clean 3-tier model (bundled → user → workspace) with explicit plugin integration. Agent Zero has a 6-tier model mirroring the prompt resolution chain — consistent but more complex.

---

## 3. Vector / semantic retrieval

### Agent Zero — keyword scoring only

`search_skills()` in `agent_zero/helpers/skills.py` uses a pure keyword scorer:

```python
for term in terms:
    if term in name:   score += 3
    if term in desc:   score += 2
    if any(term in tag for tag in tags):  score += 1
```

- No embeddings, no vector index.
- Skills are injected into the agent context when the agent explicitly calls `skills_tool` or when the agent loop detects relevance via keyword match.
- The `description` field is the primary retrieval signal — it must be written to match the natural language queries the agent will issue.

### OpenClaw — compact XML injection + optional vector memory

OpenClaw injects a **compact XML list** of eligible skills into the system prompt on every session start:

```
total_chars = 195 + Σ (97 + len(name) + len(description) + len(location))
```

The model itself decides which skill to invoke based on this injected list. Skills are not retrieved on demand — they are always present (if eligible) as a structured list.

For **memory** (not skills directly), OpenClaw has a full vector pipeline:
- SQLite-vec for vector storage
- Hybrid BM25 + vector search (configurable weights)
- Multiple embedding providers (OpenAI, Gemini, Voyage, local GGUF via node-llama-cpp)
- QMD backend (BM25 + vectors + reranking via external sidecar)
- Session transcript indexing (experimental)
- Embedding cache to avoid re-embedding unchanged chunks

Skills themselves are not vector-indexed in OpenClaw — the compact XML list is small enough to fit in the system prompt. Vector search is reserved for the larger, unstructured memory corpus.

**Key difference**: Agent Zero uses on-demand keyword retrieval for skills. OpenClaw uses always-on XML injection for skills (bounded by eligibility gating) and reserves vector search for memory. Agent Zero has no vector layer at all currently.

---

## 4. Knowledge / documentation as context

### OpenClaw workspace templates

OpenClaw ships a set of Markdown templates that live in the agent workspace and are read at session start:

| File           | Purpose                                             |
| -------------- | --------------------------------------------------- |
| `SOUL.md`      | Agent identity, personality, behavioral constraints |
| `IDENTITY.md`  | Who the agent is in this deployment                 |
| `AGENTS.md`    | Multi-agent routing rules                           |
| `BOOT.md`      | Startup instructions                                |
| `BOOTSTRAP.md` | First-run setup                                     |
| `HEARTBEAT.md` | Periodic background task instructions               |
| `TOOLS.md`     | Tool usage guidance                                 |
| `USER.md`      | User preferences and context                        |

These are **always-loaded** structured documents, not retrieved on demand. They form a persistent identity layer separate from skills.

### Agent Zero

Agent Zero has no equivalent always-loaded knowledge layer. Knowledge lives in:
- The prompt system (`prompts/` — always loaded)
- Memory fragments (retrieved via vector search at query time)
- Skills (loaded on demand)

There is no `SOUL.md`-equivalent — agent identity is entirely in the prompt files.

---

## 5. Design principles and gaps

### What OpenClaw does well

1. **Load-time gating** — skills that can't run (missing binary, missing API key) are never injected. Keeps the system prompt clean and avoids confusing the model with unavailable tools.

2. **Bounded injection cost** — the compact XML format makes the per-skill token cost deterministic and predictable (~24 tokens/skill). Agent Zero's on-demand load injects the full SKILL.md body, which can be large.

3. **Hybrid memory search** — BM25 + vector covers both semantic similarity and exact token matching (IDs, code symbols, error strings). Pure vector search misses the latter.

4. **Identity layer** — `SOUL.md`, `IDENTITY.md`, `USER.md` give the agent a stable, inspectable identity that persists across sessions without being buried in prompt files.

5. **Plugin-shipped skills** — skills can be bundled with plugins, enabling clean packaging of capability + configuration + skill as a unit.

### What Agent Zero does well

1. **Profile-scoped skills** — skills can be scoped to an agent profile, enabling per-persona skill sets without separate workspaces.

2. **Project-scoped skills** — skills can be scoped to a project, enabling per-project capability sets.

3. **Richer frontmatter** — `triggers`, `allowed-tools`, `compatibility` fields give more control over when and how a skill activates.

4. **Consistent resolution chain** — skill roots mirror the prompt resolution chain exactly, making the mental model uniform across the system.

### Gaps in Agent Zero

| Gap                  | Impact                                                        | Suggested direction                                                 |
| -------------------- | ------------------------------------------------------------- | ------------------------------------------------------------------- |
| No load-time gating  | Skills for unavailable tools injected into context            | Add `requires` block to frontmatter; filter in `list_skills()`      |
| Keyword-only search  | Poor recall for paraphrase queries                            | Add embedding index over `name + description`; hybrid BM25 + vector |
| No bounded injection | Full SKILL.md body loaded on demand; token cost unpredictable | Separate "summary" (always injected) from "body" (loaded on demand) |
| No identity layer    | Agent identity buried in prompt files; not inspectable        | Add `IDENTITY.md` / `SOUL.md` equivalent in workspace               |
| No skill registry    | No equivalent to ClawHub                                      | Out of scope for now, but worth noting                              |

---

## 6. Recommended design for Agent Zero skill improvement

### Principle 1: Separate discovery metadata from execution content

The frontmatter (`name`, `description`, `tags`) is the discovery surface. The body is the execution surface. They should be loaded independently:

- **Discovery pass**: load frontmatter only → build index → inject compact list into system prompt
- **Execution pass**: load full body only when the agent selects a skill

This mirrors OpenClaw's compact XML injection + on-demand full load.

### Principle 2: Add load-time gating

Extend frontmatter with a `requires` block:

```yaml
requires:
  env: [OPENAI_API_KEY]
  bins: [ffmpeg]
  config: [browser.enabled]
```

Filter in `list_skills()` before building the discovery index. Skills that can't run should never appear in the agent's context.

### Principle 3: Add a vector index over descriptions

Index `name + description + tags` as a single embedding per skill. At query time, combine vector similarity with the existing keyword scorer (hybrid). This improves recall for paraphrase queries without replacing the fast keyword path.

Storage: a small SQLite file per agent (mirroring OpenClaw's per-agent SQLite). Invalidate on SKILL.md mtime change.

### Principle 4: Bounded system prompt injection

Replace full-body injection with a compact summary list (name + description + path), injected once at session start for all eligible skills. Full body loaded only when the agent explicitly requests a skill via `skills_tool`.

Token budget: ~30 tokens/skill × 50 skills = ~1500 tokens — acceptable overhead.

### Principle 5: Consider an identity layer

OpenClaw's `SOUL.md` / `IDENTITY.md` / `USER.md` pattern is worth adopting. A `WORKSPACE.md` or `IDENTITY.md` in `usr/` that is always loaded alongside the prompt system would give agents a stable, user-editable identity layer without touching prompt files.

---

## 7. Summary table

| Dimension             | Agent Zero          | OpenClaw                                    | Gap                  |
| --------------------- | ------------------- | ------------------------------------------- | -------------------- |
| Skill format          | SKILL.md + YAML     | SKILL.md + YAML (single-line JSON metadata) | Minor                |
| Load-time gating      | ❌                   | ✅ (bins, env, config, os)                   | Medium               |
| Retrieval             | Keyword scorer      | Compact XML injection (always-on)           | Medium               |
| Vector search         | ❌                   | ✅ for memory (hybrid BM25+vec)              | High                 |
| Injection model       | On-demand full body | Compact list always + full body on demand   | Medium               |
| Identity layer        | ❌                   | ✅ (SOUL/IDENTITY/USER/AGENTS.md)            | Low–Medium           |
| Skill registry        | ❌                   | ✅ (ClawHub)                                 | Out of scope         |
| Profile-scoped skills | ✅                   | ❌ (workspace-per-agent instead)             | Agent Zero advantage |
| Project-scoped skills | ✅                   | ❌                                           | Agent Zero advantage |
