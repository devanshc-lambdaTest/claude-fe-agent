# My Development Context

## About Me
Full-stack developer with frontend focus. Working across multiple React/Vue projects.

## My Daily Tools
- **Jira** - Ticket management (LambdaTest organization)
- **Figma** - Design specs and mockups
- **GitHub** - Code hosting, PRs, issues

## My Workflow

### Typical Feature Implementation
1. Pick up Jira ticket (TTN-XXXXX, FORCE-XXXX, ASE-XX formats)
2. Review Figma design if applicable
3. Implement following project conventions
4. Create PR, get review, merge

### Code Review Preferences
- Check for performance issues (unnecessary re-renders, missing memoization)
- Verify proper error handling
- Ensure accessibility compliance
- Follow existing patterns in the codebase

## Projects I Work On

### LambdaTest Projects
- **mobile-web-client** - React + Redux + CRACO
- **lt-components** - Component library (Primer-based)
- **accessibility-testing-chrome-extension** - React extension

### Common Tech Stacks I Use
- React 18+ with hooks
- Redux + Redux Saga OR Zustand
- TypeScript (when available)
- Jest/Vitest for testing
- Tailwind OR CSS Modules OR SCSS

## My Conventions

### Git
- Branch naming: `TTN-XXXXX-short-description` or `fix/TTN-XXXXX`
- Commit style: Conventional commits preferred
- PR target: Usually `stage` or `develop`, not `main` directly

### Code Style
- Functional components with hooks
- Custom hooks for shared logic
- Container/Component pattern when using Redux
- Explicit error handling

## Jira Configuration

**Cloud ID:** `3def4f78-101d-4614-9b65-735c17a98a93`
**Site:** lambdatest.atlassian.net

**Ticket Formats:**
- `TTN-XXXXX` - Main project tickets
- `FORCE-XXXX` - Force tickets
- `ASE-XX` - ASE tickets

## Figma Access
Authenticated via Figma MCP plugin. Design links usually in Jira tickets.

## Preferences

### When Implementing
- Always check for existing components first
- Follow the project's established patterns
- Write tests when test framework is set up
- Keep PRs focused and reviewable

### When Reviewing
- Be constructive
- Focus on correctness, performance, maintainability
- Suggest improvements, don't just criticize

## Project Memory

I use `/remember` to cache project context for faster switching between repos.

**How it works:**
- First visit: `/remember` scans and caches project structure (~2 min)
- Return visits: Context loads instantly from cache (~1 sec)
- Speeds up `/implement`, `/explore-codebase`, and other skills

**Memory location:** `~/.claude/memory/projects/`

**Commands:**
- `/remember` - Scan & cache current project
- `/remember --list` - See all remembered projects
- `/remember --view` - View cached context
- `/remember --refresh` - Force re-scan

## Quick References

### Common Commands
```bash
# Start dev server (varies by project)
npm start
npm run dev

# Run tests
npm test
npm run test:watch

# Lint
npm run lint

# Create PR
gh pr create --base stage

# Remember this project (for faster future sessions)
/remember
```

### MCP Tools Available
- `mcp__atlassian__*` - Jira/Confluence operations
- `mcp__plugin_figma_figma__*` - Figma design access
- `mcp__github__*` - GitHub operations (if configured)

---

_This is my personal development context. Project-specific details are in each project's CLAUDE.md file._
