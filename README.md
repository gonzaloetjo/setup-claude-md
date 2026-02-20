# generate-claude-md

A Claude Code plugin that iteratively generates and refines `CLAUDE.md` files using the Ralph loop.

## What It Does

Instead of generating a `CLAUDE.md` in one shot, this plugin repeatedly:

1. **Invents** a realistic development task (bug fix, new feature, refactor, etc.)
2. **Plans** the task against the current `CLAUDE.md` without executing it
3. **Discovers** what information is missing — every guess or assumption is a gap
4. **Feeds** those learnings back into the `CLAUDE.md`

Each iteration stress-tests the `CLAUDE.md` from a different angle, producing a file that contains exactly what Claude needs and nothing it doesn't.

## Installation

### From the Plugin Marketplace (Recommended)

```bash
/plugin marketplace add gonzaloetjo/generate-claude-md
/plugin install generate-claude-md@generate-claude-md
```

### As a Standalone Skill

Copy the skill files directly into your project:

```bash
cp -r skills/generate/ your-project/.claude/skills/generate-claude-md/
cp -r skills/configure/ your-project/.claude/skills/configure-claude-md/
```

### For Development

Clone the repo and point Claude Code at it:

```bash
git clone https://github.com/gonzaloetjo/generate-claude-md.git
claude --plugin-dir ./generate-claude-md
```

## Usage

### 1. Configure (Optional)

```
/generate-claude-md:configure
```

Walks you through setting iteration count, interaction mode, focus areas, max line count, and whether to generate `.claude/rules/` files. Settings are saved to `.claude-md-config.json` in your project root.

### 2. Generate

```
/generate-claude-md:generate
```

Or override the iteration count:

```
/generate-claude-md:generate 3
```

## How the Ralph Loop Works

### Phase 1: Repository Scan
Scans your repository to identify the tech stack, project structure, build/test/lint commands, and existing documentation. Generates an initial `CLAUDE.md` from this analysis.

### Phase 2–N: Iterative Refinement
For each iteration, the plugin:

1. **Picks a task archetype** from 10 categories (bug fix, test addition, new feature, refactor, API change, dependency update, CI/CD change, performance optimization, security hardening, cross-cutting concern)
2. **Generates a specific task** referencing real files in your repo
3. **Plans the task step-by-step**, noting every guess or assumption
4. **Categorizes the gaps** found during planning
5. **Updates the CLAUDE.md** with critical learnings
6. **Prunes aggressively**, removing anything Claude could infer from code

Task complexity escalates: early iterations are single-file, middle iterations span multiple files, later iterations are cross-cutting.

### Phase Final: Validation
Reviews every line one last time, verifying that removing it would cause Claude to make a concrete mistake. Spot-checks commands by running them.

## Configuration Options

| Setting | Options | Default |
|---------|---------|---------|
| **Iterations** | 1–10 | 5 |
| **Interaction mode** | `autonomous`, `confirm-base`, `confirm-each` | `confirm-base` |
| **Focus areas** | `all`, or subset of: commands, architecture, testing, style, gotchas, workflow | `all` |
| **Max lines** | 50–300 | 150 |
| **Generate rules** | true/false — extract long sections to `.claude/rules/` | false |

## Interaction Modes

- **`autonomous`** — Runs all iterations without stopping. Best for trusted repos you know well.
- **`confirm-base`** — Shows the initial `CLAUDE.md` for review, then runs iterations autonomously. Good balance of control and speed.
- **`confirm-each`** — Pauses after every iteration to show the invented task, learnings, and proposed changes. Maximum control.

## Project Structure

```
generate-claude-md/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata
│   └── marketplace.json     # Marketplace manifest
├── skills/
│   ├── generate/
│   │   ├── SKILL.md         # Main generation skill (the Ralph loop)
│   │   └── references/
│   │       ├── claude-md-standards.md   # Quality criteria & pruning rules
│   │       └── task-archetypes.md       # 10 task archetypes for iterations
│   └── configure/
│       └── SKILL.md         # Configuration skill
├── README.md
└── LICENSE
```

## License

MIT
