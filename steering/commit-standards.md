---
inclusion: always
---

# Commit Message Standards

## Conventional Commits Format

All commit messages MUST follow this format:

```
<type>(<scope>): <subject>

<body>

<footer>
```

### Type (Required)

| Type | When to Use |
|------|-------------|
| **feat** | New feature for the user |
| **fix** | Bug fix for the user |
| **docs** | Documentation only changes |
| **style** | Code style changes (formatting, no logic change) |
| **refactor** | Code change that neither fixes a bug nor adds a feature |
| **perf** | Performance improvement |
| **test** | Adding or updating tests |
| **chore** | Changes to build process, dependencies, or tooling |

### Scope (Optional but Recommended)

Specifies what part of the codebase is affected:
- Component: `auth`, `api`, `ui`, `database`
- Module: `user-service`, `payment-processor`
- Feature: `login`, `checkout`, `dashboard`

### Subject (Required)

Brief description of the change:

**Rules:**
- Use imperative mood: "add" not "added" or "adds"
- Don't capitalize first letter
- No period at the end
- Maximum 50 characters
- Be specific and descriptive

**Why imperative?** Matches git's convention ("Merge branch", "Revert commit")

### Body (Optional but Recommended)

Detailed explanation of the change:
- What changed and why (not how - code shows that)
- Context and motivation
- Comparison with previous behavior
- Wrap at 72 characters

### Footer (Optional)

Metadata about the change:
- `Refs: TICKET-ID` - References a Jira ticket
- `Closes: TICKET-ID` - Closes a Jira ticket
- `BREAKING CHANGE:` - Describes breaking changes

---

## Examples

### Good Examples

**Simple Feature:**
```
feat(auth): add OAuth2 authentication

Implements OAuth2 flow with Google and GitHub providers.
Users can now sign in using their existing accounts.

Refs: DEV-123
```

**Bug Fix:**
```
fix(api): handle null response from user service

The user service occasionally returns null when database
is under heavy load. Added null check and return 503 with
retry-after header when this occurs.

Closes: BUG-456
```

**Refactoring:**
```
refactor(payment): extract validation to separate module

Moved payment validation from PaymentController to
PaymentValidator class to improve testability.

No functional changes - all tests still pass.

Refs: TECH-234
```

**Breaking Change:**
```
feat(api): change user endpoint response format

Changed /api/users response from array to paginated object
for better performance with large datasets.

BREAKING CHANGE: API clients must update to handle new
paginated response format. See migration guide in docs.

Refs: DEV-789
```

### Bad Examples (Don't Do This)

**Too Vague:**
```
fix: bug fix
```
❌ No scope, vague subject, no context

**Wrong Tense:**
```
feat(api): Added new endpoint
```
❌ Past tense, capitalized

**Too Long:**
```
feat(auth): implement OAuth2 with Google and GitHub including token refresh and error handling
```
❌ Subject exceeds 50 characters

**Multiple Changes:**
```
feat(api): add endpoint and fix bug and update docs
```
❌ Should be split into separate commits

---

## Ticket References

### Format

Always reference Jira tickets in the footer:

```
Refs: TICKET-ID        # Work related to ticket
Closes: TICKET-ID      # Work that completes ticket
```

Multiple tickets:
```
Refs: DEV-123, DEV-124
```

### When to Use Refs vs Closes

- **Refs**: Work related to ticket but doesn't complete it
- **Closes**: Work that completes the ticket

**Example workflow:**
```
feat(auth): add login form
Refs: DEV-123

feat(auth): add login API
Refs: DEV-123

feat(auth): add session management
Closes: DEV-123
```

---

## Commit Best Practices

### Atomic Commits

Each commit should be one logical change:

**✅ Good:**
```
feat(auth): add login form
feat(auth): add login API endpoint
feat(auth): add session management
```

**❌ Bad:**
```
feat(auth): add complete authentication system
```

### When to Commit

**Do commit:**
- After completing a logical unit of work
- When tests pass
- Before switching tasks

**Don't commit:**
- Broken code (unless marked WIP)
- Every single line change
- Generated files (unless necessary)
- Secrets or credentials

---

## Quick Reference

### Commit Message Template

```
<type>(<scope>): <subject>
# |<----  Max 50 characters  ---->|

<body>
# |<----  Wrap at 72 characters  ---->|

Refs: TICKET-ID
```

### Subject Line Checklist

- [ ] Uses imperative mood ("add" not "added")
- [ ] First letter not capitalized
- [ ] No period at the end
- [ ] 50 characters or less
- [ ] Specific and descriptive

### Common Types

- `feat` - New feature
- `fix` - Bug fix
- `docs` - Documentation
- `refactor` - Code restructure
- `test` - Tests
- `chore` - Maintenance

---

## Summary

**Good commit messages:**
- Follow conventional commits format
- Use imperative mood
- Explain what and why (not how)
- Reference Jira tickets
- Are atomic (one logical change)
- Provide context in the body

**Remember:** Write commit messages for your future self and teammates who need to understand the change without additional context.
