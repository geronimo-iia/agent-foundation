---
title: "Lifecycle YAML Format"
summary: "Complete specification for lifecycle.yaml format - agent-assisted skill dependency management with install, update, and uninstall commands"
read_when:
  - Writing lifecycle.yaml for a skill
  - Implementing lifecycle command execution in an agent
  - Understanding skill dependency management
status: active
last_updated: "2025-01-16"
---

# Lifecycle YAML Format

## Purpose

The `lifecycle.yaml` file enables agent-assisted dependency management for skills. It defines install, update, and uninstall commands that the agent executes with user approval.

## File Location

```
skill-name/
├── SKILL.md
├── lifecycle.yaml    # Optional
├── scripts/
└── references/
```

## Schema

```yaml
variables:
  # Custom variables with default values
  VAR_NAME: value

install:
  - command: shell command
    description: Human-readable explanation
    platform: all | linux | macos | windows
    requires_approval: true | false

update:
  - command: shell command
    description: Human-readable explanation
    platform: all | linux | macos | windows
    requires_approval: true | false

uninstall:
  - command: shell command
    description: Human-readable explanation
    platform: all | linux | macos | windows
    requires_approval: true | false
```

## Built-in Variables

All commands have access to these built-in variables:

| Variable | Description | Example |
|----------|-------------|---------|
| `${SKILL_NAME}` | Skill name from SKILL.md frontmatter | `marker-pdf` |
| `${SKILL_PATH}` | Absolute path to skill installation directory | `~/.local/share/skills/marker-pdf` or `~/.agent/skills/marker-pdf` |
| `${HOME}` | User home directory | `/home/user` |
| `${PLATFORM}` | Current platform | `linux`, `macos`, or `windows` |

**Note**: `SKILL_PATH` is set by the agent based on where the skill is installed (global or project-specific). Skills should not hardcode installation paths.

## Custom Variables

Define custom variables in the `variables` section:

```yaml
variables:
  VENV_PATH: ${SKILL_PATH}/.venv
  PYTHON_BIN: ${VENV_PATH}/bin/python
  CONFIG_FILE: ${SKILL_PATH}/.config
  CACHE_DIR: ${SKILL_PATH}/cache
```

**Evaluation order**:
1. Built-in variables initialized first
2. Custom variables evaluated in declaration order
3. Variables can reference previously defined variables
4. Forward references not allowed

**Example evaluation**:

```yaml
variables:
  VENV_PATH: ${SKILL_PATH}/.venv          # Uses built-in SKILL_PATH
  PYTHON_BIN: ${VENV_PATH}/bin/python     # Uses VENV_PATH defined above
```

## Command Fields

### command

Shell command to execute. Can be single-line or multi-line:

```yaml
# Single-line
command: uv venv ${VENV_PATH}

# Multi-line
command: |
  cat > ${CONFIG_FILE} << 'EOF'
  export CACHE_DIR="${SKILL_PATH}/cache"
  EOF
```

### description

Human-readable explanation shown to user before execution:

```yaml
description: Create virtual environment and install dependencies
```

### platform

Target platform(s) for this command:

- `all` - Run on all platforms (default)
- `linux` - Linux only
- `macos` - macOS only
- `windows` - Windows only

```yaml
platform: macos
```

### requires_approval

Whether user must approve this command:

- `true` - User must approve (default for install/update/uninstall)
- `false` - Execute without approval (use for safe operations like creating config files)

```yaml
requires_approval: true
```

## Command Sections

### install

Commands executed when skill is first installed:

```yaml
install:
  - command: uv venv ${VENV_PATH}
    description: Create virtual environment
    platform: all
    requires_approval: true
  
  - command: uv pip install --python ${PYTHON_BIN} package-name
    description: Install Python package
    platform: all
    requires_approval: true
```

### update

Commands executed when skill is updated to a new version:

```yaml
update:
  - command: uv pip install --python ${PYTHON_BIN} --upgrade package-name
    description: Update Python package to latest version
    platform: all
    requires_approval: true
```

### uninstall

Commands executed when skill is removed:

```yaml
uninstall:
  - command: rm -rf ${VENV_PATH}
    description: Remove virtual environment
    platform: all
    requires_approval: true
  
  - command: rm -rf ${CACHE_DIR}
    description: Remove cached data
    platform: all
    requires_approval: true
```

## Common Patterns

### Python Virtual Environment

```yaml
variables:
  VENV_PATH: ${SKILL_PATH}/.venv
  PYTHON_BIN: ${VENV_PATH}/bin/python

install:
  - command: uv venv ${VENV_PATH} && uv pip install --python ${PYTHON_BIN} package-name
    description: Create venv and install package
    platform: all
    requires_approval: true

uninstall:
  - command: rm -rf ${VENV_PATH}
    description: Remove virtual environment
    platform: all
    requires_approval: true
```

### Configuration File

```yaml
variables:
  CONFIG_FILE: ${SKILL_PATH}/.config

install:
  - command: |
      cat > ${CONFIG_FILE} << 'EOF'
      # Skill configuration
      export CACHE_DIR="${SKILL_PATH}/cache"
      EOF
    description: Create default configuration
    platform: all
    requires_approval: false
```

### Platform-Specific Commands

```yaml
install:
  - command: brew install ffmpeg
    description: Install ffmpeg via Homebrew
    platform: macos
    requires_approval: true
  
  - command: apt-get install -y ffmpeg
    description: Install ffmpeg via apt
    platform: linux
    requires_approval: true
```

### Cache Management

```yaml
variables:
  CACHE_DIR: ${SKILL_PATH}/cache

uninstall:
  - command: rm -rf ${CACHE_DIR}
    description: Remove cached data
    platform: all
    requires_approval: true
  
  - command: rm -rf ~/Library/Caches/skill-name
    description: Remove system cache (macOS)
    platform: macos
    requires_approval: true
  
  - command: rm -rf ~/.cache/skill-name
    description: Remove system cache (Linux)
    platform: linux
    requires_approval: true
```

## Security

### User Approval

- All commands visible in lifecycle.yaml (auditable)
- User must approve each command individually
- Agent explains what each command does
- No hidden or obfuscated commands

### Safe Operations

Commands that don't require approval (use `requires_approval: false`):

- Creating configuration files
- Creating directories
- Setting environment variables in config files

Commands that require approval (use `requires_approval: true`):

- Installing packages
- Executing downloaded code
- Modifying system files
- Network operations
- Deleting files

## Validation Rules

1. **Required fields**: `command`, `description`, `platform`, `requires_approval`
2. **Valid platforms**: `all`, `linux`, `macos`, `windows`
3. **Variable references**: Must use `${VAR_NAME}` syntax
4. **No circular references**: Variables cannot reference themselves
5. **Forward references**: Variables must be defined before use

## Complete Example

```yaml
variables:
  VENV_PATH: ${SKILL_PATH}/.venv
  PYTHON_BIN: ${VENV_PATH}/bin/python
  CONFIG_FILE: ${SKILL_PATH}/.config
  CACHE_DIR: ${SKILL_PATH}/cache

install:
  - command: uv venv ${VENV_PATH} && uv pip install --python ${PYTHON_BIN} marker-pdf[full] pypdf
    description: Create virtual environment and install marker-pdf with dependencies
    platform: all
    requires_approval: true
  
  - command: |
      cat > ${CONFIG_FILE} << 'EOF'
      # Marker PDF Configuration
      export MARKER_CACHE_DIR="${SKILL_PATH}/cache"
      export TORCH_HOME="${SKILL_PATH}/torch-cache"
      EOF
    description: Create default configuration file
    platform: all
    requires_approval: false

update:
  - command: uv pip install --python ${PYTHON_BIN} --upgrade marker-pdf
    description: Update marker-pdf to latest version
    platform: all
    requires_approval: true

uninstall:
  - command: rm -rf ${VENV_PATH}
    description: Remove virtual environment
    platform: all
    requires_approval: true
  
  - command: rm -rf ${CACHE_DIR}
    description: Remove cached data
    platform: all
    requires_approval: true
  
  - command: rm -rf ~/Library/Caches/datalab
    description: Remove system cache (macOS)
    platform: macos
    requires_approval: true
  
  - command: rm -rf ~/.cache/datalab
    description: Remove system cache (Linux)
    platform: linux
    requires_approval: true
```

## Related Specifications

- [lifecycle.md](lifecycle.md) - Skill lifecycle states and transitions
- [authoring-guide.md](authoring-guide.md) - How to write skills
- [definition.md](definition.md) - What skills are
