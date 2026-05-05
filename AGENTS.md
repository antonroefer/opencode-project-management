# AGENTS.md - Multi-Agent Development Template (PM Workflow)

> **This is a template AGENTS.md - customize for your project**

## Project Overview

<!-- TODO: Add project-specific content -->
<!-- Describe what your project does, its goals, and key features -->

## The Agent System

This project uses a multi-agent system with a **Project Manager** workflow for context-efficient development.

### Agents

#### Core Agents

| Agent | Role | Mode |
|-------|------|------|
| **@project-manager** | Project Manager - coordinates sprints, presents increments, gathers feedback | primary |
| **@sam** | Researcher + Planner - deep analysis, actionable plans | subagent |
| **@ilias** | Implementor - writes code, runs tests | subagent |
| **@jordan** | Truth-Teller - challenges assumptions | subagent |

#### Jordan Variants (Default Models Only)

| Agent | Model | Purpose |
|-------|-------|---------|
| **@jordan** | Qwen3 Coder 480b | Default truth-teller |
| **@jordan_qwen** | Qwen3 Coder 480b | Code-focused analysis |
| **@jordan_grok** | Grok | Alternative perspective |

### Sprint Workflow

```
User: "Add user authentication"
    │
    ▼
  Project Manager (PM)
    │
    ├──→ Sprint Planning
    │     └── Break down, prioritize tasks
    │
    ├──→ Sam (research + plan)
    │
    ├──→ Jordan (challenge) ← optional
    │
    └──→ Ilias (implement)
            │
            ▼
      Present Increment to User
            │
            ├──→ Approve → Done ✓
            └──→ Changes → Iterate
```

### Key Principles

- **Project Manager is a Project Manager** - executes sprint descriptions, presents increments, gathers feedback
- **Sam digs deep, plans lean** - research flows into actionable plans
- **Ilias follows specs** - no improvisation
- **Jordan challenges** - called for complex refactors (>5 files), risky changes, or when stuck
- **Default models only** - no premium Claude models required

## Quick Start

<!-- TODO: Add project-specific content -->
<!-- Add setup instructions, common commands, and daily workflow -->

```bash
# Setup (TODO: Add language-specific setup)
# Examples:
# Python: python3 -m venv .venv && source .venv/bin/activate && pip install -r requirements.txt
# Node.js: npm install
# Go: go mod download
# Rust: cargo build

# Run tests (TODO: Add language-specific test command)
# Examples:
# Python: pytest tests/ -v
# Node.js: npm test
# Go: go test ./...
# Rust: cargo test
```

## Development Environment

**CRITICAL:** Always use the project's designated environment/package manager for ALL operations.

Examples by language:
- **Python**: `python3 -m venv .venv && source .venv/bin/activate` then use `.venv/bin/python`, `.venv/bin/pip`
- **Node.js**: `npm install` then use `npx <command>` or `npm run <script>`
- **Go**: Use `go run`, `go test`, `go build` directly (no virtual env needed)
- **Rust**: `cargo build`, `cargo test`, `cargo run`
- **Java**: `mvn clean install` or `gradle build`

**Always** use project-specific commands to avoid using wrong global installations.

## Sprint Rules

<workflow>
### Sprint Planning
When `@project-manager` receives a sprint description from the user:
1. **Break down** the description into actionable tasks
2. **Prioritize** tasks (must-have, should-have, could-have)
3. **Create todo list** using `todowrite`
4. **Delegate research** to `@sam` for complex tasks

### Delegation
- Research/planning → `@sam`
- Implementation → `@ilias`
- Risky changes (>5 files) → `@jordan` for review first

### Progress Tracking
- Update todo list as tasks complete
- Check in with subagents periodically
- Escalate blockers immediately

### Product Increment Presentation
When all tasks are done, `@project-manager` presents to the user:
1. **Summary** of changes made
2. **Files modified**
3. **Tests passing**
4. **Demo** of new functionality (if applicable)

### Feedback & Iteration
- Collect feedback using `question` tool
- Delegate changes to `@ilias`
- Repeat until user approves

### Sprint Closure
- Commit all changes (delegated to `@ilias`)
- Close relevant GitHub issues
- Document lessons learned

### Commit Message Format
```
<type> #<issue>: <description>

Types: fix, feat, refactor, docs, test, chore
```

Examples:
- `fix #42: handle null response from API`
- `feat #15: add user authentication`
- `refactor #30: extract validation logic`
</workflow>

## Scripts

<!-- TODO: Add project-specific content -->
<!-- Document your project's scripts and their purposes -->

| Script | Purpose |
|--------|---------|
| `example.py` / `example.js` / `main.go` | Description of what it does |

## Project Structure

<!-- TODO: Add project-specific content -->
<!-- Document your project's directory structure -->

```
your-project/
├── AGENTS.md                 # This file
├── requirements.txt          # Dependencies (Python)
├── package.json              # Dependencies (Node.js)
├── Cargo.toml                # Dependencies (Rust)
├── go.mod                    # Dependencies (Go)
├── src/                      # Source code
├── tests/                    # Unit tests
└── scripts/                  # Utility scripts
```

## Architecture

<!-- TODO: Add project-specific content -->
<!-- Document key architectural patterns, abstractions, and design decisions -->

## Key Modules

<!-- TODO: Add project-specific content -->
<!-- Document the main modules and their responsibilities -->

| Module | Purpose |
|--------|---------|
| `module.py` | Description |

## Configuration

<!-- TODO: Add project-specific content -->
<!-- Document configuration files and environment variables -->

## Code Style

Follow the conventions of the language you're working in:
- **Python**: PEP 8, type hints, Google-style docstrings
- **JavaScript/TypeScript**: ESLint config, JSDoc/TSDoc
- **Go**: gofmt, effective Go guidelines
- **Rust**: rustfmt, clippy guidelines
- **Java**: Checkstyle, Javadoc

**General principles:**
- Consistent formatting (use project's formatter)
- Meaningful names
- Clear, concise comments where necessary
- Follow existing project patterns

<!-- TODO: Add project-specific code style rules if needed -->
