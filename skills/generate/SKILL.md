---
name: generate
description: >
  Iteratively generates and refines a CLAUDE.md file for a repository using the Ralph loop.
  Invents random dev tasks, plans them (without executing), discovers missing context,
  and feeds learnings back into the CLAUDE.md. Run /generate-claude-md:configure first to set options.
argument-hint: "[iterations=5]"
disable-model-invocation: true
allowed-tools:
  - Read
  - Glob
  - Grep
  - Bash
  - Write
  - Edit
  - Task
  - AskUserQuestion
---

# Generate CLAUDE.md — The Ralph Loop

You are a CLAUDE.md generator. You iteratively build a high-quality CLAUDE.md by inventing realistic development tasks, planning them against the current CLAUDE.md, discovering what information is missing, and feeding those learnings back in. Each iteration stress-tests the CLAUDE.md from a different angle.

**CRITICAL: You NEVER execute the invented tasks. You only PLAN them to discover what context is missing.**

---

## Phase 0: Setup

1. **Parse arguments.** Check `$ARGUMENTS` for an iteration count override (e.g., `/generate-claude-md:generate 3`). If not provided, fall back to config, then default to 5.

2. **Load config.** Use `Glob` to check for `.claude-md-config.json` in the project root. If found, `Read` it and extract settings:
   - `iterations` (default: 5)
   - `interaction_mode` (default: `confirm-base`)
   - `focus_areas` (default: `all`)
   - `max_lines` (default: 150)
   - `generate_rules` (default: false)

3. **Check for existing CLAUDE.md.** Use `Glob` to find `CLAUDE.md` in the project root. If it exists, `Read` it — this is your starting point. You'll refine it rather than replace it.

4. **Load quality standards.** `Read` the file `references/claude-md-standards.md` (relative to this skill file). Keep these criteria in mind throughout.

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
`Read` README.md, CONTRIBUTING.md, and any existing CLAUDE.md. Extract information relevant to developers working in the codebase.

### Step 1.5 — Architectural Analysis
Use `Glob` and `Read` to understand:
- Monorepo vs single package
- Top-level directory roles (use `Bash` with `ls` only if needed)
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

### Step 1.7 — Interaction Gate
Check the `interaction_mode` setting:
- If `confirm-base` or `confirm-each`: Present the base CLAUDE.md to the user using `AskUserQuestion`. Ask: "Here's the initial CLAUDE.md. Should I proceed with iterative refinement, or do you want to edit anything first?" Provide options: "Looks good, proceed", "I have feedback" (let them type).
- If `autonomous`: Continue without stopping.

---

## Phase 2–N: The Ralph Loop

Execute this loop for each iteration `i` from 1 to `iterations`.

Announce each iteration: "**Iteration {i}/{total} — {archetype_name}**"

### Step A — Invent a Development Task

1. `Read` the file `references/task-archetypes.md` (relative to this skill file) if you haven't already cached it.
2. Select archetype number `((i - 1) % 10) + 1` from the list. If the repo has fewer than 5 source files, skip archetypes 8, 9, 10 and use `((i - 1) % 7) + 1`.
3. Generate a specific, concrete task using the archetype template. The task MUST:
   - Reference real files and modules discovered in Phase 1
   - Be 2–3 sentences
   - Be specific enough to plan step-by-step
   - Match the complexity escalation: iterations 1–2 single-file, 3–4 multi-file, 5+ cross-cutting

Output the task clearly:
> **Invented Task:** [task description]

### Step B — Plan the Task (DO NOT EXECUTE)

Think through the implementation step by step. For each step, note what information you needed:

1. **Which files to modify?** — List specific file paths. If unsure where something goes, that's a gap.
2. **Which functions/classes to change?** — If you need to guess at the interface, that's a gap.
3. **What commands to run?** — Build, test, lint, type-check. If you're unsure of exact commands, that's a gap.
4. **What conventions to follow?** — Naming, patterns, error handling, logging. If you'd guess, that's a gap.
5. **What tests to write?** — Where do test files go? What framework? What patterns? Gaps if unsure.
6. **What could go wrong?** — Side effects, breaking changes, env requirements. Note assumptions.

**Keep a running frustration log** — every time you have to guess, assume, or feel uncertain, write it down:
```
Frustration log:
- Wasn't sure if tests go in __tests__/ or alongside source files
- Don't know the exact test command for a single file
- Unclear whether this project uses exceptions or Result types for errors
- ...
```

### Step C — Extract Learnings

Review your frustration log. For each gap, categorize it:

| Category | Examples |
|----------|----------|
| **Commands** | Build/test/lint commands not documented |
| **Architecture** | Module boundaries, data flow, design decisions |
| **Conventions** | Naming, file placement, patterns, error handling |
| **Testing** | Test location, framework, patterns, coverage expectations |
| **Gotchas** | Non-obvious constraints, env vars, platform issues |
| **Workflow** | PR process, branch naming, CI/CD quirks |

Rate each learning:
- **Critical** — Not knowing this would cause Claude to produce broken code or fail a CI check
- **Helpful** — Not knowing this would cause Claude to produce working but non-idiomatic code

**Drop** anything Claude could figure out just by reading the source code. If the answer is in a file Claude would naturally open, it doesn't belong in CLAUDE.md.

If `focus_areas` is not `all`, only retain learnings in the configured focus categories.

### Step D — Update CLAUDE.md

1. **Add learnings** to the appropriate sections of the current CLAUDE.md. Create new sections only if no existing section fits.

2. **Apply the pruning pass** (from `references/claude-md-standards.md`):
   - Remove anything Claude would know from reading the code
   - Remove vague or aspirational statements
   - Merge duplicate or overlapping entries
   - Apply the "removal test" to each line: if removing it wouldn't cause a concrete mistake, remove it

3. **Check length.** If the CLAUDE.md exceeds `max_lines`:
   - First, prune more aggressively (remove "helpful" items, keep only "critical")
   - If `generate_rules` is enabled and a section exceeds ~15 lines, extract it to `.claude/rules/<topic>.md` and replace the section body with an `@rules/<topic>.md` import line

4. **Write the updated CLAUDE.md** using `Edit` (if updating) or `Write` (if creating).

5. **Interaction gate** — If `interaction_mode` is `confirm-each`:
   - Show the user: the invented task, the frustration log, and the proposed CLAUDE.md changes
   - Use `AskUserQuestion`: "Here are the changes from iteration {i}. Accept, modify, or skip?"
   - Options: "Accept changes", "Skip these changes", "I have feedback"

---

## Phase Final: Validation & Cleanup

After all iterations are complete:

### Step F.1 — Full Review
`Read` the complete CLAUDE.md. Review every line.

### Step F.2 — Final Pruning
Apply the removal test one last time to every line:
> "If I deleted this line, would Claude make a concrete mistake on a real development task?"
- Clear yes → keep
- Anything else → delete

### Step F.3 — Format Check
Verify consistent formatting:
- `##` headers for sections
- Bullet lists for items within sections
- Fenced code blocks for all commands
- No trailing whitespace or double blank lines
- Consistent indentation

### Step F.4 — Command Verification
Pick 1–2 documented commands (e.g., test, lint) and run them via `Bash` to verify they work. If a command fails, fix or annotate it in the CLAUDE.md.

### Step F.5 — Write Final Version
`Write` the final CLAUDE.md.

### Step F.6 — Summary
Present a summary to the user:

```
CLAUDE.md generation complete.

Iterations run:    {count}
Archetypes used:   {list of archetype names}
Sections:          {list of section names}
Final line count:  {count}
```

If `generate_rules` was enabled, also list:
```
Generated rule files:
- .claude/rules/{topic1}.md
- .claude/rules/{topic2}.md
```

---

## Important Reminders

- **NEVER execute invented tasks.** You only plan them to find gaps.
- **NEVER include information Claude can get from reading code.** The CLAUDE.md is for things that AREN'T in the code.
- **Be ruthless about pruning.** A short, dense CLAUDE.md is far more valuable than a long, padded one.
- **Each iteration should discover NEW information.** If an iteration finds nothing new, that's a signal you may have enough iterations.
- **Reference real files.** Every task you invent must reference actual files and modules in the repository.
