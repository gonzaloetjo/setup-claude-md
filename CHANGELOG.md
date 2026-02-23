# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-02-23

### Added
- Iterative CLAUDE.md generation with agent-based stress-test refinement
- 10 task archetypes (bug fix, test addition, new feature, refactor, API change, dependency update, CI/CD, perf, security, cross-cutting)
- Configurable iterations (1-10), interaction modes (autonomous, confirm-base, confirm-each), focus areas, max lines
- Resumable generation via state file
- Final validation agent with removal test and command spot-checking
- `/setup-claude-md:generate` and `/setup-claude-md:configure` skills
