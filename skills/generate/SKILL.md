---
name: generate
description: >
  Iteratively generates and refines a CLAUDE.md file using iterative stress-test refinement
  with sequential agents. Spawns a fresh agent for each iteration — each gets a clean context window.
  Run /setup-claude-md:configure first to set options.
argument-hint: "[iterations=5]"
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, Task, AskUserQuestion
---

# Generate CLAUDE.md — Iterative Stress-Test Refinement

You are the orchestrator for CLAUDE.md generation. You coordinate iterative refinement by spawning a fresh agent for each iteration. Each agent invents a realistic development task, plans it against the current CLAUDE.md, discovers what information is missing, and feeds those learnings back in.

**You never do iteration work yourself.** You handle setup, repo scanning, agent spawning, interaction gates, and the final summary. The agents do the analytical work.

---

## Phase 0: Setup

1. **Parse arguments.** Check `$ARGUMENTS` for an iteration count override (e.g., `/setup-claude-md:generate 3`). If not provided, fall back to config, then default to 5.

2. **Load config.** Use `Glob` to check for `.claude-md-config.json` in the project root. If found, `Read` it and extract settings:
   - `iterations` (default: 5)
   - `interaction_mode` (default: `confirm-base`)
   - `focus_areas` (default: `all`)
   - `max_lines` (default: 150)
   - `generate_rules` (default: false)

3. **Check for resume state.** Use `Glob` to check for `.claude-md-generator-state.json` in the project root. If found:
   - `Read` it and extract the state (phase, completed_iterations, config, repo_summary, archetypes_used)
   - Use `AskUserQuestion` to ask: "Found state from a previous run ({completed_iterations}/{total_iterations} iterations completed). Resume from where it left off, or start fresh?" Options: "Resume", "Start fresh"
   - If "Resume": skip to Phase 2–N, starting from iteration `completed_iterations + 1`, using the saved config and repo_summary
   - If "Start fresh": delete the state file and logs directory, continue to Phase 1

4. **Check for existing CLAUDE.md.** Use `Glob` to find `CLAUDE.md` in the project root. If it exists, `Read` it — this is your starting point. You'll refine it rather than replace it.

5. **Load quality standards.** `Read` the file `references/claude-md-standards.md` (relative to this skill file). Keep these criteria in mind throughout.

---

## Phase 1: Repository Scoping & Base CLAUDE.md

Perform a comprehensive repository analysis. Do NOT skip any of these steps.

### Step 1.1 — Structure Scan
Use `Glob` to find key files. Run these in parallel:
- `**/package.json`, `**/Cargo.toml`, `**/pyproject.toml`, `**/go.mod`, `**/Gemfile`
- `**/Makefile`, `**/Dockerfile`, `**/docker-compose.*`, `**/Justfile`
- `**/.github/workflows/*`, `**/.gitlab-ci.yml`, `**/Jenkinsfile`
- `**/tsconfig.json`, `**/*.config.*`, `**/.eslintrc*`, `**/.prettierrc*`
- `**/README.md`, `**/CONTRIBUTING.md`, `**/docs/**`

### Step 1.2 — Tech Stack Identification
From the files found, identify:
- Primary language(s) and version(s)
- Frameworks (web, CLI, library, etc.)
- Build system (npm scripts, make, cargo, etc.)
- Test runner and assertion library
- Linter(s) and formatter(s)
- Package manager (npm, yarn, pnpm, pip, cargo, etc.)

### Step 1.3 — Read Config Files
`Read` the most important config files found in Step 1.1. Extract:
- Build commands
- Test commands (all tests, single test, watch mode)
- Lint/format commands
- Run/serve commands (dev and production)
- Scripts that aren't obvious from their name

### Step 1.4 — Read Existing Documentation
`Read` README.md, CONTRIBUTING.md, and any existing CLAUDE.md. Extract information relevant to developers working in the codebase. Note whether these files contain content worth referencing via `@README.md` or `@CONTRIBUTING.md` imports rather than duplicating in the generated CLAUDE.md.

### Step 1.5 — Architectural Analysis
Use `Glob` and `Read` to understand:
- Monorepo vs single package
- Top-level directory roles (prefer `Glob` patterns; use `Bash` with `ls` only as a fallback)
- Module boundaries and how they communicate
- Entry points (main files, route definitions, CLI entry)
- Shared utilities and where they live

### Step 1.6 — Generate Initial CLAUDE.md
Based on your analysis, generate the initial CLAUDE.md. Include ONLY sections that are warranted by what you found. Do not create empty sections or sections with generic content.

Possible sections (include only if you have substantive content):
- **Project overview** — 1–2 sentences max
- **Tech stack** — language, framework, key deps
- **Common commands** — build, test, lint, run in code blocks
- **Project structure** — brief directory map with roles (only non-obvious ones)
- **Code conventions** — only things not enforced by tooling
- **Known gotchas** — things that would trip up a developer

Apply the quality standards from `references/claude-md-standards.md` as you write.

If the project is a monorepo with packages that use different tech stacks or conventions, generate sub-directory CLAUDE.md files (e.g., `packages/frontend/CLAUDE.md`) for packages whose conventions differ significantly from the root. Keep the root CLAUDE.md for shared conventions only. Claude automatically loads sub-directory CLAUDE.md files on demand when working in those directories.

### Step 1.7 — Write State File
Compose a repo summary string (e.g., "TypeScript/React project using Vite, pnpm, Vitest. Monorepo: src/, tests/, docs/.") and write the initial state file:

```json
{
  "phase": "iterating",
  "total_iterations": <iterations>,
  "completed_iterations": 0,
  "archetypes_used": [],
  "config": {
    "interaction_mode": "<interaction_mode>",
    "focus_areas": "<focus_areas>",
    "max_lines": <max_lines>,
    "generate_rules": <generate_rules>
  },
  "repo_summary": "<repo_summary>"
}
```

Write this to `.claude-md-generator-state.json` in the project root. Also create the `.claude-md-generator-logs/` directory using `Bash` with `mkdir -p`.

### Step 1.8 — Interaction Gate
Check the `interaction_mode` setting:
- If `confirm-base` or `confirm-each`: Present the base CLAUDE.md to the user using `AskUserQuestion`. Ask: "Here's the initial CLAUDE.md. Should I proceed with iterative refinement, or do you want to edit anything first?" Provide options: "Looks good, proceed", "I have feedback" (let them type).
- If `autonomous`: Continue without stopping.

---

## Phase 2–N: The Iteration Loop

For each iteration `i` from 1 to `iterations` (or from `completed_iterations + 1` if resuming):

Announce: "**Iteration {i}/{total} — {archetype_name}**"

### Step 2.1 — Determine Archetype
Select archetype number `((i - 1) % 10) + 1` from the archetypes list. If the repo has fewer than 5 source files, skip archetypes 8, 9, 10 and use `((i - 1) % 7) + 1`.

Look up the archetype name and template text from `references/task-archetypes.md`.

### Step 2.2 — Determine Complexity Guidance
- Iterations 1–2: "Single-module scope. Task touches 1–2 files."
- Iterations 3–4 (or 3–5): "Multi-file scope. Task spans 3–5 files across 2+ directories."
- Iterations 5+ (or 6+): "Cross-cutting scope. Task affects a subsystem or requires coordinated changes."

### Step 2.3 — Construct Agent Prompt
`Read` the file `references/iteration-prompt.md` (relative to this skill file). Fill in all `{placeholder}` values:

- `{iteration}` — current iteration number
- `{total}` — total iterations
- `{archetype_name}` — name from Step 2.1 (e.g., "Bug Fix (Low Complexity)")
- `{archetype_template}` — template and example text from the archetypes file for this archetype
- `{complexity_guidance}` — from Step 2.2
- `{claude_md_path}` — absolute path to the project's CLAUDE.md
- `{standards_path}` — absolute path to `references/claude-md-standards.md`
- `{archetypes_path}` — absolute path to `references/task-archetypes.md`
- `{log_path}` — absolute path to `.claude-md-generator-logs/iteration-{i}.md`
- `{focus_areas}` — from config
- `{max_lines}` — from config
- `{generate_rules}` — from config
- `{repo_summary}` — from state file or Phase 1

### Step 2.4 — Spawn Iteration Agent
Spawn the agent using the `Task` tool:
```
Task(
  subagent_type: "general-purpose",
  description: "Refinement iteration {i}",
  prompt: <the constructed prompt from Step 2.3>
)
```

### Step 2.5 — Post-Iteration Bookkeeping
After the agent completes:
1. `Read` the iteration log at `.claude-md-generator-logs/iteration-{i}.md`
2. Update the state file: increment `completed_iterations`, append the archetype name to `archetypes_used`
3. Write the updated state file

### Step 2.6 — Interaction Gate
If `interaction_mode` is `confirm-each`:
- Show the user the iteration log (invented task, frustration log, learnings, changes made)
- Use `AskUserQuestion`: "Here are the changes from iteration {i}. Accept, modify, or skip?" Options: "Accept changes", "Skip these changes", "I have feedback"
- If "Skip these changes": revert CLAUDE.md to the version before this iteration (re-read the previous version from iteration `i-1`'s log or the original if `i == 1`)

---

## Phase Final: Review Agent

Spawn a final review agent using the `Task` tool:

```
Task(
  subagent_type: "general-purpose",
  description: "Final CLAUDE.md review",
  prompt: <final review prompt below>
)
```

The final review prompt (construct inline):

```
You are a CLAUDE.md reviewer. Read the CLAUDE.md at {claude_md_path} and perform a final quality pass.

Read the quality standards at {standards_path}.

1. Apply the removal test to every line: "If I deleted this line, would Claude make a concrete mistake on a real development task?" Remove anything that fails.
2. Verify formatting: ## headers for sections, bullet lists, fenced code blocks for commands, no trailing whitespace or double blank lines.
3. Spot-check 1–2 documented commands by running them via Bash. If a command fails, fix or annotate it.
4. Write the final CLAUDE.md to {claude_md_path}.
```

---

## Phase Cleanup

1. Delete `.claude-md-generator-state.json` using `Bash` with `rm`.
2. Delete `.claude-md-generator-logs/` directory using `Bash` with `rm -rf`.
3. Present the summary:

```
CLAUDE.md generation complete.

Iterations run:    {count}
Archetypes used:   {list of archetype names}
Sections:          {list of section names from final CLAUDE.md}
Final line count:  {count}
```

If `generate_rules` was enabled, also list:
```
Generated rule files:
- .claude/rules/{topic1}.md
- .claude/rules/{topic2}.md
```

4. **Show the final CLAUDE.md content** — `Read` the final CLAUDE.md and display it to the user so they can see exactly what was generated without having to open the file themselves.

---

## Important Reminders

- **NEVER execute invented tasks.** Agents only plan them to find gaps.
- **NEVER include information Claude can get from reading code.** The CLAUDE.md is for things that AREN'T in the code.
- **Be ruthless about pruning.** A short, dense CLAUDE.md is far more valuable than a long, padded one.
- **Each iteration should discover NEW information.** If an agent finds nothing new, that's a signal you may have enough iterations.
- **Reference real files.** Every task agents invent must reference actual files and modules in the repository.
