---
title: "Hub Configuration"
summary: "Unified configuration format for managing both skill and documentation hub sources in agent systems"
read_when:
  - Implementing hub source management in agent systems
  - Designing agent configuration files
  - Understanding hub discovery and caching mechanisms
  - Configuring multiple skill and doc hubs
status: active
last_updated: "2025-01-16"
---

# Hub Configuration

## 1. Purpose

Agents configure multiple hub sources for both skills and documentation using a unified configuration format. This specification defines how to manage skill hubs and doc hubs in a single configuration file with shared structure and independent control.

---

## 2. Configuration Schema

**Schema**: [`schemas/agentctl-config.json`](../schemas/agentctl-config.json)

**Structure**:

```json
{
  "skills_root": "~/.agent/skills",
  "skill_hubs": [
    {
      "id": "official",
      "index_url": "https://skills.example.com/index.json",
      "git_url": "https://github.com/org/skills",
      "enabled": true,
      "ttl_hours": 6
    }
  ],
  "doc_hubs": [
    {
      "id": "team-docs",
      "index_url": "https://docs.example.com/index.json",
      "git_url": "https://github.com/org/docs",
      "enabled": true,
      "ttl_hours": 24
    }
  ]
}
```

---

## 3. Hub Configuration Fields

### Required Fields

**`id`**: Unique hub identifier

- Pattern: `^[a-z0-9-]+$`
- Used in lock file keys (`hub_id:slug`)
- Must be unique across all configured hubs

**`index_url`**: URL to hub's index.json

- Must be accessible via HTTPS
- Points to `index.json`, `skills.json`, or `docs.json`
- Used for hub discovery and search

### Optional Fields

**`skills_root`**: Root directory for installed skills

- Default: `~/.agent/skills`
- Supports `~` expansion
- Skills installed to `<skills_root>/<name>/` or `<skills_root>-<mode>/<name>/`
- Recorded in lock file `installed_path` — AI clients read this to locate skills

**`git_url`**: Git repository URL

- Used for sparse cloning during skill installation
- If omitted, skills cannot be installed (index-only mode)

**`enabled`**: Hub activation status

- Default: `true`
- Disabled hubs excluded from search and install operations
- Allows temporary hub deactivation without removal

**`ttl_hours`**: Cache time-to-live in hours

- Default: `6`
- Minimum: `1`
- Controls how long cached `index.json` is considered fresh

---

## 4. Hub Types

**Auto-detection**: Hub type determined by `index.json` `type` field

- `"type": "skills"` → Skill hub
- `"type": "docs"` → Documentation hub

**Separation**: Skills and docs use separate configuration arrays

- Allows different default TTLs
- Enables type-specific operations

---

## 5. Index Caching

**Cache location**: Implementation-defined (e.g., `~/.agentctl/cache/hubs/<hub_id>/`)

**Cache behavior**:

1. On first access: Fetch `index_url`, cache with timestamp
2. On subsequent access: Check age against `ttl_hours`
3. If expired: Refetch and update cache
4. If network unavailable: Use stale cache with warning

**Manual refresh**: Implementations should support forced cache refresh

---

## 6. Hub Operations

### Add Hub

Add new hub to configuration with validation:

- `id` must be unique
- `index_url` must be accessible
- `index.json` must have valid `type` field

### Remove Hub

Remove hub from configuration:

- Clear cached index
- Optionally remove installed skills from this hub
- Update lock file to remove orphaned entries

### Enable/Disable

Toggle `enabled` field without removing configuration:

- Disabled hubs excluded from operations
- Preserves configuration for re-enablement

---

## 7. Multi-Hub Behavior

- **Search**: Query all enabled hubs, merge results
- **Install**: Specify hub via `hub_id:slug` identifier
- **Update**: Check all enabled hubs for newer versions
- **Conflict**: Same `hub_id:slug` can only be installed once

---

## 8. Security Considerations

**Trust model**: User trusts hub operator's CI validation

- `index_url` should use HTTPS
- `git_url` should use HTTPS or SSH with verification

**Isolation**: Hubs are independent

- One compromised hub doesn't affect others
- User controls which hubs are enabled

---

## 9. Design Principles

- **Explicit configuration**: No automatic hub discovery
- **User control**: All hub operations require user action
- **Graceful degradation**: Stale cache usable when network unavailable
- **Type separation**: Skills and docs managed independently

---

## Related Specifications

- [Skill Hub](../skills/hub.md) — Skill hub index format
- [Doc Hub](../docs/hub.md) — Documentation hub index format
- [Skill Lifecycle](../skills/lifecycle.md) — Skill installation and versioning
- [Skills Lock Schema](../schemas/skills-lock.json) — Installed skills tracking
