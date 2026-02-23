# Development Task Archetypes

Use these archetypes to generate realistic development tasks during each iteration. Each iteration MUST use a different archetype.

## The 10 Archetypes

### 1. Bug Fix (Low Complexity)
**Template:** "Fix a bug where [component] does [wrong behavior] when [condition]."
**Example:** "Fix a bug where the UserProfile component shows stale data when navigating back from the settings page."
**What this stress-tests:** Error handling patterns, debugging workflow, test commands, related modules.

### 2. Test Addition (Low Complexity)
**Template:** "Add unit tests for [module/function] covering [edge cases]."
**Example:** "Add unit tests for the parseConfig utility covering malformed input, missing required fields, and env var interpolation."
**What this stress-tests:** Test framework, test file location, naming conventions, mocking patterns, coverage expectations.

### 3. New Feature (Medium Complexity)
**Template:** "Add [feature] to [module] that [behavior]."
**Example:** "Add a rate limiter to the API gateway that limits requests to 100/minute per API key."
**What this stress-tests:** Where new code goes, architectural patterns, naming conventions, integration with existing modules.

### 4. Refactor (Medium Complexity)
**Template:** "Refactor [module] to use [pattern] instead of [current approach]."
**Example:** "Refactor the database layer to use the repository pattern instead of inline SQL queries."
**What this stress-tests:** Architecture boundaries, code organization, import patterns, module interfaces.

### 5. API Change (Medium Complexity)
**Template:** "Add/modify [endpoint/interface] to support [capability]."
**Example:** "Add a PATCH endpoint to /api/users/:id to support partial profile updates with field-level validation."
**What this stress-tests:** API conventions, validation patterns, serialization, route organization, middleware chain.

### 6. Dependency Update (Medium Complexity)
**Template:** "Update [dependency] from [old version] to [new version] and fix breaking changes."
**Example:** "Update React Router from v5 to v6 and migrate all route definitions and navigation hooks."
**What this stress-tests:** Dependency management, build commands, breaking change handling, test verification.

### 7. CI/CD Change (Medium Complexity)
**Template:** "Add/modify [CI step] to [purpose]."
**Example:** "Add a GitHub Actions workflow step that runs E2E tests against a staging database before merging to main."
**What this stress-tests:** CI/CD configuration, environment variables, secrets, deployment workflow, branch conventions.

### 8. Performance Optimization (High Complexity)
**Template:** "Optimize [slow operation] in [module]."
**Example:** "Optimize the dashboard query that aggregates user activity data — it currently takes 3s for large accounts."
**What this stress-tests:** Profiling approach, database patterns, caching strategy, benchmark methodology.

### 9. Security Hardening (High Complexity)
**Template:** "Fix [vulnerability type] in [component]."
**Example:** "Fix the SQL injection vulnerability in the search endpoint's filter parameter handling."
**What this stress-tests:** Security patterns, input validation, auth/authz conventions, security testing approach.

### 10. Cross-Cutting Concern (High Complexity)
**Template:** "Add [logging/monitoring/auth/caching] across [subsystem]."
**Example:** "Add structured logging with request correlation IDs across all API handlers."
**What this stress-tests:** Cross-cutting patterns, shared utilities, configuration management, consistency rules.

## Generation Rules

### Archetype Selection
- Each iteration MUST use a different archetype
- Cycle through in order: 1, 2, 3, ... 10, then wrap back to 1
- For repos with fewer than 5 source files, skip archetypes 8, 9, and 10 (high complexity) and cycle through 1–7

### Complexity Escalation
- **Iterations 1–2:** Single-module scope. Task touches 1–2 files.
- **Iterations 3–5:** Multi-file scope. Task spans 3–5 files across 2+ directories.
- **Iterations 6+:** Cross-cutting scope. Task affects a subsystem or requires coordinated changes.

### Task Specificity
- Tasks MUST reference real files, real modules, and real components from the existing CLAUDE.md and repo
- Task description: exactly 2–3 sentences
- Include enough detail to plan concretely — name the files, the functions, the expected behavior
- Avoid hypothetical or generic tasks. If the repo has no API, don't generate an API task

## Observation Checklist

While planning each task (without executing it), note every time you need information that isn't in the current CLAUDE.md:

- [ ] What build/test/lint commands are needed?
- [ ] What's the file naming convention for new files of this type?
- [ ] What's the import/module pattern (relative vs absolute, barrel files, etc.)?
- [ ] Are there code style rules not captured by linter config?
- [ ] What's the error handling pattern (exceptions, Result types, error codes)?
- [ ] What's the testing convention (file location, naming, framework, assertion style)?
- [ ] Are there architectural boundaries that shouldn't be crossed?
- [ ] What environment variables or config is needed?
- [ ] What are the PR/commit message conventions?
- [ ] What domain-specific terminology is used and what does it mean?
