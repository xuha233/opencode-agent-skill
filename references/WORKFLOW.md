# OpenCode Workflow Guide

This document defines workflow patterns using OpenCode CLI.

## Core Workflow Pattern

### 1. Context Gathering

```bash
# Read-only operations first
opencode run "Summarize the codebase structure"
opencode run -c "What are the key patterns?"

# Attach files for context
opencode run -f src/auth.ts -f package.json "Review auth module"
```

### 2. Planning

```bash
# Plan with assumptions
opencode run "Create a detailed plan for implementing password reset:
- Requirements: Email links, token expiration
- Constraints: Must work with existing auth
- Tech stack: Current stack (see files)"

# Get explicit approval before implementation
opencode run -c "What are the assumptions in this plan?

I'm ready to implement. Type 'APPROVE' to proceed."
```

### 3. Implementation

```bash
# After approval, execute
opencode run "Implement the plan for password reset"

# Continue with follow-up
opencode run -c "Add email validation"
opencode run -c "Add token expiration logic"
opencode run -c "Add unit tests"
```

### 4. Testing

```bash
# Run tests
opencode run -c "Run the test suite and report results"

# Fix failing tests
opencode run -c "Tests failed. Fix the issues:

Test errors:
- password-reset.spec.ts:45: Missing email validation
- password-reset.spec.ts:78: Token not expired properly"
```

### 5. Code Review

```bash
# Self-review before PR
opencode run -c "Review my password reset implementation for:

1. **Security**:
   - Are tokens generated securely?
   - Is email link validation correct?
   - Are tokens expired properly?

2. **Code Quality**:
   - Is it readable and maintainable?
   - Are error cases handled?

3. **Edge Cases**:
   - What if email delivery fails?
   - What if user clicks expired link?
   - What if user requests multiple resets?

4. **Performance**:
   - Are database queries optimized?
   - Is caching appropriate?"
```

---

## Multi-Phase Example

```bash
# === PHASE 1: UNDERSTAND ===
opencode run "Explain how authentication works:
- Session management
- JWT token flow
- Role-based access control"

# === PHASE 2: PLAN ===
opencode run -c "Create a plan for adding password reset:
1. Database schema: Add password_reset_tokens table
2. Backend API: POST /api/auth/reset-request
3. Backend API: POST /api/auth/reset-confirm
4. Frontend: Reset request form
5. Frontend: Reset confirm form
6. Email service: Send reset link

What are the risks and dependencies?"

# === PHASE 3: IMPLEMENT ===
opencode run -c "Implement phase 1: Database schema
- Create migration file
- Add token generation (UUID + expiry)
- Add unique constraint on user_id"

opencode run -c "Implement phase 2: Backend API
- reset-request endpoint
- reset-confirm endpoint
- Service layer for business logic"

opencode run -c "Implement phase 3: Frontend
- Request form component
- Confirm form component
- API integration"

opencode run -c "Implement phase 4: Email service
- Send reset link email
- Handle email delivery errors"

# === PHASE 4: TEST ===
opencode run -c "Write comprehensive tests:
- Unit tests for token generation
- Integration tests for API endpoints
- E2E tests for user flow"

opencode run -c "Run tests and fix failures:
```bash
npm test:unit
npm test:integration
 npm test:e2e
```"

# === PHASE 5: REVIEW ===
opencode run -c "Review for:
1. Security (token generation, validation, expiry)
2. Code quality (readability, error handling)
3. Edge cases (email failures, expired links, multiple requests)
4. Performance (DB queries, caching)"

# === PHASE 6: COMMIT ===
git checkout -b feat/password-reset
git add .
git commit -m "feat: add password reset functionality"
gh pr create -t "feat: add password reset" -b "## What
Implemented password reset feature with email links.

## Why
Users need to reset passwords without admin help.

## Tests
```bash
npm test
```

## AI Assistance
Generated with OpenCode Agent
Session: <session-id>"
```

---

## Session Resume Pattern

**Always resume sessions to maintain context:**

```bash
# Start with initial request
opencode run "Implement feature X"

# Continue with context preserved
opencode run --continue "Refactor function for performance"
opencode run -c "Add error handling for edge cases"

# Continue again
opencode run --continue "Run tests and fix failures"

# Use specific session ID
opencode run -s abc123 --model claude-sonnet-4 "Review with better model"
```

**When to use `--continue` vs `--session`**:

| Scenario | Command |
|----------|---------|
| Continue where I left off | `--continue` |
| Resume specific older session | `--session <id>` |
| Fork before risky change | `--continue --fork` |

---

## Workflow Commands Reference

| Phase | Command |
|-------|---------|
| Exploratory | `opencode run "Explain how X works"` |
| Planning | `opencode run "Create plan for X"` |
| Approval | `opencode run -c "Type APPROVE to implement"` |
| Implementation | `opencode run "Implement X"` |
| Follow-up | `opencode run --continue "<prompt>"` |
| Testing | `opencode run -c "Run tests"` |
| Review | `opencode run -c "Review my changes"` |
| Fixing | `opencode run -c "Fix the review findings"` |
| Re-review | `opencode run -c "Re-review after fixes"` |

---

## Context Strategies

### 1. Ask OpenCode to explain first

```bash
opencode run "Summarize the codebase:
- Core modules
- Data flow
- Architectural patterns"
```

### 2. Provide specific files

```bash
# Single file review
opencode run -f src/auth.ts "Review authentication implementation"

# Multiple files
opencode run -f src/auth.ts -f src/session.ts "Review auth + session"

# Directory
opencode run -f src/api/ "Review API layer architecture"
```

### 3. Reference existing patterns

```bash
opencode run "Implement user profile update.
Follow the pattern used in:
- src/api/users/profile.ts (PUT /users/:id/profile)
- src/services/user-service.ts (updateProfile method)
"
```

### 4. Use examples

```bash
opencode run "Implement cache layer.
Similar to how productController.ts uses cache:
1. Check cache first
2. If miss, query DB
3. Store result
4. Set TTL

Apply to userController.ts getUser()"
```

---

## Error Recovery

### Fixing OpenCode mistakes

```bash
# Ask it to fix
opencode run --continue "The tests failed:
```
npm test
``
**Output**:
FAIL src/auth/password-reset.spec.ts
  - should send email with reset link
    Expected: email sent
    Received: email not sent

Fix the implementation."
```

### Roll back and retry

```bash
# Reset to last commit
git reset --hard HEAD

# Retry with different approach
opencode run --continue "Retry with a different implementation:
- Use email queue instead of direct send
- Handle delivery failures gracefully"
```

### Fork before risky change

```bash
# Fork session
opencode run --continue --fork "Try refactoring with function composition"

# If it fails, go back to original
opencode run -s <original-id> "Keep the original implementation"
```

---

## Git Workflow

### Branch Names

```bash
# Feature
git checkout -b feat/password-reset
git checkout -b feat/user-profile

# Bugfix
git checkout -b bugfix/login-crash

# Refactor
git checkout -b refactor/dao-layer

# Docs
git checkout -b docs/api-reference
```

### Commit Messages

```
type(scope): imperative summary

Optional detailed description

Footer: fixes #issue
```

**Types**:
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Refactoring only
- `docs` - Documentation
- `test` - Tests
- `chore` - Maintenance

**Examples**:
```
feat(auth): add password reset functionality

Implements email-based password reset with token expiration.

fixes #123
```

### Pull Request Titles

```
type(scope): imperative summary
```

**Examples**:
```
feat(auth): add password reset functionality
fix(api): handle null response from user endpoint
refactor(ui): simplify component hierarchy
docs(readme): update installation instructions
```

### PR Body Template

```markdown
## What
Brief description of changes (1-2 sentences).

## Why
Reason for the change (business value, bug fix, requirement).

## Tests
How to test this change:
```bash
npm run test:unit
npm run test:e2e
```

## AI Assistance
Generated with OpenCode Agent
Session: <session-id>
```

### GitHub Commands

```bash
# Create PR
gh pr create -t "feat: add feature X" -b "## What\n..."

# Checkout PR
gh pr checkout 123

# Review PR
gh pr view 123

# Approve PR
gh pr review --approve

# Request changes
gh pr review --request-changes -b "Issues found..."

# Merge PR
gh pr merge --merge
```

---

## Review Workflow

### Code Review Checklist

**Review in this order**:

1. **Architecture** (5 min)
   - Is design appropriate?
   - Dependencies correct?
   - Code in right module?

2. **Code Quality** (10 min)
   - Readable and maintainable?
   - Obvious bugs?
   - Error handling complete?

3. **Security** (5 min)
   - Input validation?
   - SQL injection risks?
   - XSS prevention?

4. **Performance** (5 min)
   - DB queries optimized?
   - Caching appropriate?
   - N+1 queries?

5. **Testing** (5 min)
   - New code covered?
   - Edge cases tested?
   - Tests clear and meaningful?

### Interactive Review Flow

```bash
# Big change: section by section
opencode run -c "Review just the password reset service:
- src/services/password-reset-service.ts

Don't review the frontend yet."

# Wait for feedback...

opencode run -c "Now review the API endpoints:
- src/api/auth/reset-request.ts
- src/api/auth/reset-confirm.ts"

# Wait for feedback...

opencode run -c "Review frontend components:
- src/components/PasswordResetRequest.tsx
- src/components/PasswordResetConfirm.tsx"
```

**Small change**: One focused question
```bash
opencode run -c "Review this function for security risks:
```
typescript
function generateToken() {
  return Math.random().toString(36);
}
```"
```

### Fix-Review Cycle

```bash
# Review
opencode run -c "Review the PR"

# Fix issues
opencode run -c "Address review findings:
1. Fix SQL injection risk
2. Add input validation
3. Handle edge case"

# Re-review
opencode run -c "Re-review after fixes"

# Approve
gh pr review --approve
```

---

## Complete Example Projects

### 1. New Feature (User Roles)

```bash
# Phase 1: Explore
opencode run -f src/db/schema.sql -f src/auth/ "How do permissions work?"

# Phase 2: Plan
opencode run -c "Plan for adding role-based permissions:
1. Add roles table to DB
2. Assign role to user (default: 'user')
3. Middleware to check role
4. Update API endpoints with role checks"

# Phase 3: Implement
opencode run -c "Implement phase 1: Schema + migration"
opencode run -c -s <latest> "Implement phase 2: Role assignment"
opencode run -c -s <latest> "Implement phase 3: Middleware"
opencode run -c -s <latest> "Implement phase 4: API protection"

# Phase 4: Test
opencode run -c -s <latest> "Add tests:
- Admin can delete users
- User cannot delete users
- Guest cannot delete anything"

opencode run -c -s <latest> "Run tests: npm test"

# Phase 5: Review
opencode run -c -s <latest> "Review for security holes"

# Phase 6: Commit
git checkout -b feat/role-based-auth
git add .
git commit -m "feat: add role-based access control"
gh pr create -t "feat: add role-based access control" -b "..."
```

### 2. Bug Fix (Login Crash)

```bash
# Phase 1: Investigate
opencode run -f logs/error.log -c "Why is login crashing at line 45?"

# Phase 2: Root cause
opencode run -c "Is it:
1. DB connection issue?
2. Missing password field?
3. JWT token generation failure?"

# Phase 3: Fix
opencode run -c "Fix the bug: Add null check on user.password"

# Phase 4: Test
opencode run -c "Add regression test:
```typescript
it('should handle user with null password', async () => {
  const user = { email: 'test@example.com', password: null };
  await expect(login(user)).rejects.toThrow('Invalid credentials');
});
```"

opencode run -c -s <latest> "Run tests: npm test"

# Phase 5: Test in production
opencode run -c "Steps to test in prod:
1. Deploy to staging
2. Try login with valid credentials
3. Try login with invalid credentials
4. Try login with missing password field"

# Phase 6: Commit
git commit -m "fix: handle null password in login"
```

---

## Advanced Patterns

### Parallel Work (Server Mode)

```bash
# Terminal 1: Start server
opencode serve --port 4096

# Terminal 2: Work on feature A
opencode run --attach http://localhost:4096 "Implement A"

# Terminal 3: Work on feature B
opencode run --attach http://localhost:4096 "Implement B"

# Terminal 4: Run tests
opencode run --attach http://localhost:4096 "Run tests"
```

### Forking for A/B Testing

```bash
# Implementation A
opencode run "Implement authentication with JWT"

# Fork for Implementation B (OAuth)
opencode run --continue --fork "Replace JWT entirely with OAuth2 flow"

# Compare both
opencode run -s <jwt-sess-id> "List pros/cons of JWT approach"
opencode run -s <oauth-sess-id> "List pros/cons of OAuth approach"

# Choose one
opencode run -s <chosen-id> "Proceed with this implementation"
```

### Historical Exploration

```bash
# List old sessions
opencode session list --max-count 50 --format json | jq -r '.[] | select(.title | contains("auth")) | "\(.updated | . / 1000 | strftime(\"%Y-%m-%d\")) \(.title) \(.id)"'

# Export relevant session
opencode export abc123 > old-implementation.json

# Import into new session
opencode import old-implementation.json

# Analyze old approach
opencode run --session <imported-id> "What did this old approach do? How does it differ from current?"
```

---

## Summary

**Core principles**:
1. **Start with context** → Plan → Approve → Implement → Test → Review → Commit
2. **Always resume sessions** → Maintain conversation continuity
3. **Use file attachments** → Give OpenCode full context
4. **Fork risky changes** → Don't break original session
5. **Review systematically** → Architecture → Quality → Security → Performance → Tests

**Common patterns**:
- Continuous conversation with `--continue`
- Fork with `--continue --fork`
- Server mode for persistent connections
- JSON output for scripting
- Export/import for session backup
