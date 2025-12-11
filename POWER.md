---
name: "jira-codecommit"
displayName: "Jira-CodeCommit Development"
description: "Automate development workflows from Jira tickets to CodeCommit pull requests"
author: "Walid El Khatib"
keywords: ["jira", "codecommit", "git", "pr", "pull request", "ticket", "issue", "patch", "review", "implement"]
---

# Jira-CodeCommit Development Power

## Overview

Build fully autonomous AI-assisted development workflows that integrate Jira ticket management with AWS CodeCommit pull requests. This power automates the complete development cycle from ticket analysis to pull request creation, code review, and patch implementation.

The power provides three automated workflows triggered by simple commands, handling everything from fetching tickets and implementing code to creating pull requests and posting review comments. All actions are tracked in both Jira and CodeCommit with complete audit trails.

**Key capabilities:**

- **Ticket Implementation**: Fetch Jira ticket → implement code → create branch → commit → push → create PR → update ticket
- **PR Review**: Fetch PR → analyze changes → perform code review → post comments on PR → update Jira
- **PR Patch**: Fetch review comments → checkout branch → implement fixes → commit → push → post update → update Jira

**Authentication**: Requires Jira API token, AWS credentials with CodeCommit permissions, and Docker Desktop for the Atlassian MCP server.

---

## Important: Automatic Workflow Execution

**This power executes complete workflows automatically when triggered.**

**Trigger phrases:** `"Jira DEV-123"`, `"Review CodeCommit PR 5"`, `"Patch CodeCommit PR 5"`

**Expected behavior:** Immediate execution of the complete workflow without confirmation prompts.

**Do NOT ask:**
- "What would you like me to do with this ticket?"
- "Should I fetch the ticket details?"
- "Let me know what you'd like to do"

**Instead:** Immediately begin executing the appropriate workflow based on the trigger phrase.

## Available MCP Servers

### atlassian

**Connection:** Docker-based MCP server
**Repository:** https://github.com/sooperset/mcp-atlassian/
**Capabilities:** Jira operations (get issues, update tickets, add comments, transition workflows, search)
**Requirements:** Docker Desktop must be running

### git

**Connection:** Local MCP server via uvx
**Capabilities:** Git operations (branch, commit, stage, diff, log, status)
**Note:** Push operations use `git push` CLI with git-remote-codecommit

### aws-mcp

**Connection:** Remote AWS-managed MCP server
**Capabilities:** 15,000+ AWS APIs including CodeCommit operations (create PRs, post comments, get diffs, manage branches)
**Type:** Managed service running on AWS infrastructure

## Three Automated Workflows

### Workflow 1: Ticket Implementation
**Trigger:** `"Jira DEV-123"` or `"Implement Jira DEV-123"`

**What happens:**
1. Fetches and analyzes Jira ticket
2. Adds "kiro" label to ticket for tracking
3. Creates feature branch from main
4. Implements code changes following project standards
5. Commits changes with conventional commit messages
6. Pushes branch to CodeCommit
7. Creates pull request with ticket details
8. Updates Jira ticket with PR link and comment

**Example:** `"Jira MFLP-10"`

### Workflow 2: PR Review
**Trigger:** `"Review CodeCommit PR 5"`

**What happens:**
1. Fetches PR details and code diff
2. Performs comprehensive code review (security, quality, testing, performance)
3. Posts review comments directly on CodeCommit PR
4. Posts summary comment on PR
5. Extracts linked Jira ticket from PR
6. Adds "kiro" label to Jira ticket
7. Posts review summary on Jira ticket

**Example:** `"Review CodeCommit PR 5"`

### Workflow 3: PR Patch
**Trigger:** `"Patch CodeCommit PR 5"`

**What happens:**
1. Fetches PR review comments
2. Checks out PR branch
3. Implements fixes for review feedback
4. Commits and pushes changes
5. Posts update comment on PR
6. Adds "kiro" label to Jira ticket
7. Posts update on Jira ticket

**Example:** `"Patch CodeCommit PR 5"`

## Best Practices

### Workflow Execution

**Always execute complete workflows** when triggered by the standard phrases above. Each workflow is designed to run end-to-end without requiring confirmation at each step. This ensures consistency and completeness.

**Add "kiro" label immediately** after fetching any Jira ticket, before any analysis or implementation work. This provides early tracking of AI-assisted work and ensures the label is present even if the workflow fails mid-way.

**Use MCP tools instead of CLI commands** for git operations (except `git push`). The Git MCP server is configured with `GIT_PAGER=cat` to prevent pager issues. Use `git_status`, `git_checkout`, `git_log`, `git_branch` instead of shell commands.

### Jira Label Management

**The Atlassian MCP server uses `fields.labels` which replaces all labels.** Always fetch current labels first, then include them in the update array along with "kiro":

```javascript
// Step 1: Get current labels
const issue = jira_get_issue({ issue_key: "DEV-123", fields: "labels" });

// Step 2: Add "kiro" to existing labels
jira_update_issue({
  issue_key: "DEV-123",
  fields: {
    labels: [...issue.fields.labels, "kiro"]
  }
});
```

**Never use** `additional_fields.update.labels[{"add": "kiro"}]` - this syntax fails with the Atlassian MCP server.

### PR Review Comments

**Always post review comments directly on the CodeCommit PR** using AWS MCP `post-comment-for-pull-request`. Never just show feedback in chat - stakeholders need to see comments on the PR itself.

**Use the correct diff direction** with AWS MCP `get-differences`:
- `beforeCommitSpecifier`: destination commit (main branch - old state)
- `afterCommitSpecifier`: source commit (feature branch - new state)

This shows what the feature branch adds to main. Never use `git_diff` or `git diff` as they can show changes in the wrong direction.

### PR Descriptions

**Keep PR descriptions simple and single-line** to avoid AWS MCP timeouts. Complex multi-line descriptions with special characters can cause the `create-pull-request` operation to timeout (300s).

✅ **Good:**
```
"Implements OAuth2 authentication with JWT tokens. Resolves DEV-123"
"Adds search and filter functionality for messages page. Resolves DEV-456"
```

❌ **Avoid:**
```
"## Summary\n\nImplements...\n\n## Changes\n\n- Added...\n\n## Acceptance Criteria\n\n- ✅ Met..."
```

### Repository and Region Information

**Extract repository name from git remote URL** - it's already configured in the local repository. Never search for repositories or try different regions.

**The AWS MCP uses the region from its configuration** - don't specify region in API calls or search for repositories in different regions.

If pull request creation fails due to region mismatch, report the error to the user and ask them to verify their configuration. Don't attempt to debug by searching other regions.

### Comment Signatures

**All comments must include the Kiro Agent signature** for transparency:

**Jira comments:** Start with `🤖 *Kiro Agent*` (italic)
**CodeCommit comments:** Start with `🤖 **Kiro Agent**` (bold)

This distinguishes AI comments from human comments and provides transparency about AI-assisted work.

## Common Workflows

### Workflow 1: Implement Jira Ticket

```
User: "Jira DEV-123"

Execution:
→ Fetch ticket DEV-123
→ Add "kiro" label to ticket
→ Check git status and ensure clean state
→ Create branch feature/DEV-123-add-authentication
→ Implement code changes
→ Commit with message: "feat(auth): Add OAuth2 authentication"
→ Push to CodeCommit
→ Create PR with simple description
→ Update Jira ticket with PR link

Result: Complete implementation from ticket to PR
```

### Workflow 2: Review Pull Request

```
User: "Review CodeCommit PR 5"

Execution:
→ Fetch PR details and diff
→ Analyze code for security, quality, testing, performance
→ Post review comments on CodeCommit PR (critical, important, suggestions)
→ Post summary comment on PR
→ Extract Jira ticket from PR
→ Add "kiro" label to ticket
→ Post review summary on Jira ticket

Result: Complete code review with feedback on PR and Jira
```

### Workflow 3: Patch Pull Request

```
User: "Patch CodeCommit PR 5"

Execution:
→ Fetch PR review comments
→ Identify critical and important issues
→ Checkout PR branch
→ Implement fixes
→ Commit with message: "fix: address PR review comments"
→ Push to CodeCommit
→ Post update comment on PR
→ Add "kiro" label to Jira ticket
→ Post update on Jira ticket

Result: Review feedback addressed with updates on PR and Jira
```

## Best Practices Summary

### ✅ Do:

- **Execute complete workflows** when triggered by standard phrases
- **Add "kiro" label immediately** after fetching Jira tickets
- **Use MCP tools** for git operations (except `git push`)
- **Post review comments on PRs** using AWS MCP, not just in chat
- **Use simple PR descriptions** to avoid timeouts
- **Extract repository name** from git remote URL
- **Include Kiro Agent signature** in all comments
- **Preserve existing labels** when adding "kiro" label
- **Use correct diff direction** with AWS MCP `get-differences`
- **Handle errors gracefully** and report to user

### ❌ Don't:

- **Stop workflows mid-way** - execute completely unless there's an error
- **Skip adding "kiro" label** - it's required for tracking
- **Use shell commands** for git operations that have MCP tools
- **Show review feedback only in chat** - must post to PR
- **Use complex PR descriptions** - keep them simple and single-line
- **Search for repositories** - extract from git remote URL
- **Search for regions** - use AWS MCP configured region
- **Use `additional_fields.update.labels`** - use `fields.labels` instead
- **Use `git_diff` for PR reviews** - use AWS MCP `get-differences`
- **Post comments without signature** - always include Kiro Agent signature

## Configuration

**Authentication Required:**
- Jira API token (generated at https://id.atlassian.com/manage-profile/security/api-tokens)
- AWS credentials with CodeCommit permissions
- Docker Desktop running (for Atlassian MCP)

**Setup Steps:**

1. Install prerequisites:
   - Docker Desktop (must be running)
   - git-remote-codecommit: `pip install git-remote-codecommit`
   - AWS CLI configured: `aws configure`

2. Generate Jira API token at https://id.atlassian.com/manage-profile/security/api-tokens

3. Configure MCP servers in `~/.kiro/settings/mcp.json`:

```json
{
  "powers": {
    "mcpServers": {
      "power-jira-codecommit-atlassian": {
        "env": {
          "JIRA_URL": "https://your-company.atlassian.net",
          "JIRA_USERNAME": "your.email@company.com",
          "JIRA_API_TOKEN": "your_api_token_here"
        },
        "autoApprove": [
          "jira_get_issue",
          "jira_update_issue",
          "jira_add_comment",
          "jira_transition_issue",
          "jira_get_transitions",
          "jira_search_issues"
        ]
      },
      "power-jira-codecommit-git": {
        "autoApprove": [
          "git_status",
          "git_create_branch",
          "git_checkout",
          "git_add",
          "git_commit",
          "git_diff",
          "git_diff_unstaged",
          "git_diff_staged",
          "git_log",
          "git_branch"
        ]
      },
      "power-jira-codecommit-aws-mcp": {
        "env": {
          "AWS_PROFILE": "your-profile-name"
        },
        "autoApprove": [
          "aws___call_aws",
          "aws___search_documentation",
          "aws___read_documentation"
        ]
      }
    }
  }
}
```

4. (Optional) Configure trusted shell commands in Kiro Settings for fully autonomous workflows:
   - `git status *`
   - `git log *`
   - `git remote *`
   - `git push *`
   - `git pull *`
   - `git stash *`
   - `mkdir *`

## MCP Config Placeholders
Before using this power, replace the following placeholders in `~/.kiro/settings/mcp.json` with your actual values:

- **`https://your-company.atlassian.net`**: Your Jira instance URL.
  - **How to get it:** Use your company's Jira URL (e.g., `https://mycompany.atlassian.net`)

- **`your.email@company.com`**: Your Jira username/email.
  - **How to get it:** Use the email address associated with your Jira account

- **`your_api_token_here`**: Your Jira API token.
  - **How to get it:**
    1. Go to https://id.atlassian.com/manage-profile/security/api-tokens
    2. Click "Create API token"
    3. Copy the generated token and paste it here

- **`your-profile-name`**: Your AWS CLI profile name.
  - **How to get it:** Use `aws configure list-profiles` to see available profiles, or use `default`

## Troubleshooting

### Pull Request Creation Timeout

**Cause:** PR description is too complex with multi-line strings or special characters
**Solution:**

1. Use simple, single-line descriptions (max ~200 characters)
2. Template: `"Implements [feature] with [key changes]. Resolves [TICKET-ID]"`
3. Avoid `\n`, special characters (✅, ❌), and markdown formatting

If timeout persists, it may be a region mismatch - report error to user and ask them to verify AWS MCP region matches CodeCommit repository region.

### Pull Request Creation Fails - Repository Not Found

**Cause:** Region mismatch between AWS MCP configuration and CodeCommit repository
**Solution:**

1. Check AWS MCP configuration region in `mcp.json`
2. Verify CodeCommit repository region in AWS Console
3. Update MCP configuration to match repository region
4. Never search for repositories in different regions

### Review Comments Not Posted

**Cause:** Not using AWS MCP `post-comment-for-pull-request` tool
**Solution:**

1. Always use AWS MCP to post comments on PRs
2. Never just show feedback in chat
3. Each review comment should be posted individually
4. Include Kiro Agent signature in all comments

### Jira Label Update Fails

**Cause:** Using incorrect syntax for label updates
**Solution:**

1. Always fetch current labels first: `jira_get_issue` with `fields="labels"`
2. Use `fields.labels` array with all existing labels plus "kiro"
3. Never use `additional_fields.update.labels[{"add": "kiro"}]`

### Git Pager Issues

**Cause:** Using shell commands like `git log` or `git branch` that trigger pager
**Solution:**

1. Use Git MCP tools instead: `git_log`, `git_branch`, `git_status`, `git_checkout`
2. Git MCP server is configured with `GIT_PAGER=cat` to prevent pager issues
3. Only use `git push` shell command (git-remote-codecommit requirement)

### Docker Connection Failed

**Cause:** Docker Desktop not running (required for Atlassian MCP)
**Solution:**

1. Start Docker Desktop
2. Verify Docker is running: `docker ps`
3. Restart Kiro if needed to reconnect MCP server

## Tips

1. **Start with simple tickets** - Test the workflow with straightforward implementations first
2. **Use autoApprove lists** - Reduces manual approvals for fully autonomous workflows
3. **Monitor both Jira and CodeCommit** - Verify comments and updates appear correctly
4. **Keep PR descriptions simple** - Prevents timeout issues with AWS MCP
5. **Let workflows complete** - Don't interrupt unless there's an error
6. **Check "kiro" labels** - Verify labels are being added to track AI-assisted work
7. **Review steering files** - Detailed workflow instructions are in `steering/` directory
8. **Test in sandbox first** - Use test Jira projects and CodeCommit repos initially
9. **Use git-remote-codecommit** - Simplifies CodeCommit authentication via AWS credentials
10. **Follow conventional commits** - Maintains clean git history

## Resources

- [Kiro Powers Documentation](https://kiro.dev/powers/)
- [AWS CodeCommit Documentation](https://docs.aws.amazon.com/codecommit/)
- [AWS MCP Server](https://docs.aws.amazon.com/aws-mcp/latest/userguide/what-is-mcp-server.html)
- [Git MCP Server](https://github.com/modelcontextprotocol/servers/tree/main/src/git)
- [Atlassian MCP Server](https://github.com/sooperset/mcp-atlassian)
- [git-remote-codecommit](https://docs.aws.amazon.com/codecommit/latest/userguide/setting-up-git-remote-codecommit.html)

---

**License:** MIT
