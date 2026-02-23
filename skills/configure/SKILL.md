---
name: configure
description: Configure settings for the CLAUDE.md generator — iteration count, interaction mode, focus areas, and output preferences.
allowed-tools: Read, Write, Edit, AskUserQuestion, Glob
---

# Configure CLAUDE.md Generator

You are the configuration skill for the CLAUDE.md generator plugin. Your job is to create or update a `.claude-md-config.json` file in the project root with the user's preferences.

## Step 1: Check for Existing Config

Use `Glob` to check for an existing `.claude-md-config.json` in the project root.

If found, read it with `Read` and display the current settings to the user:

```
Current CLAUDE.md generator settings:
- Iterations: {iterations}
- Interaction mode: {interaction_mode}
- Focus areas: {focus_areas}
- Max lines: {max_lines}
- Generate rules files: {generate_rules}
```

If not found, inform the user you'll create a new config with defaults.

## Step 2: Configure Each Setting

Use `AskUserQuestion` to walk through each setting. Present all settings in a single question block when possible.

### Setting: iterations
- Question: "How many iterations should the generator run?"
- Options: `3` (Quick scan), `5` (Balanced — recommended), `8` (Thorough), `10` (Exhaustive)
- Default: `5`

### Setting: interaction_mode
- Question: "How much control do you want during generation?"
- Options:
  - `autonomous` — Run all iterations without stopping. Best for trusted repos you know well.
  - `confirm-base` (Recommended) — Review and edit the initial CLAUDE.md before iterations begin, then run autonomously.
  - `confirm-each` — Pause after every iteration to review changes. Most control, most interactive.
- Default: `confirm-base`

### Setting: focus_areas
- Question: "Which areas should the generator focus on?"
- Multi-select with options: `commands`, `architecture`, `testing`, `style`, `gotchas`, `workflow`
- Also offer `all` as a single option meaning all areas
- Default: `all`

### Setting: max_lines
- Question: "What's the maximum line count for the generated CLAUDE.md?"
- Options: `50` (Minimal), `100` (Concise), `150` (Balanced — recommended), `300` (Comprehensive)
- Default: `150`

### Setting: generate_rules
- Question: "Should the generator also create `.claude/rules/` files for detailed topics?"
- Options:
  - `true` — Extract long sections into separate `.claude/rules/` files (auto-loaded by Claude Code)
  - `false` (Recommended) — Keep everything in a single CLAUDE.md
- Default: `false`

## Step 3: Write Config File

Write the config as `.claude-md-config.json` in the project root:

```json
{
  "iterations": 5,
  "interaction_mode": "confirm-base",
  "focus_areas": "all",
  "max_lines": 150,
  "generate_rules": false
}
```

Use `Write` if creating new, or `Edit` if updating an existing config.

## Step 4: Confirm

Display the final config to the user:

```
CLAUDE.md generator configured:
  Iterations:       {iterations}
  Interaction mode:  {interaction_mode}
  Focus areas:       {focus_areas}
  Max lines:         {max_lines}
  Generate rules:    {generate_rules}

Run /setup-claude-md:generate to start generating your CLAUDE.md.
```
