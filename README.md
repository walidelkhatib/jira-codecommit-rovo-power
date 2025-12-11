# Jira-CodeCommit Development Power

A Kiro Power that enables autonomous AI-assisted development workflows integrating Jira ticket management with AWS CodeCommit pull requests. This power allows Kiro Agent to autonomously manage three complete development workflows:

### 1. Ticket Implementation Workflow
**Trigger:** `"Jira DEV-123"`

Automatically fetches Jira ticket, implements code changes, creates feature branch, commits, pushes to CodeCommit, creates pull request, and updates Jira ticket.

### 2. PR Review Workflow
**Trigger:** `"Review CodeCommit PR 5"`

Automatically fetches PR details, analyzes code changes, performs comprehensive code review, posts review comments directly on CodeCommit PR, and updates linked Jira ticket.

### 3. PR Patch Workflow
**Trigger:** `"Patch CodeCommit PR 5"`

Automatically fetches review comments, checks out PR branch, implements fixes, commits and pushes changes, posts update on PR, and updates linked Jira ticket.

## Demo

https://github.com/user-attachments/assets/cb5d4ba7-0840-4031-97f4-6234fc564eeb

### Key Features

- **Fully Autonomous Workflows** - If MCP tools are trusted, Kiro handles everything end-to-end

- **Automatic Jira Integration** - Adds "kiro" label and comments on all tickets for tracking and analytics

- **Direct CodeCommit Integration** - Creates PRs, posts review comments, and manages branches

- **Complete Audit Trail** - All actions tracked in both Jira and CodeCommit with clear timestamps

## Installation

1. **In Kiro**: Go to Powers → Add Custom Power → Import power from GitHub
2. **Add URL**: `https://github.com/walidelkhatib/jira-codecommit-power/tree/main/jira-codecommit`

## Prerequisites

### Before using this power, ensure you have:

- **Docker Desktop** - Running (required for Atlassian MCP server)
- **git-remote-codecommit** - Installed (`pip install git-remote-codecommit`)
- **AWS CLI** - Configured with a profile that has CodeCommit permissions
- **Jira API token** - Generated at https://id.atlassian.com/manage-profile/security/api-tokens

### Configure Trusted Commands (Optional)

For fully autonomous workflows, add these trusted commands in **Kiro Settings → Kiro Agent: Trusted Commands**:

```
git status *
git log *
git remote *
git push *
git pull *
git stash *
mkdir *
```

### Configure MCP Settings

The power's `mcp.json` includes `autoApprove` lists for reference, but you must manually add them to your user-level configuration.

Edit `~/.kiro/settings/mcp.json` and add your credentials:

**⚠️ Security Notice:** The `autoApprove` lists below are suggestions for fully autonomous workflows. Auto-approving `aws___call_aws` grants Kiro broad AWS access. **Strongly recommended:** Use a dedicated AWS profile with CodeCommit-only permissions. See the power documentation for detailed security recommendations.

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

**Important:** Update `JIRA_URL`, `JIRA_USERNAME`, `JIRA_API_TOKEN`, and `AWS_PROFILE` with your actual values.

## Usage Examples

All three workflows use a combination of:
- **Git MCP** for local operations (branch, commit, status)
- **AWS MCP** for CodeCommit operations (create PR, post comments, get diff)
- **Atlassian MCP** for Jira operations (fetch ticket, update, add comments)
- **git-remote-codecommit** for pushing code to CodeCommit

### Workflow 1: Implement Jira Ticket
```
"Jira MFLP-10"
```
Kiro will automatically fetch the ticket, implement the code, create a PR, and update Jira.

### Workflow 2: Review Pull Request
```
"Review CodeCommit PR 5"
```
Kiro will automatically review the code and post detailed feedback on the PR.

### Workflow 3: Patch Pull Request
```
"Patch CodeCommit PR 5"
```
Kiro will automatically implement fixes for review comments and update the PR.

## Resources

- [Kiro Powers Documentation](https://kiro.dev/powers/)
- [AWS CodeCommit Documentation](https://docs.aws.amazon.com/codecommit/)
- [AWS MCP Server](https://docs.aws.amazon.com/aws-mcp/latest/userguide/what-is-mcp-server.html)
- [Git MCP Server](https://github.com/modelcontextprotocol/servers/tree/main/src/git)
- [Atlassian MCP Server](https://github.com/sooperset/mcp-atlassian)

## Contributing

Feedback and contributions are welcome. Please open an issue on GitHub.

## License

MIT License - see [LICENSE](LICENSE) file for details.
