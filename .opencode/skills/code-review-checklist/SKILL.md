---
name: code-review-checklist
description: Language-agnostic code review checklist. Covers security, performance, readability, maintainability, testing, and common pitfalls across languages. Use when performing code reviews, creating review guidelines, or when the user mentions code review, PR review, review checklist, or code quality.
---

# Code Review Checklist Skill

Perform thorough, language-agnostic code reviews with a comprehensive checklist.

## When to Use
- Reviewing pull requests
- Creating code review guidelines
- Mentoring developers
- Establishing review standards
- When the user mentions code review or PR review

## The Review Process

### 1. Understand the Context
- Read the PR description and linked issues
- Understand what the code is supposed to do
- Check if tests are included
- Verify documentation is updated

### 2. Review in Layers
1. **High-level**: Architecture, design patterns, approach
2. **Mid-level**: Function/method design, error handling, logic
3. **Low-level**: Naming, formatting, comments

## Comprehensive Checklist

### Architecture & Design
- [ ] **Single Responsibility**: Each class/function has one clear purpose
- [ ] **Open/Closed**: Open for extension, closed for modification
- [ ] **Don't Repeat Yourself (DRY)**: No duplicated code
- [ ] **Composition over Inheritance**: Prefer composition where appropriate
- [ ] **Interface Segregation**: Small, focused interfaces
- [ ] **Dependency Inversion**: Depend on abstractions, not concretions

### Security
- [ ] **Input validation**: All user input is validated/sanitized
- [ ] **SQL injection**: Parameterized queries used (no string concatenation)
- [ ] **XSS prevention**: Output is escaped in web contexts
- [ ] **Secrets**: No API keys, passwords, or tokens in code
- [ ] **Authentication**: Proper auth checks on protected endpoints
- [ ] **Authorization**: User has permission for the action
- [ ] **HTTPS**: Sensitive data sent over secure connections
- [ ] **Rate limiting**: APIs have rate limiting where appropriate

### Performance
- [ ] **N+1 queries**: Database queries are batched (not in loops)
- [ ] **Algorithm complexity**: O(n²) or worse is justified
- [ ] **Caching**: Expensive operations are cached where appropriate
- [ ] **Pagination**: Large result sets are paginated
- [ ] **Memory usage**: No obvious memory leaks or excessive allocation
- [ ] **Async/await**: I/O operations don't block unnecessarily

### Readability & Maintainability
- [ ] **Naming**: Variables, functions, classes have clear, descriptive names
- [ ] **Function length**: Functions are focused (< 50 lines ideally)
- [ ] **Complexity**: Cyclomatic complexity is manageable
- [ ] **Comments**: Explain "why", not "what" (code should be self-documenting)
- [ ] **Formatting**: Consistent with project style guide
- [ ] **Magic numbers**: Replaced with named constants
- [ ] **Dead code**: No unused variables, functions, or imports

### Error Handling
- [ ] **Try/catch**: Errors are caught and handled appropriately
- [ ] **Error messages**: User-friendly, not exposing internals
- [ ] **Logging**: Errors are logged with context
- [ ] **Fail fast**: Invalid state is caught early
- [ ] **Graceful degradation**: System handles failures gracefully

### Testing
- [ ] **Unit tests**: New code has corresponding tests
- [ ] **Test coverage**: Important paths are covered
- [ ] **Test quality**: Tests are meaningful, not just for coverage
- [ ] **Edge cases**: Boundary conditions are tested
- [ ] **Mocks/Stubs**: External dependencies are mocked appropriately
- [ ] **Test naming**: Test names describe what they test

### Documentation
- [ ] **Public API**: Functions/classes have docstrings/comments
- [ ] **Complex logic**: Non-obvious code has explanation
- [ ] **README**: Updated if setup/usage changed
- [ ] **API docs**: Endpoint documentation updated
- [ ] **Comments**: Not excessive or redundant

### Git/PR Hygiene
- [ ] **Commit messages**: Clear, follow project convention
- [ ] **PR size**: Not too large (preferably < 500 lines)
- [ ] **Branch**: From correct base branch
- [ ] **Conflicts**: Resolved cleanly
- [ ] **History**: No unnecessary merge commits

## Language-Specific Gotchas

### Python
- [ ] Mutable default arguments avoided (`def func(x=[])` ← bad)
- [ ] Context managers used for resources (`with open(...)`)
- [ ] f-strings or `.format()` used (not `%` formatting)
- [ ] Type hints included for public APIs

### JavaScript/TypeScript
- [ ] `var` not used (use `const`/`let`)
- [ ] `==` avoided (use `===`)
- [ ] Callbacks replaced with async/await where possible
- [ ] `any` type avoided in TypeScript

### Go
- [ ] Errors are checked (not ignored)
- [ ] `defer` used for cleanup
- [ ] No `panic()` in production code
- [ ] Interfaces are small and focused

### Java
- [ ] Null checks (or use Optional)
- [ ] Resources use try-with-resources
- [ ] Exceptions are specific (not generic Exception)
- [ ] Collections use generics

## Review Comment Templates

### For Architectural Concerns
```
**Architecture Concern**
This approach couples X and Y too tightly. Consider using [pattern] to decouple them.
This would make it easier to [benefit].
```

### For Security Issues
```
**Security Issue** 🔒
Found: [description of vulnerability]
Suggestion: Use [approach] to prevent this.
Reference: [OWASP link if applicable]
```

### For Performance
```
**Performance Note** ⚡
This loop executes N+1 queries. Consider using [eager loading/batch approach] instead.
Current: O(n) queries
Suggested: O(1) query
```

### For Nitpicks
```
**Nitpick** (non-blocking)
Consider renaming `x` to `userCount` for clarity.
```

## Review Etiquette

### Do
- Be respectful and constructive
- Explain "why", not just "what"
- Suggest solutions, not just problems
- Praise good code
- Ask questions if unclear

### Don't
- Be sarcastic or condescending
- Block on personal preferences (use "consider")
- Review too fast (give it proper time)
- Focus only on nitpicks
- Demand changes without explanation

## Verification

After completing a code review:
1. Verify all checklist items were considered
2. Ensure feedback is actionable
3. Check that tone is constructive
4. Confirm you've tested the code (if possible)
5. Approve only when confident in the changes
