---
name: generate
description: >
  Iteratively refines an existing CLAUDE.md file using stress-test refinement
  with sequential agents. Spawns a fresh agent for each iteration — each gets a clean context window.
  Requires an existing CLAUDE.md (run /init first). Run /setup-claude-md:configure to set options.
argument-hint: "[iterations=5]"
allowed-tools: Read, Glob, Grep, Bash, Write, Edit, Task, AskUserQuestion
---

# Generate CLAUDE.md — Iterative Stress-Test Refinement

You are the orchestrator for CLAUDE.md generation. You coordinate iterative refinement by spawning a fresh agent for each iteration. Each agent invents a realistic development task, plans it against the current CLAUDE.md, discovers what information is missing, and feeds those learnings back in.

**You never do iteration work yourself.** You handle setup, agent spawning, interaction gates, and the final summary. The agents do the analytical work.

**Prerequisite:** A CLAUDE.md must already exist in the project root (e.g., from `/init` or manual creation). This skill refines an existing CLAUDE.md — it does not generate one from scratch.

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
   - If "Resume": skip to Phase 1–N, starting from iteration `completed_iterations + 1`, using the saved config and repo_summary
   - If "Start fresh": delete the state file and logs directory, continue from step 5

4. **Load quality standards.** `Read` the file `references/claude-md-standards.md` (relative to this skill file). Keep these criteria in mind throughout.

5. **Require CLAUDE.md.** Use `Glob` to find `CLAUDE.md` in the project root. If it does NOT exist: tell the user "No CLAUDE.md found. Run `/init` first to create a base, then re-run `/setup-claude-md:generate`." and **STOP**.

6. **Read & synthesize repo summary.** `Read` the existing CLAUDE.md. Synthesize a one-sentence `repo_summary` from its content (tech stack, project type, key directories). Example: "TypeScript/React app using Vite, pnpm, Vitest. Dirs: src/, tests/, docs/."

7. **Write state file.** Write the initial state:

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

8. **Interaction gate.** Check the `interaction_mode` setting:
   - If `confirm-base` or `confirm-each`: Present the existing CLAUDE.md to the user using `AskUserQuestion`. Ask: "I'll refine this CLAUDE.md through {N} iterations of stress-testing. Proceed?" Provide options: "Looks good, proceed", "I have feedback" (let them type).
   - If `autonomous`: Continue without stopping.

---

## Phase 1–N: The Iteration Loop

For each iteration `i` from 1 to `iterations` (or from `completed_iterations + 1` if resuming):

Announce: "**Iteration {i}/{total} — {archetype_name}**"

### Step 1.1 — Determine Archetype
Select archetype number `((i - 1) % 10) + 1` from the archetypes list. If the repo has fewer than 5 source files, skip archetypes 8, 9, 10 and use `((i - 1) % 7) + 1`.

Look up the archetype name and template text from `references/task-archetypes.md`.

### Step 1.2 — Determine Complexity Guidance
- Iterations 1–2: "Single-module scope. Task touches 1–2 files."
- Iterations 3–4 (or 3–5): "Multi-file scope. Task spans 3–5 files across 2+ directories."
- Iterations 5+ (or 6+): "Cross-cutting scope. Task affects a subsystem or requires coordinated changes."

### Step 1.3 — Construct Agent Prompt
`Read` the file `references/iteration-prompt.md` (relative to this skill file). Fill in all `{placeholder}` values:

- `{iteration}` — current iteration number
- `{total}` — total iterations
- `{archetype_name}` — name from Step 1.1 (e.g., "Bug Fix (Low Complexity)")
- `{archetype_template}` — template and example text from the archetypes file for this archetype
- `{complexity_guidance}` — from Step 1.2
- `{claude_md_path}` — absolute path to the project's CLAUDE.md
- `{standards_path}` — absolute path to `references/claude-md-standards.md`
- `{archetypes_path}` — absolute path to `references/task-archetypes.md`
- `{log_path}` — absolute path to `.claude-md-generator-logs/iteration-{i}.md`
- `{focus_areas}` — from config
- `{max_lines}` — from config
- `{generate_rules}` — from config
- `{repo_summary}` — from state file

### Step 1.4 — Spawn Iteration Agent
Spawn the agent using the `Task` tool:
```
Task(
  subagent_type: "general-purpose",
  description: "Refinement iteration {i}",
  prompt: <the constructed prompt from Step 1.3>
)
```

### Step 1.5 — Post-Iteration Bookkeeping
After the agent completes:
1. `Read` the iteration log at `.claude-md-generator-logs/iteration-{i}.md`
2. Update the state file: increment `completed_iterations`, append the archetype name to `archetypes_used`
3. Write the updated state file

### Step 1.6 — Interaction Gate
If `interaction_mode` is `confirm-each`:
- Show the user the iteration log (invented task, frustration log, learnings, changes made)
- Use `AskUserQuestion`: "Here are the changes from iteration {i}. Accept, modify, or skip?" Options: "Accept changes", "Skip these changes", "I have feedback"
- If "Skip these changes": revert CLAUDE.md to the version before this iteration (re-read the previous version from iteration `i-1`'s log or the original if `i == 1`)

---

## Phase 2: Review Agent

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

## Phase 3: Cleanup

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
