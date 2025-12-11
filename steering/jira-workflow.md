---
inclusion: always
---

# Jira Ticket to Pull Request Workflow

## CRITICAL: Automatic Workflow Execution

**When a user provides a Jira ticket ID with the "Jira" keyword (e.g., "Jira DEV-123" or "Implement Jira DEV-123"), you MUST execute the COMPLETE workflow automatically without asking for confirmation at each step.**

**This is a fully automated workflow. Execute ALL phases in sequence:**

0. Add "kiro" Label (FIRST THING!)
1. Pre-Implementation Check (ensure clean git state)
2. Ticket Analysis
3. Implementation
4. Pull Request Creation
5. Ticket Update (add PR link comment + transition status to "In Review")

**Do NOT stop after any phase. Continue to completion unless there's an error.**

**Do NOT ask "Should I create a PR?" or "Should I update the ticket?" - Just do it!**

## Overview

This workflow automates the complete development cycle from Jira ticket analysis to pull request creation. When triggered with a Jira ticket ID, execute all phases automatically without requiring confirmation at each step.

**Trigger phrases:** `"Jira DEV-123"`, `"Implement Jira DEV-123"`, `"Work on Jira ticket DEV-123"`

**Expected behavior:** Complete end-to-end execution from ticket fetch to pull request creation and Jira update.

## Workflow Phases

### Phase 0: Add "kiro" Label (First Action)

**When:** Immediately after fetching the Jira ticket, before any analysis or implementation work.

**Why:** Provides early tracking of AI-assisted work. If the workflow fails mid-way, the label still indicates Kiro attempted the ticket.

**Steps:**

1. Fetch ticket details:
```javascript
jira_get_issue({
  issue_key: "PROJECT-123",
  fields: "summary,description,labels,status,assignee,priority"
})
```

2. Add "kiro" to existing labels:
```javascript
// Extract current labels from the response
const currentLabels = issue.fields.labels; // e.g., ["bug", "high-priority"]

// Add "kiro" to the array
jira_update_issue({
  issue_key: "PROJECT-123",
  fields: {
    labels: [...currentLabels, "kiro"]
  }
})
```

**Important:** The `fields.labels` parameter replaces all labels, so always include existing labels in the array. Never use `additional_fields.update.labels` syntax as it fails with the Atlassian MCP server.

### Phase 1: Pre-Implementation Check

**Purpose:** Ensure a clean starting state before creating a new feature branch.

**Steps:**

1. Check for uncommitted changes using Git MCP:
```javascript
git_status()
```

2. If uncommitted changes exist, handle them:
   - **Recommended:** Stash with `git stash push -m "WIP: Stashing before starting new ticket"`
   - **Alternative:** Commit to current branch
   - **Alternative:** Ask user for guidance

3. Checkout and update main branch:
```javascript
git_checkout({ ref: "main" })
```
Then pull latest: `git pull origin main`

4. Verify clean state with `git_status()` - should show "nothing to commit, working tree clean"

**Why this matters:** Prevents mixing changes from different tickets and ensures the new branch is based on the latest main.

**Tool usage:** Always use Git MCP tools (`git_status`, `git_checkout`) instead of shell commands to avoid pager issues.

### Phase 2: Ticket Analysis

**Purpose:** Understand requirements and create an implementation plan.

**Steps:**

1. Parse ticket requirements:
   - Functional requirements (what needs to be built)
   - Technical requirements (how it should be built)
   - Acceptance criteria (definition of done)
   - Dependencies (related tickets or components)

2. Identify affected files:
   - Files to modify
   - New files to create
   - Tests to update
   - Documentation to change

3. Create implementation plan (optional - for transparency)

### Phase 3: Implementation

**Purpose:** Create branch and implement the code changes.

**Steps:**

1. Create feature branch using Git MCP:
```javascript
git_create_branch({
  branch: "feature/DEV-123-add-authentication"
})
```

**Branch naming:** `feature/TICKET-ID-brief-description`

**CRITICAL:** Immediately after creating the branch, checkout to it:
```javascript
git_checkout({
  ref: "feature/DEV-123-add-authentication"
})
```

**Why this is critical:** `git_create_branch` creates the branch but does NOT switch to it. You must explicitly checkout to the new branch before making any commits, otherwise commits will go to the current branch (usually main).

2. Implement changes following these principles:
   - Incremental: Make small, logical changes
   - Testable: Write tests alongside code
   - Documented: Add comments for complex logic
   - Conventional: Follow project coding standards (see `code-generation.md`)

3. Validate changes before committing:
   - Run linters and formatters
   - Execute unit tests
   - Check for compilation errors
   - Review changes with git diff

4. Commit changes locally using Git MCP:
```javascript
git_add({ files: ["path/to/file1.py", "path/to/file2.py"] })
git_commit({
  message: "feat(auth): Add OAuth2 authentication\n\nImplements OAuth2 flow with Google and GitHub providers\nas specified in the requirements.\n\nRefs: DEV-123"
})
```

**Commit format:** Follow conventional commits (see `commit-standards.md`)

### Phase 4: Push and Pull Request Creation

**Purpose:** Push code to CodeCommit and create a pull request.

**Steps:**

1. Ensure remote is configured:

Check existing remotes: `git remote -v`

**Extract repository name from the remote URL for later use:**
- Format: `codecommit::<region>://<profile>@<REPO_NAME>`
- Example: `codecommit::us-east-1://profile@my-repo` → REPO_NAME = "my-repo"

**Handle remote configuration:**
- If no remote: `git remote add origin codecommit::<AWS_REGION>://<AWS_PROFILE>@<REPO_NAME>`
- If remote uses HTTPS/SSH: Update with `git remote set-url origin codecommit::<AWS_REGION>://<AWS_PROFILE>@<REPO_NAME>`
- If remote already uses codecommit::: No action needed

**URL format:** `codecommit::<region>://<profile-name>@<repository-name>`

2. Push branch to CodeCommit:

Use `git push` CLI command: `git push origin feature/DEV-123-add-authentication`

**Why git push CLI:** Preserves local commit history and uses git-remote-codecommit with AWS credentials.

3. Create pull request using AWS MCP:

**Important:** Keep PR descriptions simple and single-line to avoid timeouts.

```javascript
aws___call_aws({
  service: "codecommit",
  operation: "create-pull-request",
  parameters: {
    title: "[DEV-123] Add OAuth2 authentication",
    description: "Implements OAuth2 authentication with Google and GitHub providers. Resolves DEV-123",
    targets: [{
      repositoryName: "<REPO_NAME_FROM_GIT_REMOTE>",
      sourceReference: "feature/DEV-123-add-authentication",
      destinationReference: "main"
    }]
  }
})
```

**PR description guidelines:**
- Use concise, single-line descriptions (max ~200 characters)
- Template: `"Implements [feature] with [key changes]. Resolves [TICKET-ID]"`
- Avoid multi-line strings with `\n`, special characters (✅, ❌), or markdown formatting

**Why:** Complex descriptions cause AWS MCP timeouts (300s).

**Repository information:**
- Repository name: Extract from git remote URL (already configured in Step 1)
- Region: AWS MCP automatically uses the configured region
- Never search for repositories or try different regions

**If pull request creation fails:**

Common causes:
1. **Timeout:** Description is too complex → Retry with simple single-line description
2. **Region mismatch:** MCP region doesn't match CodeCommit repository region → Report error to user and ask them to verify configuration

**Error handling:** Report the error with the exact message and explain possible causes. Don't attempt to search for repositories in different regions.

4. Verify pull request creation (optional):
```javascript
aws___call_aws({
  service: "codecommit",
  operation: "list-pull-requests",
  parameters: {
    repositoryName: "<REPO_NAME>",
    pullRequestStatus: "OPEN"
  }
})
```

### Phase 5: Ticket Update

**Purpose:** Update Jira ticket with PR link and transition status.

**Steps:**

1. Add PR link as comment:
```javascript
jira_add_comment({
  issueKey: "<TICKET_ID>",
  comment: "🤖 *Kiro Agent*\n\n✅ Pull Request Created\n\nPR #<PR_ID>: [<PR_TITLE>](<PR_URL>)\n\n**Changes:**\n- <summary of changes>\n\n**Status:** Ready for review"
})
```

**Note:** All Jira comments must start with "🤖 *Kiro Agent*" signature for transparency.

2. Transition ticket status:

**⚠️ REQUIRED STEP - DO NOT SKIP: Always attempt to transition the ticket to "In Review" or the appropriate next status.**

First, get available transitions:
```javascript
jira_get_transitions({
  issue_key: "<TICKET_ID>"
})
```

Then transition to "In Review" (or similar status like "Code Review", "Review", etc.):
```javascript
jira_transition_issue({
  issue_key: "<TICKET_ID>",
  transition: "<TRANSITION_ID_FOR_IN_REVIEW>"
})
```

**Note:** If the transition fails (e.g., workflow doesn't support it or ticket is already in the right status), that's okay - continue with the workflow. But always attempt the transition.

3. Update custom fields (optional):
   - Set "PR Link" field if it exists
   - Update story points if estimated

**Note:** Do NOT use `jira_create_remote_issue_link` - the PR link in the comment is sufficient.

---

**⚠️ BEFORE COMPLETING: Verify you executed BOTH required steps above:**
1. **Posted PR link comment** (Step 1) - Check you called `jira_add_comment`
2. **Transitioned ticket status** (Step 2) - Check you called `jira_get_transitions` AND `jira_transition_issue`

**If you skipped the transition step, go back and do it now!**

---

### ✅ Phase 5 Completion Checklist (VERIFY YOU DID ALL):

Before proceeding, verify you completed ALL of these steps:

- [ ] ✅ Posted comment with PR link (Step 1)
- [ ] ✅ Transitioned ticket status to "In Review" (Step 2)
- [ ] ✅ "kiro" label was added in Phase 0 (verify it was done)

**If you didn't do ALL of the above, go back and complete the missing steps!**

## MCP Tool Usage

**Always use MCP server tools instead of CLI commands** for better authentication, structured responses, and cross-platform compatibility.

### Git Operations → Use Git MCP Server

Use these MCP tools:
- `git_status`, `git_create_branch`, `git_checkout`, `git_add`, `git_commit`
- `git_diff_unstaged`, `git_diff_staged`, `git_log`, `git_branch`

**Exception:** Use `git push` CLI command (required for git-remote-codecommit)

**Why MCP tools:** Git MCP server is configured with `GIT_PAGER=cat` to prevent pager issues that require manual 'q' press.

### AWS Operations → Use AWS MCP Server

Use `aws___call_aws` with service/operation/parameters:
```javascript
aws___call_aws({
  service: "codecommit",
  operation: "create-pull-request",
  parameters: { ... }
})
```

Never use `aws codecommit ...` CLI commands.

### Jira Operations → Use Atlassian MCP Server

Use these MCP tools:
- `jira_get_issue`, `jira_update_issue`, `jira_add_comment`
- `jira_transition_issue`, `jira_search_issues`

Never use curl to Jira REST API.

## Label Management

**The Atlassian MCP server requires using `fields.labels` which replaces all labels.**

**Adding labels:**

1. Fetch current labels: `jira_get_issue` with `fields="labels"`
2. Add new label to the array:
```javascript
jira_update_issue({
  issue_key: "PROJECT-123",
  fields: {
    labels: ["existing-label-1", "existing-label-2", "new-label"]
  }
})
```

**If ticket has no existing labels (or labels field is missing from response):**
```javascript
jira_update_issue({
  issue_key: "PROJECT-123",
  fields: { labels: ["new-label"] }
})
```

**Important edge case:** If the Jira API response has no `labels` field at all (not even an empty array), treat it as empty labels `[]` and proceed with setting `labels: ["new-label"]`. Do not stop the workflow - this is a normal response for tickets with no labels.

**Removing labels:**

To remove specific labels while keeping others:
1. Fetch current labels
2. Filter out the labels to remove
3. Set the remaining labels:
```javascript
jira_update_issue({
  issue_key: "PROJECT-123",
  fields: { labels: ["keep-this", "and-this"] }
})
```

**Key points:**
- Always use `fields.labels` (replaces all labels)
- Always fetch current labels first before updating
- Include all existing labels you want to keep in the array
- Never use `additional_fields.update.labels` syntax

## Error Handling

### Git Push Failures

If `git push` fails:

1. Verify git-remote-codecommit is installed: `pip list | grep git-remote-codecommit`
2. Check AWS credentials: `aws sts get-caller-identity --profile <PROFILE_NAME>`
3. Verify CodeCommit permissions: `aws codecommit get-repository --repository-name <REPO_NAME> --profile <PROFILE_NAME>`
4. Check remote URL format: `git remote -v` (should show `codecommit://profile-name@repository-name`)

Common errors:
- "fatal: unable to access": AWS credentials not configured
- "fatal: repository not found": Wrong repository name or no permissions
- "codecommit: command not found": git-remote-codecommit not installed

### Pull Request Creation Failures

**Common causes:**

1. **Timeout (300s):** Description is too complex
   - Solution: Use simple, single-line description (max ~200 chars)
   - Example: `"Implements OAuth2 authentication with JWT tokens. Resolves DEV-123"`

2. **Region mismatch:** AWS MCP configured for one region but CodeCommit repository is in another
   - Solution: Report error to user and ask them to verify region configuration

**What not to do:**
- Don't search for repositories in different regions
- Don't run `aws codecommit list-repositories --region <ANY_REGION>`
- Don't assume region from API Gateway URLs or other AWS resources in the code

**Error response:**
```
I tried to create the PR but got an error: [ERROR_MESSAGE]

This might be because:
1. The PR description was too complex (I'll retry with a simpler description), OR
2. The AWS MCP is configured for region X, but your CodeCommit repository 
   might be in a different region.

The code has been pushed successfully to the branch. Let me try with a simpler 
description first...
```

### Jira Update Failures

If ticket update fails:
1. Verify Jira API token is valid
2. Check ticket exists and is accessible
3. Ensure transition is valid for current status
4. Verify user has permission to update ticket

## Best Practices

### Code Quality
- Follow project's coding standards (see `code-generation.md`)
- Write self-documenting code
- Add comments only when necessary
- Keep functions small and focused

### Testing
- Write tests before or alongside code
- Aim for high coverage of new code
- Include edge cases and error scenarios
- Run full test suite before pushing

### Communication
- Keep PR descriptions clear and concise
- Link all related tickets and PRs
- Tag relevant reviewers
- Respond promptly to review comments

### Git Hygiene
- Keep commits atomic and logical
- Write clear commit messages (see `commit-standards.md`)
- Avoid committing generated files
- Don't commit secrets or credentials

## Workflow Execution Summary

When a user provides a Jira ticket ID:

1. Fetch ticket and add "kiro" label immediately (Phase 0)
2. Check git state and prepare clean workspace (Phase 1)
3. Analyze the ticket (Phase 2)
4. Create branch and implement code (Phase 3)
5. Push and create pull request (Phase 4)
6. Update Jira ticket (Phase 5)

**Execute all phases automatically without stopping for confirmation** unless an error occurs or the user explicitly asks to stop.

**Example execution:**
```
User: "Jira DEV-123"

Execution:
→ Fetching Jira ticket DEV-123...
→ Adding "kiro" label to ticket... ✓
→ Checking git status...
→ Stashing uncommitted changes...
→ Checking out main branch...
→ Pulling latest changes...
→ Analyzing requirements...
→ Creating branch feature/DEV-123-add-authentication...
→ Implementing OAuth2 authentication...
→ Running tests... ✓
→ Committing changes locally...
→ Pushing branch to CodeCommit...
→ Creating pull request...
→ Updating Jira ticket DEV-123...

✅ Complete! 
- Branch: feature/DEV-123-add-authentication
- PR: https://console.aws.amazon.com/codesuite/codecommit/.../pull-requests/42
- Jira ticket updated and moved to "In Review"
```

This is the expected behavior for a complete workflow execution.
