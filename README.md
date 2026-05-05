# opencode-agents-pm

Multi-agent AI development template with **Project Manager** workflow for [opencode](https://github.com/sst/opencode).

## What Is This?

A ready-to-use template for setting up **multi-agent AI development workflows** with a Project Manager (PM) role. Instead of a single AI assistant doing everything, work is coordinated by Project Manager (the PM) and delegated to specialized agents—each optimized for their role.

## The PM Workflow

```
User: "Add user authentication to the app"
    │
    ▼
  Project Manager (PM)
    │
    ├──→ Sprint Planning
    │     ├── Break down description
    │     └── Create task list
    │
    ├──→ Sam (research + plan)
    │
    ├──→ Jordan (challenge) ← optional, for risky changes
    │
    ├──→ Ilias (implement)
    │
    └──→ Present Product Increment
          │
          ▼
        User Feedback
          │
          ├──→ Approve → Sprint Complete ✓
          └──→ Request Changes → Delegate to Ilias → Re-present
```

**Key Difference from opencode-agents:**
- Oscar is now a **Project Manager**, not just an Orchestrator
- Executes **sprint descriptions** from the user
- **Presents product increments** when work is done
- **Gathers feedback** and iterates until user approves
- Uses whatever default model is configured

## The Agents

### Core Agents

| Agent | Role | Key Trait |
|-------|------|-----------|
| **Project Manager** | Project Manager | Coordinates sprints, presents increments, gathers feedback |
| **Sam** | Researcher + Planner | Digs deep into codebases, creates actionable implementation plans |
| **Ilias** | Implementor | Writes code, runs tests, follows specs precisely |
| **Jordan** | Truth-Teller | Challenges assumptions, finds blind spots (called for risky changes) |

### Jordan Variants

| Agent | Use Case |
|-------|----------|
| **jordan** | Default truth-teller |
| **jordan_qwen** | Code-focused analysis |
| **jordan_grok** | Alternative perspective |

### The Project Manager Pattern

```
User Request (Sprint Description)
    │
    ▼
  Project Manager (PM)
    │
    ├──→ Sprint Planning
    │     └── Create todo list, prioritize tasks
    │
    ├──→ Sam (research + plan)
    │         │
    │         ├──→ Jordan (challenge) ← optional
    │         │
    │         ▼
    └──→ Ilias (implement)
                │
                ▼
          Work Complete
                │
                ▼
          Present Increment to User
                │
                ├──→ Feedback: "Looks good" → Sprint Done ✓
                └──→ Feedback: "Change X" → Delegate to Ilias → Re-present
```

**Why this pattern?**

- **Sprint-based** — Work is organized into manageable sprints from user descriptions
- **Feedback loops** — User reviews product increments before sprint closure
- **Context efficiency** — Oscar stays lean, delegating heavy lifting to specialists
- **Quality gates** — Jester provides adversarial review for risky changes
- **Default model** — Uses whatever model is configured in opencode

## Sprint Management Workflow

### 1. Sprint Planning
User provides a sprint description (e.g., "Add user authentication", "Fix all critical bugs"). Project Manager:
- Breaks down into actionable tasks using `@sam` for research/planning
- Creates a todo list using `todowrite` to track progress
- Prioritizes tasks (must-have, should-have, could-have)

### 2. Delegation
- Research/planning → `@sam`
- Implementation → `@ilias`
- Risky changes (>5 files) → `@jordan` for review first

### 3. Progress Tracking
- Updates todo list as tasks complete
- Checks in with subagents periodically
- Escalates blockers immediately

### 4. Product Increment Presentation
When all tasks are done, Project Manager presents to the user:
- Summary of changes made
- Files modified
- Tests passing
- Demo of new functionality (if applicable)

### 5. Feedback & Iteration
- Collects feedback using `question` tool
- Delegates changes to `@ilias`
- Repeats until user approves

### 6. Sprint Closure
- Commits all changes (delegated to `@ilias`)
- Closes relevant GitHub issues
- Documents lessons learned

## Installation

### 1. Install opencode

```bash
curl -fsSL https://opencode.ai/install | bash
```

Or see [opencode installation docs](https://github.com/sst/opencode#installation).

### 2. Run the installer

```bash
# Clone this repo
git clone https://github.com/yourusername/opencode-agents-pm.git
cd opencode-agents-pm

# Run the installer script
./install.sh
```

The installer copies agent definitions to `~/.config/opencode/agent/`.

### 3. Configure opencode

```bash
# Copy the example configuration
cp opencode.json.example ~/.config/opencode/opencode.json

# Edit to customize models (optional)
nano ~/.config/opencode/opencode.json
```

### 4. Copy AGENTS.md to your project

```bash
# Copy and customize the template AGENTS.md
cp AGENTS.md /path/to/your/project/
```

Edit `AGENTS.md` in your project to add project-specific context.

### 5. Start using agents

```bash
# In your project directory
opencode
```

Then talk to Project Manager with a sprint description:

```
@project_manager: Add user authentication with JWT tokens. Include login, logout, and protected routes.
```

## Configuration

The `opencode.json.example` file contains the full agent configuration:

```json
{
  "default_agent": "project_manager",
  "agent": {
    "project_manager": { ... },
    "sam": { ... },
    "ilias": { ... },
    "jordan": { ... },
    "jordan_qwen": { ... },
    "jordan_grok": { ... }
  }
}
```

### Model Configuration

This template doesn't specify models, allowing opencode to use whatever default model is installed.

### Customizing Agents

Edit `~/.config/opencode/opencode.json` to:
- **Change the default agent** — Update the `"default_agent"` field
- **Add new variants** — Create additional Jordan entries with different prompts or configurations
- **Modify agent settings** — Adjust descriptions, modes, or other agent properties

## File Structure

```
opencode-agents-pm/
├── .opencode/
│   ├── agent/
│   │   ├── project_manager.md # Project Manager (NEW)
│   │   ├── sam.md             # Researcher + Planner
│   │   ├── ilias.md           # Implementor
│   │   └── jordan.md          # Truth-Teller
│   └── skills/
│       ├── python-code-review/   # Python code review checklist
│       ├── python-testing/       # pytest patterns and best practices
│       ├── python-venv/          # Virtual environment management
│       ├── pr-review/            # Pull request review guidelines
│       ├── git-commit/           # Commit message conventions
│       ├── issue-triage/         # GitHub issue triage workflow
│       ├── prompt-engineering/   # LLM prompt design patterns
│       ├── data-pipeline/        # Data pipeline best practices
│       ├── ml-experiment/        # ML experiment tracking
│       └── agent-tuning/         # Agent prompt optimization
├── AGENTS.md             # Template for project-specific context
├── README.md             # This file
├── install.sh            # Installer script
└── opencode.json.example # Example configuration (default models only)
```

## Skills

Skills are reusable knowledge modules that agents can load on-demand using the `Skill` tool. Each skill contains domain-specific expertise in a `SKILL.md` file.

### Available Skills

| Skill | Description |
|-------|-------------|
| **python-code-review** | Comprehensive Python code review checklist covering style, types, error handling, and performance |
| **python-testing** | pytest patterns, fixtures, mocking strategies, and test organization |
| **python-venv** | Virtual environment setup, dependency management, and common pitfalls |
| **pr-review** | Pull request review guidelines for thorough, constructive feedback |
| **git-commit** | Conventional commit message format and best practices |
| **issue-triage** | GitHub issue triage workflow for prioritization and labeling |
| **prompt-engineering** | LLM prompt design patterns, few-shot examples, and optimization techniques |
| **data-pipeline** | Data pipeline architecture, validation, and monitoring patterns |
| **ml-experiment** | ML experiment tracking, reproducibility, and model versioning |
| **agent-tuning** | Agent prompt optimization and behavior refinement techniques |
| **moscow** | MoSCoW prioritization (Must/Should/Could/Won't) |
| **rice** | RICE prioritization scoring |
| **retro** | Sprint retrospective framework |

### How Skills Work

Agents with `skill: true` in their frontmatter can load skills dynamically:

```yaml
---
tools: [Read, Write, Glob, Grep, Bash, Task]
skill: true
---
```

When an agent needs specialized knowledge, they call the Skill tool:

```
Agent: I need to review this Python code thoroughly.
[Loads skill: python-code-review]
Agent: Now applying the checklist...
```

## Key Principles

1. **Project Manager is a Project Manager** — Coordinates sprints, presents increments, gathers feedback
2. **Sam digs deep, plans lean** — Research flows naturally into actionable tasks
3. **Ilias follows specs** — No improvisation; if the plan is unclear, ask
4. **Jordan challenges** — Called for complex refactors (>5 files) or risky changes
5. **Model agnostic** — Works with whatever default model is configured in opencode

## When to Call Jordan

Jordan runs at high temperature (0.8) intentionally—he's a wildcard oracle. Call him when:

- Complex refactors touching >5 files
- Risky architectural changes
- The team is stuck or going in circles
- A plan feels "correct" but dead
- Everyone agrees too quickly (dangerous!)

Most of what Jordan says is noise, but buried in there is golden insight. Pan for gold.

## Customization

The agent files are designed to be project-agnostic. Customize them by:

1. **Adjusting tool permissions** in the frontmatter
2. **Adding project-specific rules** to `AGENTS.md`
3. **Modifying code standards** in Ilias's file for your language/framework
4. **Updating Project Manager's PM workflow** for your team's sprint process

## License

MIT
