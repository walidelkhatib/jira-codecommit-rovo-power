---
inclusion: always
---

# Pull Request Patch Workflow

## CRITICAL: Automatic PR Patching

**When a user asks to patch a PR (e.g., "Patch PR #42" or "Fix the issues in PR #42"), you MUST execute the COMPLETE patch workflow automatically.**

**This is a fully autonomous workflow. Execute ALL steps automatically without waiting for confirmation.**

**Trigger phrases:** `"Patch PR #42"`, `"Address review comments in PR #42"`, `"Fix the issues in PR #42"`

**Do NOT ask for confirmation at any step. Proceed immediately through all steps.**

## Overview

This workflow automates the implementation of PR review feedback from fetching review comments to pushing fixes and posting updates. When triggered with a PR number, execute all steps automatically without requiring confirmation.

**Expected behavior:** Complete end-to-end execution from fetching comments to implementing fixes and updating both PR and Jira.

## Workflow Steps

### Step 1: Fetch PR Details and Review Comments

**Purpose:** Identify the PR branch and understand what needs to be fixed.

**Steps:**

1. Fetch PR details to identify the branch using AWS MCP:
```javascript
aws___call_aws({
  service: "codecommit",
  operation: "get-pull-request",
  parameters: {
    pullRequestId: "<PR_ID>"
  }
})
```

**Extract from response:**
- Source branch name (the feature branch to checkout)
- Destination branch name (usually main/master)
- Repository name
- Source commit ID
- Destination commit ID

2. Fetch review comments using AWS MCP:
```javascript
aws___call_aws({
  service: "codecommit",
  operation: "get-comments-for-pull-request",
  parameters: {
    pullRequestId: "<PR_ID>",
    repositoryName: "<REPO_NAME>"
  }
})
```

3. Analyze and categorize comments:
   - 🔴 Critical issues (must fix)
   - 🟡 Important issues (should fix)
   - 🔵 Suggestions (nice to have)

4. Determine scope:
   - If user specified a specific issue (e.g., "Fix the SQL injection"), only address that
   - If user said "Fix the critical issues", only address critical issues
   - If user said "Patch PR" or "Address review comments", address all critical and important issues

### Step 2: Show Implementation Plan (Optional)

**Purpose:** Provide transparency about what will be fixed.

**You may show the plan but proceed immediately without waiting:**

```markdown
Found the following issues to address in PR #42:

🔴 Critical Issues:
1. SQL injection vulnerability in `src/api/users.py` line 45
   - Will use parameterized queries

2. Hardcoded API key in `config.py` line 12
   - Will move to environment variable

🟡 Important Issues:
3. Missing error handling in `service.py` line 78
   - Will add try-catch block

4. Missing tests for edge cases
   - Will add tests for null inputs

Proceeding with implementation...
```

### Step 3: Checkout PR Branch

**Purpose:** Switch to the PR branch to implement fixes.

**Steps:**

1. Checkout the PR branch using Git MCP (use source branch name from Step 1):
```javascript
git_checkout({
  ref: "<SOURCE_BRANCH_NAME_FROM_PR>"
})
```

Example: If PR details show source branch is "feature/MFLP-12-add-message-edit":
```javascript
git_checkout({
  ref: "feature/MFLP-12-add-message-edit"
})
```

2. Pull latest changes: `git pull origin <SOURCE_BRANCH_NAME_FROM_PR>`

**Important:** 
- Always use Git MCP tools (`git_checkout`) instead of shell commands to avoid pager issues
- Use the exact source branch name from the PR details fetched in Step 1
- Never guess the branch name

### Step 4: Implement Fixes

**Purpose:** Fix the issues identified in the review comments.

**Implementation order:**
1. Critical issues first
2. Important issues second
3. Suggestions if user specifically requested

**Follow standards:**
- Use `code-generation.md` standards
- Add tests for fixes
- Update documentation if needed
- Keep changes minimal and focused

**Implementation guidelines:**

#### For Security Issues
- SQL injection → Use parameterized queries
- XSS vulnerabilities → Escape output, validate input
- Hardcoded secrets → Move to environment variables
- Missing authentication → Add auth checks
- Missing authorization → Add permission checks

#### For Code Quality Issues
- Code duplication → Extract to shared function
- Complex logic → Simplify and add comments
- Poor naming → Rename variables/functions
- Missing error handling → Add try-catch blocks

#### For Testing Issues
- Missing tests → Add unit tests
- Missing edge case tests → Add edge case coverage
- Failing tests → Fix the underlying issue

#### For Performance Issues
- Inefficient queries → Optimize or add indexes
- Missing caching → Add appropriate caching
- Memory leaks → Ensure proper cleanup

### Step 5: Show Changes (Optional)

**Purpose:** Provide transparency about what changed.

**You may show changes but proceed immediately without waiting:**

Use Git MCP tool:
```javascript
git_diff_unstaged()
```

**For viewing commit history, use Git MCP:**
```javascript
git_log({ max_count: 5 })
```

**Important:** Always use Git MCP tools instead of shell commands to avoid pager issues.

**Optionally present summary:**
```markdown
Changes made:
- src/api/users.py: Fixed SQL injection with parameterized queries
- config.py: Moved API key to environment variable
- service.py: Added error handling
- tests/test_users.py: Added edge case tests

Committing and pushing...
```

### Step 6: Commit and Push Changes

**Purpose:** Save and push the fixes to CodeCommit.

**Steps:**

1. Commit the changes using Git MCP:
```javascript
git_add({
  files: ["src/api/users.py", "config.py", "service.py", "tests/test_users.py"]
})

git_commit({
  message: "fix: address PR review comments\n\n- Fix SQL injection vulnerability in users.py\n- Move API key to environment variable\n- Add error handling in service.py\n- Add edge case tests\n\nAddresses review comments from PR #42"
})
```

**Commit message format (follow `commit-standards.md`):**
```
fix: address PR review comments

- Fix [critical issue 1]
- Fix [critical issue 2]
- Improve [important issue 1]
- Add [missing tests]

Addresses review comments from PR #<PR_ID>
```

2. Push to CodeCommit using git CLI:

Use the source branch name from Step 1:
```bash
git push origin <SOURCE_BRANCH_NAME_FROM_PR>
```

Example: `git push origin feature/MFLP-12-add-message-edit`

### Step 7: Post Update Comment on PR

**Purpose:** Notify reviewers that feedback has been addressed.

**Steps:**

Post update comment using AWS MCP:
```javascript
aws___call_aws({
  service: "codecommit",
  operation: "post-comment-for-pull-request",
  parameters: {
    pullRequestId: "<PR_ID>",
    repositoryName: "<REPO_NAME>",
    beforeCommitId: "<BEFORE_COMMIT>",
    afterCommitId: "<AFTER_COMMIT>",
    content: "🤖 **Kiro Agent**\n\n## ✅ Review Comments Addressed\n\nI've implemented the following fixes:\n\n- 🔴 Fixed SQL injection vulnerability in `users.py`\n- 🔴 Removed hardcoded API key, now using environment variable\n- 🟡 Added error handling in `service.py`\n- 🟡 Added edge case tests\n\nAll tests passing. Ready for re-review!"
  }
})
```

**Comment format:**
- Must start with "🤖 **Kiro Agent**" signature
- List of issues addressed
- Brief description of each fix
- Test status (if applicable)
- Ready for re-review message

### Step 8: Update Jira Ticket (If Linked)

**Purpose:** Update the linked Jira ticket with patch summary.

**Steps:**

1. Extract Jira ticket ID from PR:

From the PR title or description fetched in Step 1, extract the Jira ticket ID.

Common patterns:
- PR title: `[DEV-123] Add authentication`
- PR title: `DEV-123: Add authentication`
- PR description: Contains link to `https://company.atlassian.net/browse/DEV-123`

If no Jira ticket is found, skip Step 8.

2. Add "kiro" label to ticket:

**Fetch current labels:**
```javascript
jira_get_issue({
  issue_key: "<TICKET_ID>",
  fields: "labels"
})
```

**Add "kiro" to labels array:**
```javascript
jira_update_issue({
  issue_key: "<TICKET_ID>",
  fields: {
    labels: [...currentLabels, "kiro"]
  }
})
```

**If ticket has no existing labels (or labels field is missing from response):**
```javascript
jira_update_issue({
  issue_key: "<TICKET_ID>",
  fields: { labels: ["kiro"] }
})
```

**Important:** `fields.labels` replaces all labels, so always include existing labels in the array. Never use `additional_fields.update.labels` syntax.

**Edge case:** If the Jira API response has no `labels` field at all (not even an empty array), treat it as empty labels `[]` and proceed with setting `labels: ["kiro"]`. Do not stop the workflow - this is normal for tickets with no labels.

3. Transition ticket status:

**Always attempt to transition the ticket back to "In Review" (ready for re-review).**

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

**Note:** If the transition fails (e.g., ticket is already in "In Review" or workflow doesn't support it), that's okay - continue with the workflow. But always attempt the transition.

**Why transition:** After fixes are applied, the PR is ready for re-review. This signals to reviewers that feedback has been addressed.

4. Post comment to Jira ticket:

```javascript
jira_add_comment({
  issueKey: "<TICKET_ID>",
  comment: "🤖 *Kiro Agent*\n\n✅ PR Review Comments Addressed\n\nI've updated PR #<PR_ID> with fixes for the review feedback:\n\n🔴 Fixed SQL injection vulnerability in users.py\n🔴 Removed hardcoded API key, moved to environment variable\n🟡 Added error handling in service.py\n🟡 Added edge case tests\n\nPR: https://console.aws.amazon.com/codesuite/codecommit/repositories/<REPO_NAME>/pull-requests/<PR_ID>\n\nReady for re-review!"
})
```

**Comment format:**
- Must start with "🤖 *Kiro Agent*" signature
- Summary of fixes implemented
- List of critical and important issues addressed
- Link to the PR
- Status (ready for re-review)

**Why update Jira:**
- Keeps stakeholders informed
- Maintains traceability in Jira
- Notifies watchers that PR was updated
- Provides audit trail of work done
- "kiro" label enables tracking and analytics

---

### ✅ Step 8 Completion Checklist (VERIFY YOU DID ALL):

Before proceeding, verify you completed ALL of these steps:

- [ ] ✅ Extracted Jira ticket ID from PR (Step 1)
- [ ] ✅ Added "kiro" label to ticket (Step 2)
- [ ] ✅ Transitioned ticket status to "In Review" (Step 3)
- [ ] ✅ Posted comment with update (Step 4)

**If you didn't do ALL of the above, go back and complete the missing steps!**

## Example Workflows

### Example 1: Addressing All Comments

**User input:**
```
"Patch PR #42"
```

**Automatic execution:**
```
Fetching PR #42 details...
- Source branch: feature/MFLP-12-add-message-edit
- Repository: my-repo

Fetching review comments...

Found 4 issues to address:
🔴 Critical: SQL injection vulnerability
🔴 Critical: Hardcoded API key
🟡 Important: Missing error handling
🟡 Important: Missing edge case tests

Proceeding with implementation...

Checking out branch feature/MFLP-12-add-message-edit...
Implementing fixes...
- Fixed SQL injection in users.py
- Moved API key to environment variable in config.py
- Added error handling in service.py
- Added edge case tests in test_users.py

Committing changes...
Pushing to CodeCommit...
Posting update comment on PR...
Updating Jira ticket MFLP-12...

✅ Done! All review comments addressed and pushed to PR #42. Jira ticket updated.
```

### Example 2: Addressing Specific Issue

**User input:**
```
"Fix the SQL injection in PR #42"
```

**Automatic execution:**
```
Fetching PR #42 details...
- Source branch: feature/MFLP-8-message-delete
- Repository: my-repo

Fetching review comments...

Found SQL injection issue in `src/api/users.py` line 45.

Proceeding with fix...

Checking out branch feature/MFLP-8-message-delete...
- Fixed SQL injection with parameterized queries
- Added test to verify SQL injection is prevented

Committing changes...
Pushing to CodeCommit...
Posting update comment on PR...
Updating Jira ticket MFLP-8...

✅ Done! SQL injection fixed in PR #42. Jira ticket updated.
```

## Important Notes

### Autonomous Execution
- Execute all steps automatically without waiting for confirmation
- You may show plan/progress for transparency
- Proceed immediately through all steps
- Support both "all comments" and "specific issue" targeting

### Scope Handling
- "Patch PR #42" → Address all critical and important issues
- "Fix the critical issues in PR #42" → Address only critical issues
- "Fix the SQL injection in PR #42" → Address only that specific issue

### MCP Tool Usage
- Use `git_checkout` instead of `git checkout` shell command
- Use `git_status` instead of `git status` shell command
- Use `git_log` instead of `git log` shell command
- Use `git_branch` instead of `git branch` shell command
- Use `git_diff_unstaged` and `git_diff_staged` MCP tools
- Use `git_add` and `git_commit` MCP tools
- **Exception:** Use `git push` shell command (git-remote-codecommit requirement)
- **Why:** Git MCP server has `GIT_PAGER=cat` configured to prevent pager issues

### Standards to Follow
- Follow `code-generation.md` for code standards
- Follow `commit-standards.md` for commit messages
- Add tests for all fixes
- Keep changes minimal and focused
- Post update comment after pushing

## Workflow Execution Summary

When a user asks to patch a PR:

1. Fetch PR details to identify the source branch (Step 1)
2. Fetch and analyze review comments (Step 1)
3. Optionally show plan (Step 2) - but don't wait
4. Checkout the source branch from PR details (Step 3)
5. Implement all fixes (Step 4)
6. Optionally show changes (Step 5) - but don't wait
7. Commit and push to source branch (Step 6)
8. Post update comment on PR (Step 7)
9. Update Jira ticket if linked (Step 8):
   - Extract Jira ticket ID from PR
   - Add "kiro" label to ticket
   - Post comment with update

**Execute all steps automatically** without stopping for confirmation unless there's an error.

This is the expected behavior for a complete patch workflow execution.
