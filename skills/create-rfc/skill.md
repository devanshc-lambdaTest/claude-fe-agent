# Create RFC Skill

## Description
Creates an RFC (Request for Comments) document from gathered requirements, codebase exploration, and chosen approach. Uses the LambdaTest RFC template, populates it with context from previous phases, and creates a PR in the `internal-docs` repository.

## Triggers
- Called automatically as Phase 4 of `/implement` workflow
- Can be invoked standalone: `/create-rfc <ticket-id>`
- Can be invoked standalone: `/create-rfc` (manual mode)

---

## Prerequisites

This skill expects outputs from earlier phases (when run as part of `/implement`):
- `requirements.json` (Phase 1 - Gather Requirements)
- `exploration.json` (Phase 2 - Explore Codebase)
- `approach.json` (Phase 3 - Decide Approach)

When run standalone, it will prompt the user for required information.

---

## RFC Template

The RFC follows the official LambdaTest template from:
`LambdatestIncPrivate/internal-docs/docs/Features-RFC/TEMPLATE.md`

### Template Sections

| Section | Auto-populated from | Requires manual input |
|---------|--------------------|-----------------------|
| 1. Basic Information | Jira ticket, git config | PRD Link |
| 2. Classification | requirements.json | Confirmation |
| 3. Context & Problem Statement | requirements.json | Review |
| 4. Solution Design | approach.json, exploration.json | Architecture diagram |
| 5. Microservices Affected | exploration.json | Confirmation |
| 6. Dev Effort Estimates | approach.json | Estimates |
| 7. Impact Analysis | exploration.json | Layer owners |
| 8. DevOps & Infrastructure | approach.json | Confirmation |
| 9. Quality Assurance Strategy | Auto-detect test framework | QA owner |
| 10. Rollout Strategy | - | User input |
| 11. Sign-Offs | - | Names |
| 12. Additional Notes | - | Optional |

---

## Workflow

### Step 1: Gather Context

1. **Read previous phase outputs** (if available)
   - Load `requirements.json`, `exploration.json`, `approach.json` from `.claude/workflow-state/`
   - If running standalone, prompt user for:
     - Ticket ID (Jira or manual)
     - Problem description
     - Proposed solution

2. **Get author info**
   ```bash
   # Get GitHub username
   gh api user --jq '.login'
   # Get git author name
   git config user.name
   ```

3. **Get Jira ticket details** (if ticket ID provided)
   - Use `mcp__plugin_atlassian_atlassian__getJiraIssue` to fetch ticket details
   - Extract: title, description, acceptance criteria, linked PRD

### Step 2: Populate RFC Document

Build the RFC markdown by filling in the template sections:

#### Section 1: Basic Information
```markdown
| Field | Details |
|-------|---------|
| **Author** | @<github-username> |
| **Date** | <current-date in DD-MMM-YYYY> |
| **JIRA Ticket** | [<ticket-id>](https://lambdatest.atlassian.net/browse/<ticket-id>) |
| **PRD Link** | <from-jira-or-ask-user> |
| **Reopened?** | No |
| **Review Status** | Draft |
```

#### Section 2: Classification
- Determine type from requirements (Bug Fix / New Feature / Enhancement)
- Present to user for confirmation

#### Section 3: Context & Problem Statement
- For bugs: Extract RCA from requirements, ask user for 5 Whys
- For features: Map problem statement and success metrics from requirements

#### Section 4: Solution Design
- **High-Level Architecture:** Generate a Mermaid diagram from the chosen approach
- **Technical Details:**
  - Flow details from approach.json
  - Data structures / DB changes from exploration
  - API changes from approach

#### Section 5: Microservices Affected
- List services identified during codebase exploration
- Ask user to confirm and add any missing services

#### Section 6: Dev Effort Estimates
- Ask user for total dev effort estimate
- Ask for per-service breakdown (optional)
- Valid estimates: 2h, 4h, 6h, or 8h per service

#### Section 7: Impact Analysis
- Pre-fill layers based on exploration findings
- Ask user to confirm impacted layers and assign owners

#### Section 8: DevOps & Infrastructure
- Auto-detect from approach (new env vars, infra needs)
- Ask user to confirm

#### Section 9: Quality Assurance Strategy
- Auto-detect test framework from project context
- Ask user for QA owner and test strategy

#### Section 10: Rollout Strategy
- Default to standard phased rollout
- Ask user for feature flag key and rollout plan

#### Section 11: Sign-Offs
- Leave as placeholders for user to fill

#### Section 12: Additional Notes
- Include links to related Slack discussions, meeting notes if mentioned

### Step 3: Review with User

1. **Present the draft RFC** to the user in full
2. **Ask for review:**
   ```
   question: "How does this RFC draft look?"
   header: "RFC Review"
   options:
     - label: "Looks good, create PR"
       description: "Create RFC PR in internal-docs repo"
     - label: "I want to edit some sections"
       description: "Make changes before creating PR"
     - label: "Skip RFC for now"
       description: "Continue without creating RFC"
   ```
3. **If edits requested:** Ask which sections to modify, make changes, re-present
4. **If skipped:** Log skip reason, continue to next phase

### Step 4: Create RFC PR in internal-docs

1. **Clone or access the internal-docs repo**
   ```bash
   # Check if internal-docs is already cloned nearby
   # If not, clone it to a temp location
   gh repo clone LambdatestIncPrivate/internal-docs /tmp/internal-docs-rfc-<ticket-id> -- --depth 1
   ```

2. **Create a new branch**
   ```bash
   cd /tmp/internal-docs-rfc-<ticket-id>
   git checkout -b rfc/<ticket-id>-<short-description>
   ```

3. **Write the RFC file**
   - File path: `docs/Features-RFC/<ticket-id>-<short-title>.md`
   - Use the populated RFC content from Step 2

4. **Commit and push**
   ```bash
   git add docs/Features-RFC/<ticket-id>-<short-title>.md
   git commit -m "rfc: Add RFC for <ticket-id> - <title>"
   git push -u origin rfc/<ticket-id>-<short-description>
   ```

5. **Create the Pull Request**
   ```bash
   gh pr create \
     --repo LambdatestIncPrivate/internal-docs \
     --title "RFC: <ticket-title>" \
     --body "$(cat <<'EOF'
   ## RFC: <ticket-title>

   **Jira Ticket:** [<ticket-id>](https://lambdatest.atlassian.net/browse/<ticket-id>)

   This RFC proposes the technical design for <brief-description>.

   ### Sections
   - Problem Statement
   - Solution Design
   - Impact Analysis
   - Dev Effort Estimates
   - Rollout Strategy

   Please review and leave comments on specific sections.

   ---
   Generated with [Claude Code](https://claude.com/claude-code)
   EOF
   )"
   ```

6. **Capture PR URL** for linking in workflow state

### Step 5: Update Workflow State

1. **Save RFC output**
   Create `.claude/workflow-state/rfc.json`:
   ```json
   {
     "phase": 4,
     "status": "completed",
     "rfc_file": "docs/Features-RFC/<ticket-id>-<short-title>.md",
     "pr_url": "<pr-url>",
     "pr_number": "<pr-number>",
     "repo": "LambdatestIncPrivate/internal-docs",
     "created_at": "<timestamp>",
     "skipped": false
   }
   ```

2. **Clean up temp clone** (if used)
   ```bash
   rm -rf /tmp/internal-docs-rfc-<ticket-id>
   ```

---

## Phase Completion

```
═══════════════════════════════════════════════════
PHASE 4: CREATE RFC — COMPLETE
═══════════════════════════════════════════════════

RFC: <ticket-title>
PR:  <pr-url>
Repo: LambdatestIncPrivate/internal-docs

The RFC has been created and is ready for team review.
Proceeding to Phase 5: Plan Implementation...
═══════════════════════════════════════════════════
```

**Transition prompt:**
```
question: "RFC created. Ready to proceed to implementation planning?"
header: "Phase 4 Done"
options:
  - label: "Continue to Phase 5"
    description: "Proceed to Plan Implementation"
  - label: "Pause workflow"
    description: "Wait for RFC review before continuing"
  - label: "View RFC PR"
    description: "Open the RFC PR in browser"
```

---

## Standalone Usage

When invoked outside of `/implement`:

```bash
# With Jira ticket
/create-rfc TTN-30874

# Manual mode
/create-rfc
```

**Standalone workflow:**
1. Prompt for ticket ID or manual description
2. Fetch Jira details if ticket provided
3. Ask structured questions for each RFC section
4. Generate RFC document
5. Create PR in internal-docs
6. Return PR URL

---

## Error Handling

| Error | Action |
|-------|--------|
| No access to internal-docs repo | Ask user to check GitHub permissions, offer to save RFC locally |
| Jira ticket not found | Fall back to manual input |
| Branch already exists | Append timestamp suffix or ask user |
| PR creation fails | Save RFC locally, provide manual instructions |
| Template fetch fails | Use embedded template copy |

---

## Output

- **Primary:** RFC PR in `LambdatestIncPrivate/internal-docs`
- **Artifact:** `.claude/workflow-state/rfc.json`
- **File:** `docs/Features-RFC/<ticket-id>-<short-title>.md` in internal-docs repo
