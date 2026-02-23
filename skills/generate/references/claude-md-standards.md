# CLAUDE.md Quality Standards

## Quality Criteria

- **Target length**: 50–150 lines. Hard cap: 300 lines. (Anthropic's official ceiling is ~500 lines, but CLAUDE.md loads as user messages — bloated files cause instructions to get lost. Aim for the range where every line earns its place.)
- **Every line must be actionable** — something Claude should DO or AVOID when working in this repo.
- **Nothing Claude can infer from reading code.** If a convention is enforced by a linter, a type system, or is obvious from file structure, don't document it.
- **Use specific language.** Write "Use ES modules (`import`/`export`), never CommonJS `require()`" — not "follow modern patterns."
- **Reserve NEVER, MUST, IMPORTANT** for rules Claude actively tends to violate. Overuse dilutes their impact.
- **Commands must be exact.** Include flags, env vars, and working directory if non-obvious. Wrap in code blocks.
- **Tag deterministic rules with `[deterministic]`.** If a rule describes an action that must happen every time with zero exceptions (e.g., "always run lint after editing"), append `[deterministic]` to the line. These are candidates for conversion to Claude Code hooks.
- **No aspirational content.** Document what IS, not what SHOULD BE. If the codebase doesn't follow a pattern consistently, don't claim it does.

## Section Templates

### Project Overview
1–2 sentences max. Follow this pattern:
> This is a [framework] [type] app that [core purpose]. It uses [key architectural choice].

### Tech Stack
Language version, framework, and key dependencies only. Skip transitive dependencies.

### Commands
Exact commands in fenced code blocks. Include:
- Build (dev and production)
- Test (all, single file, single test, watch mode)
- Lint / format (with auto-fix variants)
- Run / serve (dev and production)
- Deploy (if not CI-only)
- Any non-obvious setup steps

### Code Style & Conventions
Only document deviations from language/framework defaults:
- Naming conventions that differ from ecosystem norms
- File organization rules not obvious from directory structure
- Import ordering if enforced but not by a tool
- Error handling patterns specific to this project

### Architecture
Keep brief. Focus on non-obvious decisions:
- Why a monorepo (or not)
- Module boundary rules
- Data flow patterns that aren't standard for the framework
- Where shared code lives and how it's accessed

### Gotchas
Format as imperative warnings:
- "NEVER do X because Y happens"
- "IMPORTANT: X requires Y first"
- "After changing X, you MUST also update Y"

If the project has critical context that must survive long-session compaction, add as the last Gotchas item:
> When compacting, always preserve [specific critical context].
Use sparingly — only when losing this context would cause real mistakes.

## Anti-Patterns — Do NOT Include

- **File-by-file codebase descriptions.** Claude can read the file tree.
- **Inline code snippets or duplicated docs.** Use `@path/to/file` imports to reference existing documentation (README.md, CONTRIBUTING.md) rather than duplicating content. Use `file:line` references for specific code locations. Never paste code verbatim.
- **Generic wisdom.** "Write clean code", "follow best practices", "keep functions small" — useless.
- **Detailed API documentation.** Link to docs or reference the source file.
- **Over-specified workflows.** Don't write a 20-step PR checklist. Summarize what's non-obvious.
- **Dependency lists.** Claude can read package.json / requirements.txt / go.mod.
- **Type definitions or interface descriptions.** Claude reads the source.
- **Obvious directory roles.** Don't document that `src/` contains source code.
- **Personal or local-only preferences.** Sandbox URLs, personal test data, local machine quirks belong in `CLAUDE.local.md` (auto-gitignored), not in the shared CLAUDE.md.

## The Pruning Heuristic

Apply this test to every line in the CLAUDE.md:

> "If I deleted this line, would Claude make a concrete, observable mistake on a real development task?"

- **Clear yes** → Keep the line.
- **Maybe / probably not** → Delete it.
- **It's just nice to know** → Delete it.

Ruthlessly prune. A 50-line CLAUDE.md where every line matters is better than a 200-line one with filler.

## Length Management

When the CLAUDE.md exceeds `max_lines`:
1. First, prune using the heuristic above
2. Merge entries that convey the same information
3. If still over, move the longest section to `.claude/rules/<topic>.md` and add an `@rules/<topic>.md` import (only if `generate_rules` is enabled). When creating rules files, add YAML frontmatter with `paths:` to scope them to relevant file patterns (e.g., testing conventions → `paths: ["**/*.test.*"]`). Also consider whether content already exists in README.md or CONTRIBUTING.md — use `@README.md` imports rather than duplicating.
4. As a last resort, remove "helpful" items and keep only "critical" ones
