# setup-claude-md

> Iterative CLAUDE.md generation through agent-based stress-test refinement.

## The Problem

Claude Code's built-in `/init` generates a CLAUDE.md in a single pass — one scan, one output, done. That works for simple projects, but for complex codebases the result misses non-obvious conventions, gotchas, and workflow details that only surface when you actually try to do real work. `setup-claude-md` closes this gap by spawning multiple agents that each attempt a realistic development task against your documentation, discovering what's missing from the perspective of a developer who has never seen the codebase before.

## How It Works

Generation runs in three phases. Each phase uses fresh agents with clean context windows, spawned via Claude Code's `Task` tool.

```
Phase 0: Read existing CLAUDE.md
    │
    ▼
┌─────────────────────────────────────────┐
│  Phase 1–N: Iterative Refinement        │
│                                         │
│  For each iteration:                    │
│    1. Select archetype                  │
│    2. Invent a concrete dev task        │
│    3. Plan the task (never execute)     │
│    4. Record every guess/assumption     │
│    5. Update CLAUDE.md with learnings   │
│    6. Prune aggressively                │
│                                         │
│  (one fresh agent per iteration)        │
└─────────────────────────────────────────┘
    │
    ▼
Phase 2: Validation Agent
```

### Prerequisite: Run `/init` first

This plugin requires a CLAUDE.md to already exist in your project root. Run `/init` to create one, or write it manually. The plugin reads the existing CLAUDE.md, synthesizes a repo summary from its content, and uses that as the starting point for iterative refinement. State is written to `.claude-md-generator-state.json` so interrupted runs can resume.

### Phase 1–N: Iterative Refinement

Each iteration spawns a fresh `general-purpose` agent that picks a task archetype, invents a specific task referencing real files in your repo, plans it step-by-step without executing, and records every point where it had to guess. Those gaps become CLAUDE.md updates.

**Agent Roster — 10 Task Archetypes:**

| # | Archetype | Complexity | What It Stress-Tests |
|---|-----------|------------|----------------------|
| 1 | Bug Fix | Low | Error handling, debugging workflow, test commands |
| 2 | Test Addition | Low | Test framework, file location, naming, mocking patterns |
| 3 | New Feature | Medium | Where new code goes, architectural patterns, naming |
| 4 | Refactor | Medium | Architecture boundaries, code organization, interfaces |
| 5 | API Change | Medium | API conventions, validation, route organization |
| 6 | Dependency Update | Medium | Build commands, breaking change handling, verification |
| 7 | CI/CD Change | Medium | CI config, env vars, deployment workflow, branch conventions |
| 8 | Performance Optimization | High | Profiling, database patterns, caching, benchmarks |
| 9 | Security Hardening | High | Security patterns, input validation, auth conventions |
| 10 | Cross-Cutting Concern | High | Shared utilities, config management, consistency rules |

Task complexity escalates across iterations: early iterations target single files, middle iterations span multiple directories, later iterations tackle cross-cutting concerns.

### Phase 2: Validation Agent

A final agent reads the complete CLAUDE.md with fresh eyes. It applies the removal test to every line ("if deleted, would Claude make a concrete mistake?"), verifies formatting, and spot-checks documented commands by running them.

## Installation

### From the Marketplace (Recommended)

```
/plugin marketplace add gonzaloetjo/setup-claude-md
/plugin install setup-claude-md@setup-claude-md
```

### Interactive Discovery

```
/plugin
```

Navigate to the **Discover** tab and search for `setup-claude-md`.

### Manual / Development

```bash
git clone https://github.com/gonzaloetjo/setup-claude-md.git
claude --plugin-dir ./setup-claude-md
```

## Quick Start

**1. Create initial CLAUDE.md (if you don't have one):**

```
/init
```

**2. Configure (optional):**

```
/setup-claude-md:configure
```

Interactive walkthrough for iteration count, interaction mode, focus areas, max line count, and rule file generation.

**3. Generate:**

```
/setup-claude-md:generate
```

Override iteration count inline:

```
/setup-claude-md:generate 3
```

## Configuration

Settings are stored in `.claude-md-config.json` in your project root. Run `/setup-claude-md:configure` to create or update it interactively.

| Field | Options | Default | When to Change |
|-------|---------|---------|----------------|
| `iterations` | `1`–`10` | `5` | Use `3` for small projects, `8`+ for large monorepos |
| `interaction_mode` | `autonomous`, `confirm-base`, `confirm-each` | `confirm-base` | Use `autonomous` for trusted repos, `confirm-each` for first run on unfamiliar codebases |
| `focus_areas` | `all`, or subset: `commands`, `architecture`, `testing`, `style`, `gotchas`, `workflow` | `all` | Restrict when you only need specific sections updated |
| `max_lines` | `50`–`300` | `150` | Lower for focused libs, higher for monorepos |
| `generate_rules` | `true` / `false` | `false` | Enable to extract long sections into `.claude/rules/` files |

## Interaction Modes

- **`autonomous`** — Runs all iterations without stopping. Best for repos you know well where you trust the output.
- **`confirm-base`** — Shows the existing CLAUDE.md and pauses for your review before starting iterations, then runs all iterations autonomously. Good balance of control and speed.
- **`confirm-each`** — Pauses after every iteration showing the invented task, frustration log, learnings, and proposed changes. You can accept, skip, or provide feedback. Maximum control.

## When to Use

- Generating a CLAUDE.md for an established codebase with non-obvious conventions
- Replacing a stale or generic CLAUDE.md with one stress-tested against real tasks
- Onboarding Claude to a project where `/init` output was insufficient
- Auditing an existing CLAUDE.md for gaps by running iterations against it

## When Not to Use

- Brand-new projects with fewer than 5 files — `/init` is sufficient
- Projects where you need a CLAUDE.md in under a minute — this plugin runs multiple agents and is thorough, not fast
- Non-code repositories (docs-only, asset-only) — the task archetypes assume a codebase with source files

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Generation stops mid-run | Re-run `/setup-claude-md:generate` — it detects the state file and offers to resume |
| CLAUDE.md is too generic | Increase iterations to `8`+ or restrict `focus_areas` to force deeper exploration |
| CLAUDE.md is too long | Lower `max_lines` in config, or enable `generate_rules` to extract sections to `.claude/rules/` |
| Commands in CLAUDE.md are wrong | The validation agent spot-checks commands, but may not catch all. Run the documented commands manually after generation |
| Plugin not found after install | Verify with `/plugin list` — the plugin name is `setup-claude-md` |
| Skills don't appear | Ensure the plugin directory contains `.claude-plugin/plugin.json` and `skills/` with valid `SKILL.md` files |

## How It Compares to `/init`

| | `/init` | `setup-claude-md` |
|---|---------|---------------------|
| **Approach** | Single-pass scan | Multi-iteration stress-test refinement |
| **Agents** | 1 | 1 orchestrator + N iteration agents + 1 validation agent |
| **Task simulation** | None | Invents realistic dev tasks per iteration |
| **Gap detection** | Scans files | Plans tasks and records every assumption |
| **Pruning** | Basic | Removal test on every line, standards-based |
| **Resumable** | No | Yes — state file tracks progress |
| **Configurable** | No | Iterations, focus areas, interaction mode, max lines |
| **Best for** | Quick start, simple projects | Thorough docs, complex codebases |

Both tools produce valid CLAUDE.md files. Use `/init` for a quick baseline, then run `setup-claude-md` when you need deeper coverage.

## Project Structure

```
setup-claude-md/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata
│   └── marketplace.json     # Marketplace manifest
├── skills/
│   ├── generate/
│   │   ├── SKILL.md         # Orchestrator — spawns agents per iteration
│   │   └── references/
│   │       ├── claude-md-standards.md   # Quality criteria & pruning rules
│   │       ├── task-archetypes.md       # 10 task archetypes
│   │       └── iteration-prompt.md      # Iteration agent prompt template
│   └── configure/
│       └── SKILL.md         # Interactive configuration skill
├── README.md
└── LICENSE
```

## Requirements

- [Claude Code](https://docs.anthropic.com/en/docs/claude-code) CLI with plugin support
- A codebase with source files (the task archetypes assume working code)
- An existing CLAUDE.md in the project root (run `/init` to create one)

## License

MIT

## Author

[gonzaloetjo](https://github.com/gonzaloetjo)
