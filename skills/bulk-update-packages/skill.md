---
name: bulk-update-packages
description: Bulk update npm packages across multiple frontend repositories. Useful for updating shared packages like @lambdatestincprivate/lt-common-header across all FE repos.
allowed-tools: Read, Write, Bash, AskUserQuestion, TodoWrite
---

# Bulk Update Packages Skill

## Purpose

Update common npm packages across multiple frontend repositories **using `gh` CLI** - no local clones required. Perfect for rolling out updates to shared packages like `@lambdatestincprivate/lt-common-header`, `@lambdatestincprivate/lt-components`, etc.

## Triggers

- `/bulk-update-packages <package-name> <version>` - Update package to specific version
- `/bulk-update-packages <package-name>` - Update package to latest version
- `/bulk-update-packages --status <package-name>` - Show specific package versions

### Examples

```bash
# Update lt-common-header to version 2.5.0 across all repos
/bulk-update-packages @lambdatestincprivate/lt-common-header 2.5.0

# Update to latest version
/bulk-update-packages @lambdatestincprivate/lt-common-header latest

# Check current versions
/bulk-update-packages --status @lambdatestincprivate/lt-common-header

```

---

## Prerequisites

- GitHub CLI (`gh`) authenticated (run `gh auth status` to verify)
- Read access to target repositories (minimum requirement)
- Fork capability for creating PRs (standard for all access levels)

---

## Workflow Mode: Fork-Only

> **CRITICAL:** This skill ALWAYS uses the fork workflow, regardless of your access level. Even if you have write/admin access to a repository, you MUST:
>
> 1. Fork the repository to your account first
> 2. Sync the fork with upstream before making changes
> 3. Create branches in YOUR fork only
> 4. Open PRs from your fork to the upstream repository

**Why fork-only?**

- Consistent workflow across all repositories
- Safer - no accidental commits to upstream
- Standard open-source contribution pattern
- Works regardless of permission level

---

## Repository Registry

> **CRITICAL INSTRUCTION:** Only scan and update the repositories **explicitly listed below**. Do NOT search the organization for other repositories. Do NOT use `gh repo list` or similar commands to discover repos. The registry below is the **single source of truth** - if a repo is not listed here, it should NOT be touched.

### LambdatestIncPrivate FE Repositories

```json
{
  "owner": "LambdatestIncPrivate",
  "teams": {
    "Application": [
      {
        "name": "LWC-2.0",
        "base_branch": "stage",
        "alias": "Falcon"
      },
      {
        "name": "mobile-web-client",
        "base_branch": "stage",
        "alias": "Mobile"
      },
      {
        "name": "hyperexecute-dashboard",
        "base_branch": "stage",
        "alias": "HyEx"
      },
      {
        "name": "kaneai-test-management-client",
        "base_branch": "stage",
        "alias": "Kane + Test Manager"
      },
      {
        "name": "lt-web-platform",
        "base_branch": "stage",
        "alias": "MonoRepo"
      },
      {
        "name": "dotlapse-frontend",
        "base_branch": "stage",
        "alias": "Smart UI"
      },
      {
        "name": "accessibility-testing-chrome-extension",
        "base_branch": "stage",
        "alias": "Accessibility"
      },
      {
        "name": "web-scanner-client",
        "base_branch": "stage",
        "alias": "Web Scanner"
      }
    ],
    "Enterprise": [
      {
        "name": "lambda-invigilator-client",
        "base_branch": "stage",
        "alias": "Accounts"
      },
      {
        "name": "lambda-ledger-client",
        "base_branch": "stage",
        "alias": "Billing"
      },
      {
        "name": "lambda-integration-client",
        "base_branch": "stage",
        "alias": "Integrations"
      },
      {
        "name": "analytics",
        "base_branch": "stage",
        "alias": "Analytics"
      }
    ]
  }
}
```

> **Note:** Update the repos list as needed. All operations happen via `gh` CLI. Repos are organized by team for tracking purposes but all are processed together during bulk updates.

### Common Packages to Update

```json
{
  "packages": [
    "@lambdatestincprivate/lt-common-header",
    "@lambdatestincprivate/lt-components"
  ]
}
```

---

## Workflow

### Step 1: Parse Input

Extract package name and version from command:

```
/bulk-update-packages @lambdatestincprivate/lt-common-header 2.5.0
                      ↑ package name                         ↑ version
```

If version is `latest` or not provided:

```bash
# Fetch latest version from npm
npm view <package-name> version
```

### Step 2: Scan Listed Repositories Only

**IMPORTANT:** Iterate ONLY over the repositories defined in the Repository Registry above. Do NOT:

- Search the GitHub organization for additional repos
- Use `gh repo list` or `gh api user/orgs` to discover repos
- Add any repository not explicitly listed in the registry

For each repository **in the registry**:

1. **Check if repo is accessible**

   ```bash
   gh repo view LambdatestIncPrivate/<repo-name> --json name 2>/dev/null
   ```

   - If accessible, proceed
   - If not, skip with error message

2. **Get package.json from base branch**

   ```bash
   gh api repos/LambdatestIncPrivate/<repo-name>/contents/package.json?ref=<base_branch>
   ```

   - Decode base64 content
   - Parse JSON to check for target package

3. **Parse and check for package**
   - Check `dependencies` and `devDependencies`
   - Record current version if found
   - Mark as "Not installed" if not found

4. **Generate scan report with access info**

   ```
   ═══════════════════════════════════════════════════════════════════════════════
   REPOSITORY SCAN RESULTS
   ═══════════════════════════════════════════════════════════════════════════════

   Package: @lambdatestincprivate/lt-common-header
   Target Version: 2.5.0 (latest)

   ┌──────────────────────────┬─────────────┬─────────────┬───────────┬──────────┐
   │ Repository               │ Current     │ Target      │ Status    │ Access   │
   ├──────────────────────────┼─────────────┼─────────────┼───────────┼──────────┤
   │ mobile-web-client        │ ^2.3.1      │ ^2.5.0      │ Outdated  │ write    │
   │ smartui-client           │ ^2.5.0      │ ^2.5.0      │ Current   │ write    │
   │ hyperexecute-portal      │ ^2.4.0      │ ^2.5.0      │ Outdated  │ read     │
   │ lt-components            │ -           │ -           │ N/A       │ write    │
   │ real-device-portal       │ ^2.3.0      │ ^2.5.0      │ Outdated  │ read     │
   └──────────────────────────┴─────────────┴─────────────┴───────────┴──────────┘

   Repos to update: 3 (all via fork workflow)
   ```

### Step 3: Confirm Update

```
question: "Ready to update 3 repositories via fork workflow?"
header: "Confirm"
options:
  - label: "Proceed with updates (Recommended)"
    description: "Fork repos, sync with upstream, and create PRs"
  - label: "Cancel"
    description: "Abort operation"
```

### Step 4: Create Updates (Fork Workflow - ALWAYS)

> **IMPORTANT:** ALL repositories use this fork workflow, regardless of your access level.

1. **Fork the repository (or use existing fork)**

   ```bash
   # Check if fork already exists
   gh repo view <your-username>/<repo-name> --json name 2>/dev/null

   # If not, create fork
   gh repo fork LambdatestIncPrivate/<repo-name> --clone=false
   ```

   - If fork exists, use it
   - If not, create new fork

2. **CRITICAL: Sync fork with upstream before any changes**

   ```bash
   # Sync the fork's base branch with upstream
   gh repo sync <your-username>/<repo-name> --branch <base_branch>
   ```

   > **WHY THIS IS CRITICAL:** If the fork is out of sync with upstream, your PR will have incorrect diffs or conflicts. ALWAYS sync before creating branches.

3. **Get the latest package.json from YOUR synced fork**

   ```bash
   # Get package.json content and SHA from your fork (now synced with upstream)
   gh api repos/<your-username>/<repo-name>/contents/package.json?ref=<base_branch>
   ```

   - Parse the content (base64 decode)
   - Save the SHA for the commit later

4. **Modify package.json and VALIDATE the change**

   ```javascript
   // Parse current package.json
   const pkg = JSON.parse(content);

   // Find and update the package version
   const currentVersion =
     pkg.dependencies["<package-name>"] ||
     pkg.devDependencies["<package-name>"];

   // CRITICAL VALIDATION: Only proceed if version actually changes
   if (
     currentVersion === targetVersion ||
     currentVersion === "^" + targetVersion
   ) {
     console.log("Package already at target version. Skipping.");
     return; // DO NOT create PR
   }

   // Update version (preserve prefix like ^, ~, or exact)
   if (pkg.dependencies["<package-name>"]) {
     pkg.dependencies["<package-name>"] = preservePrefix(
       currentVersion,
       targetVersion,
     );
   } else if (pkg.devDependencies["<package-name>"]) {
     pkg.devDependencies["<package-name>"] = preservePrefix(
       currentVersion,
       targetVersion,
     );
   }

   // Stringify with same formatting
   const updatedContent = JSON.stringify(pkg, null, 2) + "\n";
   ```

   > **VALIDATION RULE:** If the package version is already at target, SKIP this repo entirely. Do NOT create empty PRs.

5. **Create branch in YOUR fork**

   ```bash
   # Get the SHA of the base branch
   gh api repos/<your-username>/<repo-name>/git/refs/heads/<base_branch> --jq '.object.sha'

   # Create new branch from that SHA
   gh api repos/<your-username>/<repo-name>/git/refs -X POST \
     -f ref="refs/heads/deps/update-<package-short-name>-<version>" \
     -f sha="<base-branch-sha>"
   ```

6. **Commit updated package.json to YOUR fork**

   ```bash
   # Commit the modified file
   gh api repos/<your-username>/<repo-name>/contents/package.json -X PUT \
     -f message="deps: update <package-name> to <version>" \
     -f content="<base64-encoded-updated-content>" \
     -f sha="<original-file-sha>" \
     -f branch="deps/update-<package-short-name>-<version>"
   ```

7. **Create PR from YOUR fork to upstream**

   ```bash
   gh pr create \
     --repo LambdatestIncPrivate/<repo-name> \
     --head <your-username>:deps/update-<package-short-name>-<version> \
     --base <base_branch> \
     --title "deps: update <package-name> to <version>" \
     --body "<pr-body>"
   ```

   > **IMPORTANT:** The `--repo` flag specifies the UPSTREAM repo where the PR is created. The `--head` includes your username to indicate the PR comes from your fork.

### Step 5: Generate Summary Report

```
═══════════════════════════════════════════════════════════════════════════════
BULK UPDATE COMPLETE
═══════════════════════════════════════════════════════════════════════════════

Package: @lambdatestincprivate/lt-common-header@2.5.0

## Results

| Repository             | Status     | PR / Reason                                 |
|------------------------|------------|---------------------------------------------|
| mobile-web-client      | PR Created | https://github.com/LambdatestIncPrivate/mobile-web-client/pull/1234 |
| hyperexecute-portal    | PR Created | https://github.com/LambdatestIncPrivate/hyperexecute-portal/pull/567 |
| real-device-portal     | PR Created | https://github.com/LambdatestIncPrivate/real-device-portal/pull/89 |
| smartui-client         | Skipped    | Already at target version                   |
| lt-components          | Skipped    | Package not installed                       |

## Next Steps

1. Review and approve PRs in each repository
2. Ensure CI passes before merging
3. Merge PRs to deploy updates

## Created PRs
- [mobile-web-client#1234](https://github.com/LambdatestIncPrivate/mobile-web-client/pull/1234)
- [hyperexecute-portal#567](https://github.com/LambdatestIncPrivate/hyperexecute-portal/pull/567)
- [real-device-portal#89](https://github.com/LambdatestIncPrivate/real-device-portal/pull/89)

───────────────────────────────────────────────────────────────────────────────
```

---

## Commands Reference

| Command                                               | Description                          |
| ----------------------------------------------------- | ------------------------------------ |
| `/bulk-update-packages <package> <version>`           | Update package to specific version   |
| `/bulk-update-packages <package> latest`              | Update package to latest version     |
| `/bulk-update-packages <package>`                     | Update package to latest (shorthand) |
| `/bulk-update-packages --status`                      | Check all common packages versions   |
| `/bulk-update-packages --status <package>`            | Check specific package versions      |
| `/bulk-update-packages --dry-run <package> <version>` | Preview without changes              |

---

## GitHub CLI Commands Used

Since we use fork workflow exclusively, prefer `gh` CLI for reliability:

| Command                        | Purpose                               |
| ------------------------------ | ------------------------------------- |
| `gh repo view`                 | Check if fork already exists          |
| `gh repo fork`                 | Fork repo to your account             |
| `gh repo sync`                 | **CRITICAL:** Sync fork with upstream |
| `gh api .../contents/...`      | Read/write package.json via API       |
| `gh api .../git/refs`          | Create branch in fork                 |
| `gh pr create --repo upstream` | Create PR from fork to upstream       |
| `gh pr list`                   | Check if PR already exists            |

---

## Access Level Requirements

All operations use the fork workflow. Minimum required access:

| Permission | Can Read Upstream | Can Fork | Can Create PR | Result  |
| ---------- | ----------------- | -------- | ------------- | ------- |
| `admin`    | ✅                | ✅       | ✅            | Fork+PR |
| `write`    | ✅                | ✅       | ✅            | Fork+PR |
| `read`     | ✅                | ✅       | ✅            | Fork+PR |
| `none`     | ❌                | ❌       | ❌            | Skipped |

> **Note:** Even with admin/write access, we ALWAYS use fork workflow for consistency and safety.

---

## Safety Features

1. **Fork-only workflow** - NEVER commits directly to upstream repos, even with write access
2. **Fork sync required** - Always syncs fork with upstream before making changes
3. **Version validation** - Only creates PR if version actually changes (no empty PRs)
4. **No local changes** - All operations via GitHub API, no local repo needed
5. **Branch protection** - Creates new branch in fork, never modifies base branch directly
6. **PR-based workflow** - Changes require PR review before merge
7. **Registry-only** - Only touches repos explicitly listed in the registry
8. **Fork cleanup** - Option to delete forks after PRs are merged
9. **Dry run mode** - Preview before creating any PRs
10. **Version prefix preservation** - Keeps ^, ~, or exact version style
11. **Idempotent** - Won't create duplicate branches/PRs

---

## Error Handling

### Repository Not Found

```
Repository 'LambdatestIncPrivate/missing-repo' not found.
Skipping...
```

### No Access At All

```
No access to LambdatestIncPrivate/private-repo (not a collaborator).
Skipping...
```

### Package Not in Repo

```
Package @lambdatestincprivate/lt-common-header not found in lt-components.
Skipping...
```

### Fork Already Exists

```
Fork already exists: your-username/mobile-web-client
Syncing fork with upstream...
Using existing fork...
```

### Fork Out of Sync

```
Fork is behind upstream by 15 commits.
Running: gh repo sync your-username/mobile-web-client --branch stage
Fork synced successfully.
```

### Version Already at Target

```
Package @lambdatestincprivate/lt-common-header already at version ^0.3.62 in mobile-web-client.
Skipping - no changes needed.
```

### Branch Already Exists

```
Branch 'deps/update-lt-common-header-2.5.0' already exists in mobile-web-client.
Options:
1. Skip this repo
2. Update existing branch
3. Delete and recreate
```

### PR Already Exists

```
Open PR already exists for this update in mobile-web-client.
PR: https://github.com/LambdatestIncPrivate/mobile-web-client/pull/1230
Skipping...
```

---

## Example Usage

```bash
# Update lt-common-header to version 2.5.0 in all repos
/bulk-update-packages @lambdatestincprivate/lt-common-header 2.5.0

# Update lt-components to version 3.0.0
/bulk-update-packages @lambdatestincprivate/lt-components 3.0.0

# Update to latest version
/bulk-update-packages @lambdatestincprivate/lt-common-header latest

# Shorthand for latest
/bulk-update-packages @lambdatestincprivate/lt-common-header

# Check current versions of a package across all repos
/bulk-update-packages --status @lambdatestincprivate/lt-common-header

# Check all common LT packages versions
/bulk-update-packages --status

# Preview what would happen without making changes
/bulk-update-packages --dry-run @lambdatestincprivate/lt-common-header 2.5.0
```

---

## Configuration

### Adding/Removing Repositories

Edit the repository registry in this skill file to add or remove repos:

```json
{
  "name": "new-fe-repo",
  "base_branch": "stage"
}
```

### Adding Common Packages

Add packages to the common packages list:

```json
"@lambdatestincprivate/new-package"
```

---

## Workflow Diagram

```
┌─────────────────┐
│ Parse Input     │ ◄── package name + version
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Get Version     │ ◄── npm view <package> version (if latest)
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Scan Repos      │ ◄── ONLY repos in registry
│ (registry ONLY) │ ◄── get package.json, check versions
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Show Report     │ ◄── current vs target versions
│ Confirm Update  │
└────────┬────────┘
         │
         ▼
   ┌─────┴─────┐
   │ For each  │
   │ outdated  │
   │   repo    │
   └─────┬─────┘
         │
         ▼
┌─────────────────┐
│ Fork or Use     │ ◄── gh repo fork (if needed)
│ Existing Fork   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ ** SYNC FORK ** │ ◄── gh repo sync (CRITICAL!)
│ with upstream   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Get package.json│ ◄── from synced fork
│ from fork       │
└────────┬────────┘
         │
         ▼
   ┌─────┴─────┐
   │ Version   │
   │ changed?  │
   └─────┬─────┘
         │
    Yes  │  No ──► Skip repo
         │
         ▼
┌─────────────────┐
│ Create Branch   │ ◄── in YOUR fork
│ in fork         │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Commit updated  │ ◄── to YOUR fork branch
│ package.json    │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Create PR       │ ◄── from fork to upstream
│ fork → upstream │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│ Summary Report  │
│ with PR links   │
└─────────────────┘
```

---

## Notes

- **No npm install via API**: Since we're updating via GitHub API, `package-lock.json` won't be updated. The lock file will be updated when CI runs or when a developer pulls the branch locally.
- **CI Validation**: Rely on CI pipelines to validate the update (run tests, check types, etc.)
- **Fork Management**: Forks created for PRs can be deleted after merge if desired
