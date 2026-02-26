---
title: "Agent Workspace Setup"
summary: "Minimal setup guide for bootstrapping agent workspace with core repositories, rule templates, and IDE integration."
read_when:
  - Setting up the agent workspace for the first time
  - Bootstrapping with core repositories and rule templates
  - Getting minimal working setup with Amazon Q integration
  - Initial workspace configuration from scratch
status: active
last_updated: "2025-01-16"
---

# Agent Workspace Setup

## Overview

This guide provides minimal steps for bootstrapping the agent development workspace quickly.

## Prerequisites

- Git installed and configured
- Code editor with Amazon Q plugin (VS Code, Zed, etc.)
- Terminal/command line access

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
git clone <agent-foundation-url> agent-foundation
git clone <agent-software-url> agent-software
git clone <agent-software-engineering-url> agent-software-engineering
```

### Step 3: Configure IDE Integration

#### Amazon Q Plugin
1. Install Amazon Q plugin in your IDE
2. Open the agent-workspace directory
3. Copy rule templates from `agent-software-engineering/rules/templates/` to `.amazonq/rules/`
4. Customize the templates for your workspace structure
5. The `.amazonq/rules/` directory will automatically apply workspace standards

### Step 4: Ready to Use

The workspace is now ready for development:
- **agent-foundation/** - Reference stable specifications
- **agent-software/** - Use development standards and tooling
- **agent-software-engineering/** - Apply engineering practices
- **Workspace rules** - Automatically enforce standards through Amazon Q

For detailed usage, explore the specifications in each repository.