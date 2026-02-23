# OpenCode Agent Coding Standards

This document defines coding standards for OpenCode Agent.

## Core Principles

### KISS (Keep It Simple, Stupid)
- Prefer simple, readable code over clever code
- Avoid unnecessary abstractions
- Comment complex logic when unavoidable

### YAGNI (You Aren't Gonna Need It)
- Implement what's needed now, not what might be needed later
- Avoid speculative features
- Delete unused code immediately

### DRY (Don't Repeat Yourself)
- Three-strikes rule: If code appears 3+ times, abstract it
- Prefer composition over inheritance
- Don't abstract prematurely

### SRP (Single Responsibility Principle)
- Each function/class should do one thing well
- Keep functions focused and small (<50 lines preferred)
- Module-level cohesiveness

## Code Quality

### Readability
- Use descriptive names
- Avoid magic numbers (use named constants)
- Prefer explicit over implicit

### Error Handling
- Always handle errors explicitly
- Never fail silently
- Use appropriate error types

### Type Safety
- Avoid `any` when possible
- Prefer explicit interfaces
- Use type inference wisely

### Testing
- Test before deploying
- Write tests for bugs (reproduce, fix, verify)
- Keep tests simple and focused

## Git Workflow

### Branch Names
- `feat/short-description` - New features
- `bugfix/short-description` - Bug fixes
- `refactor/short-description` - Refactoring
- `docs/short-description` - Documentation

### Commit Messages
```
type(scope): imperative summary

Optional detailed description

Footer: fixes #issue
```

Types:
- `feat` - New feature
- `fix` - Bug fix
- `refactor` - Refactoring
- `docs` - Documentation
- `test` - Tests
- `chore` - Maintenance

### Pull Request Titles
```
type(scope): imperative summary
```

Examples:
- `feat(auth): add password reset functionality`
- `fix(api): handle null response from user endpoint`
- `refactor(ui): simplify component hierarchy`

## GitHub Hygiene

### PR Body Structure
```markdown
## What
Brief description of changes

## Why
Reason for the change (business value, bug fix, etc.)

## Tests
How to test this change
```bash
npm test
# or
curl -X POST https://api.example.com/test
```

## AI Assistance
Generated with OpenCode Agent
Session: <session-id or URL>
```

### Issue Titles
- **Feature**: `feat: <capability>`
- **Bug**: `bug: <symptom> when <condition>`
- **Tracking**: `TODO: <cleanup> after <dependency>`

## Code Review Checklist

### Architecture
- Is the design appropriate for the problem?
- Are dependencies correctly managed?
- Is the code in the right file/module?

### Code Quality
- Is it readable and maintainable?
- Are there obvious bugs or issues?
- Are error cases handled?

### Testing
- Do tests cover the new code?
- Are tests clear and meaningful?
- Are edge cases tested?

### Performance
- Are there performance concerns?
- Are database queries optimized?
- Is caching appropriate?

## Specific Rules

### File Organization
- One major export per file
- Group related functionality
- Use barrel exports for public APIs

### Naming Conventions
- Files: `kebab-case.ts`
- Classes: `PascalCase`
- Functions/variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`

### Comments
- Explain "why", not "what"
- Document non-obvious behavior
- Keep comments up to date

### Imports
- Order: node → external → internal
- Use absolute imports for shared modules
- Avoid circular dependencies

## Code Size Guidelines

- Functions: < 50 lines (ideal), < 100 lines (max)
- Files: < 500 lines (ideal), < 1000 lines (max)
- Modules: < 2000 lines (split if larger)

## Patterns to Avoid

- ❌ Deeply nested conditionals (max 3 levels)
- ❌ Long parameter lists (> 4 parameters)
- ❌ God functions/classes (too many responsibilities)
- ❌ Magic numbers and strings
- ❌ Copy-paste code

## Patterns to Prefer

- ✅ Early returns (guard clauses)
- ✅ Composition over inheritance
- ✅ Pure functions (no side effects)
- ✅ Explicit error handling
- ✅ Small, focused units

## Documentation

### Code Comments
```typescript
/**
 * Calculate the total price including tax.
 * @param basePrice - The base price before tax
 * @param taxRate - Tax rate as decimal (0.1 = 10%)
 * @returns Total price including tax
 */
function calculateTotal(basePrice: number, taxRate: number): number {
  return basePrice * (1 + taxRate);
}
```

### README Requirements
- Installation instructions
- Usage examples
- API documentation
- Contributing guidelines

## Security Guidelines

- Never commit secrets (use .env or secrets manager)
- Validate all user inputs
- Use parameterized queries
- Sanitize outputs (XSS prevention)
- Follow the principle of least privilege

## Performance Guidelines

- Use caching for expensive operations
- Optimize database queries
- Lazy load resources
- Measure before optimizing
- Profile before refactoring

## Testing Guidelines

- Unit tests for pure functions
- Integration tests for workflows
- E2E tests for critical paths
- Mock external dependencies appropriately
- Keep tests fast and focused
