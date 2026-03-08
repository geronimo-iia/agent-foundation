---
title: "Agent Workspace Setup"
summary: "Minimal setup guide for bootstrapping agent workspace with core repositories, rule templates, and IDE integration."
read_when:
  - Setting up the agent workspace for the first time
  - Bootstrapping with core repositories and rule templates
  - Getting minimal working setup with Amazon Q integration
  - Initial workspace configuration from scratch
status: active
last_updated: "2026-07-15"
---

# Agent Workspace Setup

## Overview

This guide provides minimal steps for bootstrapping the agent development workspace quickly.

## Prerequisites

- Git installed and configured
- Code editor with Amazon Q plugin (VS Code, Zed, etc.)
- Terminal/command line access
- `agentctl` installed (see below)

## Install agentctl

```bash
# macOS/Linux
brew tap geronimo-iia/agent && brew install agentctl

# or via cargo
cargo install agent-ctl
```

Verify: `agentctl --version`

## Installation Process

### Step 1: Create Workspace Directory

```bash
# Create workspace directory
mkdir agent-workspace
cd agent-workspace
```

### Step 2: Clone Agent Repositories

```bash
# Clone core agent repositories
git clone https://github.com/geronimo-iia/agent-foundation.git agent-foundation
git clone https://github.com/geronimo-iia/agent-skills.git agent-skills
git clone https://github.com/geronimo-iia/agent-software.git agent-software
```

### Step 3: Configure IDE Integration

#### Amazon Q Plugin
1. Install Amazon Q plugin in your IDE
2. Open the agent-workspace directory
3. Copy rule templates from `agent-foundation/rules/templates/` to `.amazonq/rules/`
4. Customize the templates for your workspace structure
5. The `.amazonq/rules/` directory will automatically apply workspace standards

### Step 4: Ready to Use

The workspace is now ready for development:
- **agent-foundation/** - Reference stable specifications
- **agent-skills/** - Available skills for agent use
- **agent-software/** - Software development standards and engineering practices
- **agentctl** - Validate and generate hub indexes
- **Workspace rules** - Automatically enforce standards through Amazon Q

For detailed usage, explore the specifications in each repository.