# Claude FE Agent

A reusable Claude Code agent configuration for frontend developers. Works with any React, Vue, Next.js, or other frontend project out of the box.

## Features

- **Auto-Detection** — Detects framework, state management, styling, and component libraries from `package.json`
- **Project Memory** — Cache project context for instant loading on return visits
- **MCP Integrations** — Jira, Figma, and GitHub via built-in MCP servers
- **Bulk Package Updates** — Update shared npm packages across multiple repos with a single command, auto-creating PRs via fork workflow
- **7-Phase Workflow** — Structured implementation from requirements to deployment
- **Extensible Skills** — Add custom skills and agents for your team's workflows

## Setup

### Prerequisites

| Tool       | Purpose               | Install             |
| ---------- | --------------------- | ------------------- |
| Node.js    | Runtime               | `brew install node` |
| Go         | GitHub MCP server     | `brew install go`   |
| GitHub CLI | Git operations & auth | `brew install gh`   |

```bash
# Authenticate GitHub CLI
gh auth login && gh auth status
```

### Installation

```bash
git clone https://github.com/YOUR_USERNAME/claude-fe-agent.git
cd claude-fe-agent
./setup.sh    # Creates symlinks to ~/.claude/
```

### MCP Servers

**Atlassian & Figma** — Pre-configured in `settings.json`. Authenticate once via `/mcp` in Claude Code (OAuth, tokens auto-refresh).

**GitHub** — Uses the [official GitHub MCP server](https://github.com/github/github-mcp-server) binary locally:

```bash
# Install the binary
go install github.com/github/github-mcp-server/cmd/github-mcp-server@latest

# Add Go bin to PATH (add to ~/.zshrc)
export PATH="$HOME/go/bin:$PATH"

# Register with Claude Code
claude mcp add-json github \
  "{\"command\":\"$(go env GOPATH)/bin/github-mcp-server\",\"args\":[\"stdio\"],\"env\":{\"GITHUB_PERSONAL_ACCESS_TOKEN\":\"$(gh auth token)\"}}" \
  --scope user

# Verify
claude mcp list   # Should show: github ✓ Connected
```

> Stored in `~/.claude.json` (user-level, never committed to git).

### Verify Setup

```bash
claude mcp list
# ✓ atlassian - Connected
# ✓ figma     - Connected
# ✓ github    - Connected
```

## Skills

| Skill                   | Description                                  |
| ----------------------- | -------------------------------------------- |
| `/remember`             | Cache project context for instant loading    |
| `/implement`            | Full 7-phase implementation workflow         |
| `/gather-requirements`  | Fetch requirements from Jira/GitHub          |
| `/explore-codebase`     | Deep codebase analysis (memory-aware)        |
| `/design-to-code`       | Convert Figma designs to code                |
| `/bulk-update-packages` | Update npm packages across multiple FE repos |

### `/bulk-update-packages`

Automate npm package updates across multiple frontend repositories — no manual PRs needed.

```bash
# Update to specific version
/bulk-update-packages @lambdatestincprivate/lt-common-header 0.3.62

# Update to latest
/bulk-update-packages @lambdatestincprivate/lt-common-header latest

# Check current versions across all repos
/bulk-update-packages --status @lambdatestincprivate/lt-common-header

# Preview without making changes
/bulk-update-packages --dry-run @lambdatestincprivate/lt-common-header 0.3.62
```

**How it works:**

```
Parse input → Scan repos → Show version report → Confirm → Fork → Sync → Update → Create PRs
```

**Key features:**
- **Fork-only workflow** — Never commits directly to upstream, even with write access
- **No local clones** — All operations via GitHub API (`gh` CLI)
- **Smart validation** — Skips repos where package isn't installed or already at target version
- **Version prefix preservation** — Maintains `^`, `~`, or exact version format
- **Registry-based** — Only touches repos explicitly listed in the skill config

**Safety:** Forks repos → syncs with upstream → creates branch in fork → opens PR from fork to upstream. Changes always require PR review before merge.

## Agents

| Agent           | Description                    |
| --------------- | ------------------------------ |
| `code-reviewer` | Reviews code in any FE project |

## Project Structure

```
claude-fe-agent/
├── CLAUDE.md                    # Personal context (customize this)
├── settings.json                # Permissions and MCP config
├── workflow-config.json         # Auto-detection config
├── setup.sh / uninstall.sh      # Install/remove scripts
├── skills/
│   ├── remember-project/
│   ├── gather-requirements/
│   ├── explore-codebase/
│   ├── design-to-code/
│   ├── implement/
│   └── bulk-update-packages/
├── agents/
│   └── code-reviewer.md
└── memory/
    └── projects/                # Cached data (gitignored)
```

## Customization

### Personal Context (`CLAUDE.md`)

Add your name, role, projects, conventions, and Jira config.

### Workflow Config (`workflow-config.json`)

Set Jira Cloud ID, directory detection patterns, and validation rules.

### Adding a Skill

```bash
mkdir skills/my-skill
# Create skills/my-skill/skill.md with frontmatter (name, description, allowed-tools)
```

### Adding an Agent

```bash
# Create agents/my-agent.md with frontmatter (name, description, tools, model)
```

## Syncing & Sharing

```bash
# On another machine
git clone https://github.com/YOUR_USERNAME/claude-fe-agent.git
cd claude-fe-agent && ./setup.sh

# Team usage: fork → customize → each member runs ./setup.sh
```

## Troubleshooting

| Issue              | Fix                                                          |
| ------------------ | ------------------------------------------------------------ |
| Skills not working | `ls -la ~/.claude/skills` — re-run `./setup.sh` if broken    |
| MCP not connecting | Run `/mcp` in Claude Code — re-authenticate if needed        |
| GitHub MCP failing | `claude mcp list` — re-run the `claude mcp add-json` command |
| Memory not loading | `/remember --refresh` to re-scan                             |

## License

MIT
