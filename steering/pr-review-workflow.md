---
inclusion: always
---

# Pull Request Review Workflow

## CRITICAL: Automatic PR Review

**When a user asks to review a PR (e.g., "Review PR #42" or "Review pull request 42"), you MUST execute the COMPLETE review workflow automatically.**

**This is a fully automated review workflow. Execute ALL phases in sequence:**
1. Fetch PR Details
2. Analyze Changes
3. Perform Code Review
4. Post Review Comments (DIRECTLY ON CODECOMMIT PR - NOT JUST IN CHAT!)
5. Update PR Status
6. Update Jira Ticket (ADD "kiro" LABEL + POST COMMENT)

**Do NOT stop after any phase. Continue to completion unless there's an error.**

**Do NOT ask user if comments should be posted - post them automatically!**

## Overview

This workflow automates comprehensive code review from fetching PR details to posting review comments on CodeCommit and updating Jira. When triggered with a PR number, execute all phases automatically without requiring confirmation.

**Trigger phrases:** `"Review PR #42"`, `"Review pull request 42"`, `"Review CodeCommit PR 5"`

**Expected behavior:** Complete end-to-end execution from PR fetch to posting comments on CodeCommit PR and updating Jira ticket.

## Workflow Phases

### Phase 1: Fetch PR Details

**Purpose:** Get PR information and code changes for review.

**Steps:**

1. Get PR information using AWS MCP:
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
- PR title and description
- Source and destination branches
- Source and destination commit IDs
- Author
- Current status
- Linked Jira ticket (if any)

2. Get PR diff using AWS MCP:

**Important:** Use the correct parameter order to show changes in the right direction.

```javascript
aws___call_aws({
  service: "codecommit",
  operation: "get-differences",
  parameters: {
    repositoryName: "<REPO_NAME>",
    beforeCommitSpecifier: "<DESTINATION_COMMIT>",  // main (old state)
    afterCommitSpecifier: "<SOURCE_COMMIT>"         // feature (new state)
  }
})
```

**Parameter order is critical:**
- `beforeCommitSpecifier` = destination commit (main branch) = old state
- `afterCommitSpecifier` = source commit (feature branch) = new state
- This shows what the feature branch adds to main

**Never use `git_diff` or `git diff` shell commands** - they can show changes in the wrong direction.

**Analyze the diff:**
- Files changed
- Lines added/removed
- Type of changes (new files, modifications, deletions)

### Phase 2: Analyze Changes

**Purpose:** Understand the context and scope of the PR.

**Steps:**

1. Understand context:
   - What is the PR trying to accomplish?
   - What Jira ticket is it linked to?
   - What are the acceptance criteria?

2. Identify scope:
   - Which components are affected?
   - What's the blast radius?
   - Are there any breaking changes?

3. Categorize changes:
   - Feature code: New functionality
   - Bug fixes: Corrections to existing code
   - Tests: Unit/integration tests
   - Documentation: README, comments, docs
   - Configuration: Config files, environment variables
   - Dependencies: Package updates

### Phase 3: Perform Code Review

**Purpose:** Evaluate code quality, security, testing, and best practices.

**Review checklist:**

#### 1. Code Quality
- Readability: Is the code easy to understand?
- Naming: Are variables/functions well-named?
- Complexity: Is the code unnecessarily complex?
- DRY: Is there code duplication?
- Comments: Are complex sections documented?

#### 2. Functionality
- Correctness: Does the code do what it claims?
- Edge cases: Are edge cases handled?
- Error handling: Are errors caught and handled properly?
- Logic: Is the business logic sound?

#### 3. Testing
- Test coverage: Are there tests for new code?
- Test quality: Do tests actually test the functionality?
- Edge case tests: Are edge cases tested?
- Integration tests: Are integrations tested?

#### 4. Security
- Input validation: Is user input validated?
- SQL injection: Are queries parameterized?
- XSS prevention: Is output escaped?
- Secrets: Are there any hardcoded secrets?
- Authentication: Is auth properly implemented?
- Authorization: Are permissions checked?

#### 5. Performance
- Efficiency: Are there obvious performance issues?
- Database queries: Are queries optimized?
- Caching: Should caching be used?
- Memory leaks: Are resources properly cleaned up?

#### 6. Best Practices
- Conventions: Does it follow project conventions?
- Patterns: Are appropriate design patterns used?
- SOLID principles: Does it follow SOLID?
- Code standards: Follows `code-generation.md` standards?

#### 7. Documentation
- Code comments: Complex logic documented?
- API docs: Public APIs documented?
- README: README updated if needed?
- Changelog: Changes documented?

#### 8. Dependencies
- Necessary: Are new dependencies necessary?
- Security: Are dependencies secure (no known vulnerabilities)?
- License: Are licenses compatible?
- Maintenance: Are dependencies actively maintained?

### Phase 4: Post Review Comments

**Purpose:** Post detailed feedback directly on the CodeCommit PR.

**Important:** Always post comments directly on the PR using AWS MCP. Never just show feedback in chat - stakeholders need to see the review on the PR itself.

**Comment structure:**

For each issue found, post a comment with:

1. **Severity level:**
   - 🔴 Critical: Must fix before merge (security, bugs)
   - 🟡 Important: Should fix before merge (code quality, performance)
   - 🔵 Suggestion: Nice to have (style, optimization)
   - ✅ Praise: Good practices worth highlighting

2. **Location:** File path, line number(s), code snippet

3. **Issue description:** What's wrong and why is it a problem?

4. **Recommendation:** How to fix it with code example if applicable

**Comment format example:**
```markdown
🤖 **Kiro Agent**

## 🔴 Critical: SQL Injection Vulnerability

**File**: `src/api/users.py`
**Line**: 45

**Issue**:
The user input is directly concatenated into the SQL query, making it vulnerable to SQL injection attacks.

**Current Code**:
```python
query = f"SELECT * FROM users WHERE email = '{email}'"
cursor.execute(query)
```

**Recommendation**:
Use parameterized queries to prevent SQL injection:

```python
query = "SELECT * FROM users WHERE email = %s"
cursor.execute(query, (email,))
```

**References**:
- OWASP SQL Injection: https://owasp.org/www-community/attacks/SQL_Injection
```

**Posting comments using AWS MCP:**

```javascript
aws___call_aws({
  service: "codecommit",
  operation: "post-comment-for-pull-request",
  parameters: {
    pullRequestId: "<PR_ID>",
    repositoryName: "<REPO_NAME>",
    beforeCommitId: "<BEFORE_COMMIT>",
    afterCommitId: "<AFTER_COMMIT>",
    content: "🤖 **Kiro Agent**\n\n## 🔴 Critical: SQL Injection Vulnerability\n\n**File**: `src/api/users.py`\n**Line**: 45\n\n**Issue**: ...\n\n**Recommendation**: ..."
  }
})
```

**Note:** All CodeCommit PR comments must start with "🤖 **Kiro Agent**" signature.

**Post comments for:**
- All critical issues (must fix)
- All important issues (should fix)
- Top 3-5 suggestions (nice to have)
- 1-2 praise comments (positive feedback)

**Guidelines:**
- Don't nitpick every minor style issue
- Don't post more than 10-15 comments (overwhelming)
- Balance criticism with praise

### Phase 5: Update PR Status

**Purpose:** Post a summary comment on the PR.

**If critical issues found:**

```markdown
🤖 **Kiro Agent**

## 🔴 Review Complete - Changes Required

I've reviewed this PR and found **3 critical issues** that must be addressed before merging:

1. SQL injection vulnerability in `users.py`
2. Missing authentication check in `api/admin.py`
3. Hardcoded API key in `config.py`

Please address these issues and I'll review again.

**Summary**:
- 🔴 Critical: 3
- 🟡 Important: 5
- 🔵 Suggestions: 7
- ✅ Good practices: 2
```

**If no critical issues:**

```markdown
🤖 **Kiro Agent**

## ✅ Review Complete - Looks Good!

I've reviewed this PR and found no critical issues. There are a few suggestions for improvement, but the code is ready to merge.

**Summary**:
- 🔴 Critical: 0
- 🟡 Important: 2
- 🔵 Suggestions: 4
- ✅ Good practices: 3

Great work on the error handling and test coverage! 🎉
```

**Note:** All PR comments must start with "🤖 **Kiro Agent**" signature.

### Phase 6: Update Jira Ticket (If Linked)

**Purpose:** Update the linked Jira ticket with review summary.

**Steps:**

1. Extract Jira ticket ID from PR:

From the PR title or description fetched in Phase 1, extract the Jira ticket ID.

Common patterns:
- PR title: `[DEV-123] Add authentication`
- PR title: `DEV-123: Add authentication`
- PR description: Contains link to `https://company.atlassian.net/browse/DEV-123`

If no Jira ticket is found, skip Phase 6.

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

3. Post review summary comment:

```javascript
jira_add_comment({
  issueKey: "<TICKET_ID>",
  comment: "🤖 *Kiro Agent*\n\n✅ PR Review Completed\n\nI've reviewed PR #<PR_ID> and posted <NUMBER> comments.\n\n**Review Summary:**\n- 🔴 Critical issues: <COUNT>\n- 🟡 Important issues: <COUNT>\n- 🔵 Suggestions: <COUNT>\n- ✅ Good practices: <COUNT>\n\nSee PR for detailed feedback: <PR_URL>"
})
```

**Comment format:**
- Must start with "🤖 *Kiro Agent*" signature
- Summary of review findings
- Count of issues by severity
- Link to the PR

**Why update Jira:**
- Keeps stakeholders informed that review is complete
- Maintains traceability in Jira
- Notifies watchers that PR has been reviewed
- Provides audit trail of AI-assisted reviews
- "kiro" label enables tracking and analytics

---

### ✅ Phase 6 Completion Checklist (VERIFY YOU DID ALL):

Before proceeding, verify you completed ALL of these steps:

- [ ] ✅ Extracted Jira ticket ID from PR (Step 1)
- [ ] ✅ Added "kiro" label to ticket (Step 2)
- [ ] ✅ Posted review summary comment (Step 3)

**If you didn't do ALL of the above, go back and complete the missing steps!**

## Review Best Practices

### Be Constructive
- Explain WHY something is an issue
- Provide specific recommendations
- Include code examples
- Link to documentation/resources
- Acknowledge good practices

### Be Balanced
- Mix criticism with praise
- Prioritize issues (critical vs. nice-to-have)
- Don't overwhelm with too many comments
- Focus on important issues

### Be Specific
- Reference exact file and line numbers
- Quote the problematic code
- Show the recommended fix
- Explain the impact

### Be Professional
- Focus on the code, not the person
- Use objective language
- Assume good intent
- Be respectful and encouraging

## Example Workflow Execution

**User input:**
```
"Review PR #42"
```

**Automatic execution:**
```
Phase 1: Fetching PR #42...
- Title: "Add OAuth2 authentication"
- Author: john.doe
- Files changed: 8
- Lines: +245, -12

Phase 2: Analyzing changes...
- Feature code: 5 files
- Tests: 2 files
- Documentation: 1 file

Phase 3: Performing code review...
- Checking code quality... ✓
- Checking functionality... ✓
- Checking tests... ⚠️ Missing edge case tests
- Checking security... 🔴 Found SQL injection vulnerability
- Checking performance... ✓
- Checking best practices... ✓
- Checking documentation... ✓

Phase 4: Posting review comments...
- Posted 1 critical issue
- Posted 2 important issues
- Posted 3 suggestions
- Posted 1 praise comment

Phase 5: Updating PR status...
- Posted summary comment

Phase 6: Updating Jira ticket DEV-123...
- Added "kiro" label
- Posted review summary comment

✅ Review complete! Posted 7 comments on PR #42. Jira ticket updated.
```

## Important Notes

### Autonomous Execution
- Execute all phases automatically without waiting for confirmation
- Proceed immediately through all phases
- Only stop if an error occurs or user explicitly asks to stop

### Comment Posting
- Always post review comments directly on the CodeCommit PR using AWS MCP
- Never just show feedback in chat - stakeholders need to see it on the PR
- Each review comment should be posted individually
- Include Kiro Agent signature in all comments

### Diff Direction
- Always use AWS MCP `get-differences` with correct parameter order
- `beforeCommitSpecifier` = destination (main) = old state
- `afterCommitSpecifier` = source (feature) = new state
- Never use `git_diff` or `git diff` commands

### Label Management
- Always fetch current labels before updating
- Use `fields.labels` array (replaces all labels)
- Include all existing labels plus "kiro" in the array
- Never use `additional_fields.update.labels` syntax

## Workflow Execution Summary

When a user asks to review a PR:

1. Fetch PR details and diff (Phase 1)
2. Analyze the changes (Phase 2)
3. Perform comprehensive code review (Phase 3)
4. Post review comments on the PR using AWS MCP (Phase 4)
5. Update PR status with summary comment (Phase 5)
6. Update Jira ticket if linked (Phase 6):
   - Extract Jira ticket ID from PR
   - Add "kiro" label to ticket
   - Post review summary comment

**Execute all phases automatically** without stopping for confirmation unless there's an error.

This is the expected behavior for a complete review workflow execution.
