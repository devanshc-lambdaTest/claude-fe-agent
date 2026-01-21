# Claude FE Agent

A generic, reusable Claude Code agent configuration for frontend developers. Works with any React, Vue, Next.js, or other frontend project out of the box.

## Features

- **Auto-Detection**: Automatically detects your project's framework, state management, styling approach, and component libraries
- **Project Memory**: Cache project context for instant loading on return visits (no more waiting for codebase scans)
- **Jira/Figma/GitHub Integration**: Built-in MCP server support for your daily tools
- **7-Phase Implementation Workflow**: Structured approach from requirements to deployment
- **Generic Skills**: Work in any frontend project without configuration

## Quick Start

### 1. Clone and Install

```bash
# Clone the repository
git clone https://github.com/YOUR_USERNAME/claude-fe-agent.git
cd claude-fe-agent

# Run the setup script (creates symlinks to ~/.claude/)
./setup.sh
```

### 2. One-Time MCP Authentication

MCP servers are pre-configured. You only need to authenticate **once** and tokens persist forever (auto-refresh):

```bash
# Start Claude Code
claude

# Authenticate MCP servers (ONE TIME ONLY)
/mcp

# You'll see:
# - atlassian: Click to authenticate with Jira/Confluence
# - figma: Click to authenticate with Figma
#
# After authenticating, tokens are stored in ~/.claude/.credentials.json
# and auto-refresh - you'll never need to authenticate again!
```

**What happens after authentication:**
- Tokens stored securely in `~/.claude/.credentials.json`
- Tokens auto-refresh before expiry
- Works across ALL projects (user-level config)
- Persists across machine restarts
- Syncs if you backup `~/.claude/.credentials.json`

### 3. Remember Your First Project

```bash
cd your-project
claude
/remember    # Caches project context for fast future sessions
```

## What's Included

### Skills

| Skill | Description |
|-------|-------------|
| `/remember` | Cache project context for instant loading |
| `/implement` | Full 7-phase implementation workflow |
| `/gather-requirements` | Fetch requirements from Jira/GitHub |
| `/explore-codebase` | Deep codebase analysis (memory-aware) |
| `/design-to-code` | Convert Figma designs to code |

### Agents

| Agent | Description |
|-------|-------------|
| `code-reviewer` | Reviews code in any FE project |

### Configuration Files

| File | Purpose |
|------|---------|
| `CLAUDE.md` | Personal context template |
| `settings.json` | Permissions and enabled plugins |
| `workflow-config.json` | Auto-detection and workflow config |

## Directory Structure

```
claude-fe-agent/
├── README.md                    # This file
├── setup.sh                     # Installation script
├── uninstall.sh                 # Removal script
├── .gitignore                   # Git ignore rules
│
├── CLAUDE.md                    # Personal context (customize this!)
├── settings.json                # Permissions and plugins
├── workflow-config.json         # Workflow configuration
│
├── skills/
│   ├── remember-project/        # Project memory caching
│   ├── gather-requirements/     # Requirements gathering
│   ├── explore-codebase/        # Codebase exploration
│   ├── design-to-code/          # Figma to code
│   └── implement/               # Master orchestrator
│
├── agents/
│   └── code-reviewer.md         # Code review agent
│
└── memory/
    ├── index.json               # Project index
    └── projects/                # Cached project data (gitignored)
```

## How It Works

### Auto-Detection

The agent automatically detects from `package.json`:

- **Framework**: React, Vue, Next.js, Nuxt, Angular, Svelte
- **State Management**: Redux, Zustand, MobX, Vuex, Pinia
- **Styling**: Tailwind, CSS Modules, Styled Components, SCSS
- **Testing**: Jest, Vitest, Cypress, Playwright
- **Component Libraries**: MUI, Chakra, Ant Design, Primer, Radix

### Project Memory

```
First visit:
  /remember → Deep scan (~2 min) → Cache saved

Return visit:
  Start session → Load cache (<1 sec) → Ready!
```

**What gets cached:**
- Tech stack and versions
- Directory structure
- Code patterns and conventions
- Architecture overview
- Key files and their purposes

### Implementation Workflow

```
Phase 1: Gather Requirements    ─┐
Phase 2: Explore Codebase        │
Phase 3: Decide Approach         │── All phases adapt to
Phase 4: Plan Implementation     │   your project's stack
Phase 5: Setup Tests (TDD)       │
Phase 6: Execute Implementation  │
Phase 7: Validate & Finalize    ─┘
```

## Customization

### Personal Context

Edit `CLAUDE.md` with:
- Your name and role
- Projects you work on
- Your conventions and preferences
- Jira configuration

### Workflow Config

Edit `workflow-config.json` to:
- Set your Jira Cloud ID
- Customize directory detection patterns
- Adjust validation rules

### Adding Skills

Create a new skill:
```bash
mkdir skills/my-skill
cat > skills/my-skill/skill.md << 'EOF'
---
name: my-skill
description: What this skill does
---

# My Custom Skill
...
EOF
```

### Adding Agents

Create a new agent:
```bash
cat > agents/my-agent.md << 'EOF'
---
name: my-agent
description: When to use this agent
tools: Read, Grep, Glob
model: sonnet
---

You are an expert at...
EOF
```

## Syncing Across Machines

Since this is a git repository:

```bash
# On your other machine
git clone https://github.com/YOUR_USERNAME/claude-fe-agent.git
cd claude-fe-agent
./setup.sh

# Keep in sync
git pull
```

## Sharing with Team

1. Fork or clone this repository
2. Customize for your team's conventions
3. Share the repository
4. Each team member runs `./setup.sh`

## MCP Authentication Details

### How It Works

```
First time setup:
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│  /mcp       │ ──▶ │  OAuth      │ ──▶ │  Tokens     │
│             │     │  (browser)  │     │  saved      │
└─────────────┘     └─────────────┘     └─────────────┘

Every subsequent session:
┌─────────────┐     ┌─────────────┐
│  Start      │ ──▶ │  Tokens     │ ──▶ MCP Ready!
│  claude     │     │  auto-load  │
└─────────────┘     └─────────────┘
```

### Token Storage

| File | Contents | Sensitive? |
|------|----------|------------|
| `~/.claude/.credentials.json` | OAuth tokens | Yes - never commit! |
| `~/.claude/settings.json` | MCP server URLs | No - safe to share |

### Pre-configured MCP Servers

| Server | URL | Purpose |
|--------|-----|---------|
| `atlassian` | `mcp.atlassian.com` | Jira, Confluence |
| `figma` | `mcp.figma.com` | Design specs |

### Syncing Auth Across Machines

To use the same authentication on another machine:

```bash
# On original machine - backup credentials
cp ~/.claude/.credentials.json ~/safe-backup/

# On new machine - after running setup.sh
cp ~/safe-backup/.credentials.json ~/.claude/
```

**Note:** Credentials file is gitignored and should NEVER be committed.

## Troubleshooting

### Skills Not Working

1. Verify symlinks: `ls -la ~/.claude/skills`
2. Check skill file: `cat ~/.claude/skills/remember-project/skill.md`
3. Re-run setup: `./setup.sh`

### MCP Not Connecting

1. Run `/mcp` to check status
2. Check if tokens exist: `ls ~/.claude/.credentials.json`
3. If expired, re-authenticate: `/mcp` → click server → OAuth
4. Verify network connectivity

### MCP Token Expired

Tokens auto-refresh, but if issues occur:
```bash
# Check token status
/mcp

# If server shows "Not authenticated", click to re-auth
# This is rare - tokens typically auto-refresh
```

### Memory Not Loading

1. Check index: `cat ~/.claude/memory/index.json`
2. Verify project was remembered: `/remember --list`
3. Refresh: `/remember --refresh`

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Submit a pull request

## License

MIT License - feel free to use, modify, and share.

---

**Made with Claude Code** - Your AI-powered development workflow
