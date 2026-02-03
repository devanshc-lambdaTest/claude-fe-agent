---
name: create-rfc
description: Create RFC (Request for Comments) documents from JIRA tickets and code changes. Generates standardized RFC following LambdaTest SDLC process for automatic JIRA ticket creation.
allowed-tools: Read, Write, Bash, AskUserQuestion, Task, TaskOutput, Glob, Grep, WebFetch, mcp__plugin_atlassian_atlassian__getJiraIssue
---

# Create RFC Skill

## Purpose

Generate standardized RFC (Request for Comments) documents from JIRA tickets and code changes. The RFC follows LambdaTest's SDLC process and enables automatic JIRA ticket creation when merged.

**Powered by:** LambdaTest SDLC RFC Automation ([Confluence Doc](https://lambdatest.atlassian.net/wiki/spaces/PROD/pages/4941053961))

**Repository:** [github.com/LambdatestIncPrivate/internal-docs](https://github.com/LambdatestIncPrivate/internal-docs)

## Triggers

- `/create-rfc <JIRA-ID>` - Create RFC from JIRA ticket
- `/create-rfc` - Create RFC manually (will ask for details)
- `/create-rfc --from-pr <PR-URL>` - Create RFC from existing PR
- `/create-rfc --validate <path>` - Validate existing RFC file
- `/create-rfc --skip-phases` - Skip Phases 1-4 and go directly to RFC collection (legacy mode)

### Examples

```bash
# Create RFC from JIRA ticket (runs Phases 1-4 first)
/create-rfc TE-1812

# Create RFC manually (runs Phases 1-4 first)
/create-rfc

# Create from PR (extracts changes and JIRA info)
/create-rfc --from-pr https://github.com/LambdatestIncPrivate/lt-web-platform/pull/264

# Validate RFC before PR
/create-rfc --validate docs/Features-RFC/my-feature.md

# Skip phases, go straight to RFC (legacy behavior)
/create-rfc --skip-phases TE-1812
```

---

## Prerequisites

- Access to LambdaTest Atlassian (JIRA) via MCP
- Clone of `internal-docs` repository (or will guide to clone)
- JIRA ticket must exist before creating RFC

---

## RFC Template Location

**Template:** `docs/Features-RFC/TEMPLATE.md`

**To create new RFC:**
```bash
cp docs/Features-RFC/TEMPLATE.md docs/Features-RFC/<ticket-id>-<feature-name>.md
```

---

## Required Fields (For RFC Automation)

The following fields are **mandatory** for the automation to work:

| Field | Section | Format |
|-------|---------|--------|
| **JIRA Ticket** | Section 1 - Basic Information | `[TE-XXX](https://lambdatest.atlassian.net/browse/TE-XXX)` |
| **Microservices Affected** | Section 5 | Table with Service, Changes Required, DB Changes |
| **Total Dev Efforts** | Section 6 | `**Total: XX hours**` |

**Optional but Recommended:**
- Service-wise effort breakdown (auto-distributed if not provided)
- DevOps & Infrastructure requirements (triggers Platform Engineering ticket)

---

## Workflow Overview

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                          CREATE-RFC WORKFLOW                                │
│                                                                             │
│   Phase 1: Gather Requirements    → /gather-requirements                   │
│   Phase 2: Explore Codebase       → /explore-codebase                      │
│   Phase 3: Decide Approach        → /decide-approach                       │
│   Phase 4: Plan Implementation    → /plan-implementation                   │
│                                                                             │
│   ──────────── Phase outputs feed into RFC generation ────────────         │
│                                                                             │
│   Step 5: Determine RFC Type                                                │
│   Step 6: Collect RFC-Specific Information                                  │
│   Step 7: Check Infrastructure Requirements                                │
│   Step 8: Generate RFC Document                                             │
│   Step 9: Validate RFC                                                      │
│   Step 10: Present Summary                                                  │
│   Step 11: Offer Next Steps                                                 │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Why Phases 1-4 first?** The RFC quality depends heavily on deep understanding of the requirements, codebase, chosen approach, and implementation plan. Running these phases first ensures the RFC is grounded in actual codebase analysis rather than surface-level guesswork.

---

## Workflow

### Phase 1: Gather Requirements (`/gather-requirements`)

**Input:** JIRA ticket ID (if provided) or manual input
**Output:** `.claude/workflow-state/requirements.json`

Execute the `/gather-requirements` skill as-is. This phase:

1. **Auto-detects project context** from `package.json`, `CLAUDE.md`, and directory structure
2. **Fetches ticket details** from JIRA (using `mcp__plugin_atlassian_atlassian__getJiraIssue`) or GitHub Issues
3. **Collects manual input** if no ticket provided (type, summary, description, acceptance criteria)
4. **Extracts design links** (Figma URLs from ticket description/comments)
5. **Correlates with codebase** (identifies likely affected directories)
6. **Validates requirements** and saves to `requirements.json`
7. **Presents summary and gets approval** before proceeding

**If `--from-pr` was provided:**
- Extract JIRA ID from PR title/body first
- Get changed files using `gh pr diff`
- Pass the JIRA ID into the gather-requirements flow
- Store PR context (changed files, diff stats) alongside requirements

**Phase transition:**
```
═══════════════════════════════════════════════════════════════════════════════
PHASE 1: GATHER REQUIREMENTS COMPLETE
═══════════════════════════════════════════════════════════════════════════════
Requirements gathered and approved. Proceeding to Phase 2: Explore Codebase.
```

---

### Phase 2: Explore Codebase (`/explore-codebase`)

**Input:** `requirements.json` from Phase 1
**Output:** `.claude/workflow-state/exploration.json`

Execute the `/explore-codebase` skill as-is. This phase:

1. **Checks for project memory** (from `/remember`) - uses fast path if cached
2. **Launches 3 parallel exploration agents:**
   - Agent 1: Similar Features Explorer
   - Agent 2: Architecture & Flow Explorer
   - Agent 3: Patterns & Conventions Explorer
3. **Combines and deduplicates** key files from all agents
4. **Reads key files** (up to ~20 most important)
5. **Synthesizes findings** into comprehensive summary
6. **Presents and gets approval** before proceeding

**RFC value:** This phase identifies affected services, existing patterns, architecture layers, and integration points - all critical for RFC Sections 4, 5, 7.

**Phase transition:**
```
═══════════════════════════════════════════════════════════════════════════════
PHASE 2: CODEBASE EXPLORATION COMPLETE
═══════════════════════════════════════════════════════════════════════════════
Codebase explored. Proceeding to Phase 3: Decide Approach.
```

---

### Phase 3: Decide Approach (`/decide-approach`)

**Input:** `requirements.json`, `exploration.json`
**Output:** `.claude/workflow-state/approach.json`

Present 2-3 implementation approaches based on the codebase exploration:

1. **Analyze exploration results** to identify viable approaches
2. **For each approach, evaluate:**
   - Alignment with existing patterns (from exploration)
   - Complexity and risk
   - Maintainability
   - Testing ease
   - Affected services and scope
3. **Present approaches to user** with trade-offs:
   ```
   question: "Which approach should we take?"
   header: "Approach"
   options:
     - label: "Approach A: [name]"
       description: "[brief summary with trade-offs]"
     - label: "Approach B: [name]"
       description: "[brief summary with trade-offs]"
     - label: "Approach C: [name]"
       description: "[brief summary with trade-offs]"
   ```
4. **Save chosen approach** to `approach.json`:
   ```json
   {
     "metadata": {
       "workflow_id": "<from requirements>",
       "phase": 3,
       "created_at": "<timestamp>"
     },
     "chosen_approach": {
       "name": "<approach name>",
       "description": "<detailed description>",
       "rationale": "<why this was chosen>",
       "patterns_to_follow": ["<from exploration>"],
       "affected_services": ["<identified services>"],
       "risks": ["<identified risks>"],
       "mitigations": ["<risk mitigations>"]
     },
     "alternatives_considered": [
       {
         "name": "<name>",
         "description": "<description>",
         "rejected_reason": "<why not chosen>"
       }
     ],
     "phase_status": {
       "status": "approved",
       "next_phase": "plan_implementation"
     }
   }
   ```

**RFC value:** This phase determines the solution design, alternatives considered, and risk analysis - directly feeds RFC Sections 3, 4, 7.

**Phase transition:**
```
═══════════════════════════════════════════════════════════════════════════════
PHASE 3: APPROACH DECIDED
═══════════════════════════════════════════════════════════════════════════════
Approach approved. Proceeding to Phase 4: Plan Implementation.
```

---

### Phase 4: Plan Implementation (`/plan-implementation`)

**Input:** `requirements.json`, `exploration.json`, `approach.json`
**Output:** `.claude/workflow-state/plan.json`

Create a step-by-step implementation plan:

1. **Break down the chosen approach** into ordered implementation steps
2. **For each step, specify:**
   - Files to create/modify/delete
   - What changes to make
   - Dependencies on other steps
   - Estimated effort (2h, 4h, 6h, 8h increments)
3. **Group by service** (maps directly to RFC microservices table)
4. **Include test plan** per step
5. **Present plan to user** for approval
6. **Save plan** to `plan.json`:
   ```json
   {
     "metadata": {
       "workflow_id": "<from requirements>",
       "phase": 4,
       "created_at": "<timestamp>"
     },
     "implementation_plan": {
       "steps": [
         {
           "step_number": 1,
           "title": "<step title>",
           "service": "<service name>",
           "files": {
             "create": ["<paths>"],
             "modify": ["<paths>"],
             "delete": ["<paths>"]
           },
           "description": "<what to do>",
           "estimated_hours": 4,
           "dependencies": []
         }
       ],
       "total_estimated_hours": "<sum>",
       "services_summary": [
         {
           "service": "<name>",
           "hours": "<sum for this service>",
           "steps": ["<step numbers>"]
         }
       ]
     },
     "test_plan": {
       "unit_tests": ["<what to test>"],
       "integration_tests": ["<what to test>"],
       "e2e_tests": ["<what to test>"]
     },
     "phase_status": {
       "status": "approved",
       "next_phase": "generate_rfc"
     }
   }
   ```

**RFC value:** This phase produces the implementation plan, service-wise effort estimates, test strategy, and file change list - directly feeds RFC Sections 5, 6, 9, 12.

**Phase transition:**
```
═══════════════════════════════════════════════════════════════════════════════
PHASE 4: IMPLEMENTATION PLAN COMPLETE
═══════════════════════════════════════════════════════════════════════════════
Plan approved. All phase data collected. Now generating RFC document.
```

---

### Step 5: Determine RFC Type

**Auto-detect from Phase 1 requirements** (type field from `requirements.json`), but confirm with user:

```
question: "What type of change is this?"
header: "Type"
options:
  - label: "Bug Fix"
    description: "Fixing existing broken functionality"
  - label: "New Feature"
    description: "Adding new functionality"
  - label: "Enhancement"
    description: "Improving existing functionality (refactor, tech debt, UX)"
```

### Step 6: Collect RFC-Specific Information

**Use data from Phases 1-4 to pre-fill, then collect any remaining gaps.**

**For Bug Fixes (RCA required):**
- Pre-fill root cause from Phase 2 exploration (code analysis) and Phase 3 approach
- Ask user to confirm/refine:
  ```
  question: "Based on codebase analysis, here's the likely root cause: [auto-detected]. Is this correct?"
  header: "Root Cause"
  options:
    - label: "Yes, that's correct"
      description: "Use the detected root cause"
    - label: "I'll refine it"
      description: "Modify the root cause analysis"
  ```
- Customer impact description

**For Features/Enhancements:**
- Pre-fill problem statement from Phase 1 requirements
- Pre-fill solution design from Phase 3 approach
- Ask for success metrics (optional)

**Services and Effort (auto-filled from Phase 4):**
- Service list and effort breakdown come directly from `plan.json` → `services_summary`
- Present for confirmation:
  ```
  Based on the implementation plan:
  - <service-1>: Xh
  - <service-2>: Xh
  Total: XXh
  ```
- Only ask if plan.json has no service breakdown

> **Note:** Valid estimates per service: 2h, 4h, 6h, 8h. Services >8h are auto-split into chunks.

### Step 7: Check Infrastructure Requirements

```
question: "Does this require DevOps/Infrastructure changes?"
header: "DevOps"
options:
  - label: "No"
    description: "No infrastructure changes needed"
  - label: "Yes"
    description: "New CI/CD, env vars, infrastructure, monitoring"
```

If yes, collect details for Section 8.

### Step 8: Generate RFC Document

1. **Create RFC file:**
   ```bash
   cp docs/Features-RFC/TEMPLATE.md docs/Features-RFC/<ticket-id>-<feature-name>.md
   ```

2. **Fill in all sections using Phase 1-4 data:**

   | RFC Section | Source |
   |-------------|--------|
   | Section 1: Basic Information | Phase 1 `requirements.json` (ticket ID, author, date) |
   | Section 2: Classification | Step 5 (confirmed type) |
   | Section 3: Context & Problem | Phase 1 (requirements) + Phase 2 (root cause from exploration) |
   | Section 4: Solution Design | Phase 3 `approach.json` (chosen approach, rationale, architecture) |
   | Section 5: Microservices Affected | Phase 4 `plan.json` → `services_summary` |
   | Section 6: Dev Effort Estimates | Phase 4 `plan.json` → `total_estimated_hours` + per-service |
   | Section 7: Impact Analysis | Phase 2 `exploration.json` (architecture layers, integration points) |
   | Section 8: DevOps & Infrastructure | Step 7 (user input) |
   | Section 9: QA Strategy | Phase 4 `plan.json` → `test_plan` |
   | Section 10: Rollout Strategy | Phase 3 `approach.json` → risks/mitigations |
   | Section 11: Sign-Offs | TBD placeholders |
   | Section 12: Additional Notes | Phase 4 (file change list), PR link if available |

3. **Add code context:**
   - Include before/after code snippets for bug fixes (from Phase 2 key files)
   - Include architecture diagrams (mermaid) for features (from Phase 2 architecture)
   - Link to PR if available
   - Include alternatives considered (from Phase 3)

### Step 9: Validate RFC

Run validation to ensure required fields are present:

```bash
./scripts/rfc-automation/validate-rfc.sh docs/Features-RFC/<file>.md
```

Check for:
- [ ] JIRA ticket link present and valid format
- [ ] Microservices table has at least one service
- [ ] Total dev effort is specified
- [ ] No TODO placeholders in required fields

### Step 10: Present Summary

```
═══════════════════════════════════════════════════════════════════════════════
RFC CREATED
═══════════════════════════════════════════════════════════════════════════════

**File:** docs/Features-RFC/<ticket-id>-<feature-name>.md
**JIRA:** <JIRA-ID>
**Type:** Bug Fix / New Feature / Enhancement

## Summary
<brief description>

## Approach
<chosen approach from Phase 3>

## Services Affected
- <service-1> (Xh)
- <service-2> (Xh)

## Total Effort: XX hours

## Phases Completed
- [x] Phase 1: Gather Requirements
- [x] Phase 2: Explore Codebase
- [x] Phase 3: Decide Approach
- [x] Phase 4: Plan Implementation
- [x] RFC Document Generated

## What Happens Next

1. Review the RFC file
2. Create PR to `internal-docs` repository
3. PR validation will run automatically
4. On merge → JIRA tickets created automatically:
   - Story Chunks for each service
   - QA Chunks (Dev/Stage/Prod)
   - Platform Engineering chunk (if DevOps work)

## Commands

# Open RFC file
code docs/Features-RFC/<file>.md

# Create PR
cd /path/to/internal-docs
git checkout -b rfc/<ticket-id>
git add docs/Features-RFC/<file>.md
git commit -m "RFC: <ticket-id> - <title>"
git push origin rfc/<ticket-id>
gh pr create --base master
```

### Step 11: Offer Next Steps

```
question: "What would you like to do next?"
header: "Done"
options:
  - label: "Open RFC file"
    description: "View/edit the generated RFC"
  - label: "Create PR"
    description: "Create PR to internal-docs"
  - label: "Validate RFC"
    description: "Run validation script"
  - label: "Done"
    description: "Handle manually"
```

---

## RFC Document Structure

The generated RFC follows this structure:

| Section | Purpose | Required? |
|---------|---------|-----------|
| 1. Basic Information | Author, date, JIRA link, PRD | **Yes** |
| 2. Classification | Bug/Feature/Enhancement type | Yes |
| 3. Context & Problem | RCA or Problem Statement | Yes |
| 4. Solution Design | Architecture, technical details | Yes |
| 5. Microservices Affected | Services table | **Yes** |
| 6. Dev Effort Estimates | Total hours + breakdown | **Yes** |
| 7. Impact Analysis | Layer impact, on-prem | Yes |
| 8. DevOps & Infrastructure | CI/CD, env vars, infra | Optional |
| 9. QA Strategy | Test coverage, test data | Yes |
| 10. Rollout Strategy | Feature flags, phases | Yes |
| 11. Sign-Offs | Approvals | Placeholder |
| 12. Additional Notes | PR links, references | Optional |

---

## JIRA Ticket Creation (On Merge)

When RFC PR is merged to `master`, automation creates:

### Story Chunks (per service)
- Type: `Story Chunk`
- Title: `[service-name] Brief description`
- Estimate: From RFC (max 8h per chunk, auto-split if larger)
- Linked to: Parent JIRA ticket

### QA Chunks (3 per RFC)
- `[PARENT-KEY] Feature - QA Dev Env` (functional testing)
- `[PARENT-KEY] Feature - QA Stage Env` (regression testing)
- `[PARENT-KEY] Feature - QA Prod Env` (smoke testing)

### Platform Engineering Chunk (conditional)
- Created only if DevOps section has actual work items
- Type: `Story Chunk`
- Team: Platform Engineering

---

## Validation Rules

The RFC validation checks:

1. **JIRA Ticket** - Must be valid link format: `[TE-XXX](https://lambdatest.atlassian.net/browse/TE-XXX)`
2. **Microservices Table** - At least one service listed
3. **Total Dev Effort** - Must be specified as `**Total: XX hours**`
4. **No TODO Placeholders** - Required fields cannot have `TODO` markers
5. **Valid Estimates** - Service estimates must be 2, 4, 6, or 8 hours

---

## Formatting Rules (MDX Compatibility)

> **CRITICAL:** RFC files are processed by MDX (Markdown + JSX). Follow these rules to avoid parsing errors.

### DO NOT Use Inside Table Cells:
- `<br>` or `<br/>` tags - MDX cannot parse these in tables
- Any self-closing HTML tags (`<hr/>`, `<img/>`, etc.)
- Multi-line content with line breaks

### Instead, Use These Patterns:

**For multi-item lists in table cells:**
```markdown
<!-- BAD - Will cause MDX error -->
| Impact | Item 1<br>Item 2<br>Item 3 |

<!-- GOOD - Short summary in table, details outside -->
| Impact | Multiple issues affecting performance and UX |

**Impact Details:**
- Item 1
- Item 2
- Item 3
```

**For long content:**
```markdown
<!-- BAD - Long content with line breaks in cell -->
| Root Cause | This is a very long explanation<br>with multiple lines<br>that breaks MDX |

<!-- GOOD - Brief summary, expand below table -->
| Root Cause | Polling mechanism lacks in-flight request tracking |

**Root Cause Details:**
The polling hooks use `setInterval` which fires regardless of whether
the previous API request has completed...
```

### Safe Markdown in Tables:
- **Bold**: `**text**` ✅
- **Inline code**: `` `code` `` ✅
- **Links**: `[text](url)` ✅
- **Simple text**: Plain text ✅

### Unsafe in Tables:
- `<br>`, `<br/>` ❌
- `<hr>`, `<hr/>` ❌
- Multi-line content ❌
- Block-level elements ❌
- Nested lists ❌

---

## Error Handling

### JIRA Not Found
```
JIRA ticket TE-XXXX not found. Please verify the ticket ID.
Options:
1. Enter correct ticket ID
2. Continue with manual input
```

### RFC Already Exists
```
RFC file already exists: docs/Features-RFC/TE-1812-*.md
Options:
1. Overwrite existing
2. Create new with suffix
3. Cancel
```

### Validation Failed
```
RFC validation failed:
- [ ] Missing: Microservices Affected table
- [ ] Missing: Total Dev Effort

Fix these issues and re-validate.
```

### Internal-docs Not Cloned
```
internal-docs repository not found.

Clone it first:
  git clone git@github.com:LambdatestIncPrivate/internal-docs.git

Or specify path:
  /create-rfc TE-1812 --repo-path /path/to/internal-docs
```

---

## Configuration

### Default Paths

| Config | Default | Override |
|--------|---------|----------|
| internal-docs repo | `~/developer/company/internal-docs` | `--repo-path` |
| RFC template | `docs/Features-RFC/TEMPLATE.md` | - |
| Output directory | `docs/Features-RFC/` | - |
| Atlassian Cloud ID | `3def4f78-101d-4614-9b65-735c17a98a93` | - |

### Service Name Mapping

```json
{
  "apps/hyperexecute": "hyperexecute-frontend",
  "apps/mobile-web-client": "mobile-web-client",
  "apps/magic-leap-dashboard": "magic-leap-frontend",
  "packages/TestPageApp": "test-page-app",
  "services/": "<extract-service-name>"
}
```

---

## Examples

### Bug Fix RFC
```bash
/create-rfc TE-1812
```

Generates RFC with:
- Section 3.1 filled with RCA (5 Whys)
- Before/after code snippets in Section 4
- PR link in Section 12

### New Feature RFC
```bash
/create-rfc TTN-50000
```

Generates RFC with:
- Section 3.2 filled with Problem Statement
- Architecture diagram in Section 4
- Feature flag details in Section 10

### From Existing PR
```bash
/create-rfc --from-pr https://github.com/LambdatestIncPrivate/lt-web-platform/pull/264
```

Auto-extracts:
- JIRA ID from PR title/body
- Changed files and services
- Code diff for technical details

---

## Quick Reference

| Command | Description |
|---------|-------------|
| `/create-rfc <JIRA-ID>` | Create RFC from JIRA ticket (runs Phases 1-4 first) |
| `/create-rfc` | Create RFC manually (runs Phases 1-4 first) |
| `/create-rfc --from-pr <url>` | Create RFC from PR |
| `/create-rfc --validate <path>` | Validate RFC file |
| `/create-rfc --skip-phases` | Skip Phases 1-4, go directly to RFC collection |

---

## Notes

1. **JIRA Ticket Required** - RFC must reference an existing JIRA ticket
2. **One RFC per Ticket** - Each JIRA ticket should have one RFC
3. **Don't Modify Template** - `TEMPLATE.md` is the base, don't edit it directly
4. **Valid Estimates Only** - Use 2, 4, 6, or 8 hours per service
5. **Merge to Master** - PRs to `master` branch trigger automation
