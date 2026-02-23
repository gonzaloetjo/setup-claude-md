# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - Unreleased

### Changed
- **Removed Phase 1 (repository scanning).** The plugin now requires an existing CLAUDE.md (e.g., from `/init` or manual creation) instead of generating one from scratch. The plugin's value is the iteration loop, not the initial scan.
- Phase 0 now includes a hard gate: if no CLAUDE.md exists, the user is told to run `/init` first.
- `repo_summary` is now synthesized from the existing CLAUDE.md content instead of a full repo scan.
- Phases renumbered: iteration loop is Phase 1–N (was Phase 2–N), review is Phase 2 (was Phase Final), cleanup is Phase 3 (was Phase Cleanup).

## [1.0.0] - 2026-02-23

### Added
- Iterative CLAUDE.md generation with agent-based stress-test refinement
- 10 task archetypes (bug fix, test addition, new feature, refactor, API change, dependency update, CI/CD, perf, security, cross-cutting)
- Configurable iterations (1-10), interaction modes (autonomous, confirm-base, confirm-each), focus areas, max lines
- Resumable generation via state file
- Final validation agent with removal test and command spot-checking
- `/setup-claude-md:generate` and `/setup-claude-md:configure` skills
