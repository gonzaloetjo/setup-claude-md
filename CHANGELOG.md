# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0](https://github.com/gonzaloetjo/setup-claude-md/compare/v1.0.0...v1.1.0) (2026-02-23)


### Features

* drop Phase 1 repo scanning, require /init before generate ([a6794eb](https://github.com/gonzaloetjo/setup-claude-md/commit/a6794eb5d052f635208e366be7eae28afdf98404))
* drop Phase 1 repo scanning, require /init before generate ([da5518d](https://github.com/gonzaloetjo/setup-claude-md/commit/da5518ded1e668349a39d89ac6ac74b24d0f8711))


### Bug Fixes

* correct plugin.json schema and restore marketplace.json ([7dda936](https://github.com/gonzaloetjo/setup-claude-md/commit/7dda9369d61116ca30f31e3f631ae8cea5a884cc))
* correct plugin.json schema and restore marketplace.json ([faf0971](https://github.com/gonzaloetjo/setup-claude-md/commit/faf0971f1dcd3312754fc11c53ee3c09f6260934))

## [1.0.0] - 2026-02-23

### Added
- Iterative CLAUDE.md generation with agent-based stress-test refinement
- 10 task archetypes (bug fix, test addition, new feature, refactor, API change, dependency update, CI/CD, perf, security, cross-cutting)
- Configurable iterations (1-10), interaction modes (autonomous, confirm-base, confirm-each), focus areas, max lines
- Resumable generation via state file
- Final validation agent with removal test and command spot-checking
- `/setup-claude-md:generate` and `/setup-claude-md:configure` skills
