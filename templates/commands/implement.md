---
description: Execute implementation following the plan and tasks
---

# Implement - Execute Feature Implementation

**Implementation Target**: $ARGUMENTS

## Capability Mode Detection

**The script automatically detects your current workflow:**

- **Parent feature branch** (`username/jira-123.feature-name`):
   - Reads from: `specs/jira-123.feature-name/plan.md`, `tasks.md`
   - Implementation: Single PR workflow

- **Capability branch** (`username/jira-123.feature-name-cap-001`):
   - Reads from: `specs/jira-123.feature-name/cap-001-auth/plan.md`, `tasks.md`
   - Implementation: Atomic PR workflow (typical: 800-1500 LOC)
   - PR target: `cap-001` branch → `main` (not to parent branch)

**No flag needed** - detection is automatic based on branch name pattern.

See "Capability PR Workflow (Atomic PRs)" section below for detailed workflow.

---

## Pre-Implementation Validation

1. **Run prerequisite check**: `scripts/bash/check-implementation-prerequisites.sh --json` from repo root
   - Parse FEATURE_DIR, PLAN_PATH, TASKS_PATH, BRANCH
   - Verify plan.md and tasks.md exist
   - Check for constitutional compliance markers

2. **Load implementation context**:
   - **REQUIRED**: Read `memory/constitution.md` for constitutional requirements
   - **REQUIRED**: Read `tasks.md` for complete task list and execution order
   - **REQUIRED**: Read `plan.md` for tech stack, architecture, and validation gates
   - **IF EXISTS**: Read `data-model.md` for entities and relationships
   - **IF EXISTS**: Read `contracts/` for API specifications and contract tests
   - **IF EXISTS**: Read `research.md` for technical decisions and constraints
   - **IF EXISTS**: Read `quickstart.md` for integration test scenarios

3. **Parse task structure**:
   - Extract task phases: Setup, Tests, Core, Integration, Polish
   - Identify task dependencies and [P] parallel markers
   - Build execution graph respecting dependencies
   - Validate TDD ordering (tests before implementation)

## Phase 0: Setup & Initialization

### Pre-Implementation Checklist

Before proceeding, verify:
- [ ] Library-first architecture confirmed in plan.md
- [ ] CLI interface design documented
- [ ] Test approach validated in tasks.md
- [ ] Observability requirements identified

### Project Setup Tasks
- Initialize project structure per plan.md
- Install dependencies and dev tools
- Configure linters, formatters, and git hooks
- Create base library structure with CLI scaffolding
- Run: `/validate plan` to ensure readiness

## Phase 1: Test-First Development (RED Phase)

Tests should be written before implementation when practical. Consider:
- What test scenarios validate the core business value?
- Which tests provide the most informative feedback during development?

### Test Directory Structure

```
tests/
├── contract/       # Contract tests
├── unit/           # Unit tests
├── integration/    # Integration tests
├── functional/     # Functional tests
└── conftest.py     # Shared fixtures
```

### Test Creation Order
1. **Contract Tests** [P] - From contracts/*.md → `tests/contract/`
2. **Entity/Model Tests** [P] - Per data model → `tests/unit/`
3. **Integration Tests** - By dependency → `tests/integration/`

After writing tests, verify they fail appropriately before implementation.

## Phase 2: Core Implementation (GREEN Phase)

### Implementation Rules

- Write minimal code to make tests pass
- Follow patterns from ai_docs/ and existing code
- Respect [P] markers for parallel execution

**Execution Flow**:
1. **Models/Entities** [P] - Implement data structures
   - Create model files per data-model.md
   - Add validation logic
   - Run tests: expect some to pass
   - Commit: `/smart-commit "feat: implement [entity] models"`

2. **Core Services** - Sequential implementation
   - Implement business logic services
   - Add error handling and logging
   - Follow existing service patterns
   - Commit: `/smart-commit "feat: implement [service] logic"`

3. **API/CLI Endpoints** [P] - Implement interfaces
   - Create endpoint handlers
   - Wire up to services
   - Add input validation
   - Commit: `/smart-commit "feat: implement [endpoint] handlers"`

4. **Integration Layer** - Connect components
   - Wire up dependencies
   - Configure middleware
   - Set up database connections
   - Commit: `/smart-commit "feat: integrate [component] layers"`

**Validation Gate**:
```bash
# All existing tests should now pass
npm test || python -m pytest || go test ./...
/validate implementation
```

## Phase 3: Refactor & Polish (REFACTOR Phase)

- Refactor for clarity without changing behavior
- Extract common patterns to utilities
- Optimize performance bottlenecks
- Add edge case tests and verify coverage

Run `/validate implementation` when complete.

## Phase 4: Documentation & Delivery

### Documentation Tasks
- [ ] Update README with usage examples
- [ ] Document API in OpenAPI/Swagger format
- [ ] Add inline code documentation
- [ ] Create architecture decision records (ADRs)
- [ ] Update CHANGELOG.md

### Smart Commit Integration
```bash
# Create comprehensive commit for the feature
/smart-commit "feat: complete implementation of [feature-name]

- Implemented [key components]
- Added comprehensive test coverage
- Follows constitutional principles
- Validates against all quality gates"
```

## Capability PR Workflow (Atomic PRs)

### If on capability branch (e.g., `username/jira-123.feature-cap-001`):

1. **Review scope**: Run `git diff main --stat` to see total changes

2. **Create PR to main**:
   ```bash
   gh pr create --base main --title "feat(cap-001): [capability description]" \
     --body "$(cat <<'EOF'
   ## Summary
   Implements Cap-001: [capability name] from [parent feature]

   - [Key component 1]
   - [Key component 2]
   - [Key component 3]

   ## Dependencies
   - Depends on: [cap-XXX if any]
   - Enables: [cap-YYY that depend on this]

   ## Test Coverage
   - All tests passing: ✓

   Part of parent feature: specs/[jira-123.feature-name]/
   EOF
   )"
   ```

3. **After PR approval and merge**:
   ```bash
   # Switch back to parent branch
   git checkout [username]/[jira-123.feature-name]

   # Pull latest main to sync merged changes
   git pull origin main

   # Optional: delete local capability branch
   git branch -d [username]/[jira-123.feature-name]-cap-001
   ```

4. **Repeat for next capability**:
   ```bash
   # Start next capability
   /plan --capability cap-002 "Next capability tech details"
   # Creates new branch: username/jira-123.feature-name-cap-002
   # Repeat implement → PR → merge cycle
   ```

### Benefits of Capability PR Workflow:
- **Fast reviews**: Smaller PRs reviewed faster
- **Parallel development**: Multiple team members work on different capabilities simultaneously
- **Early integration**: Merge to main quickly, catch integration issues early
- **Clear ownership**: Each PR has focused scope and clear acceptance criteria

## Error Handling & Recovery

### On Test Failure (Phase 1)
- Expected behavior - tests should fail initially
- If tests pass without implementation, review test quality
- Ensure tests actually test business requirements

### On Implementation Failure (Phase 2)
- Run targeted validation: `/validate implementation [failing-component]`
- Check ai_docs/ for library-specific gotchas
- Review existing patterns for similar implementations
- Use `/debug` command if systematic issues persist

### On Validation Failure (Any Phase)
- Stop implementation immediately
- Run `/validate --verbose` for detailed diagnostics
- Address all critical issues before proceeding
- Document any constitutional exceptions needed

## Parallel Execution Support

Tasks marked [P] can be executed concurrently:
```bash
# Example parallel execution for test creation
parallel ::: \
  "create test-auth.py" \
  "create test-user.py" \
  "create test-permissions.py"
```

## Progress Tracking

Use TodoWrite tool throughout:
- Mark task as in_progress when starting
- Mark completed immediately upon finishing
- Update with blockers if encountered
- Add new tasks discovered during implementation

## Recommended Practices

- Write tests before implementation when practical
- Implement features as libraries with CLI interfaces
- Include structured logging for observability

## Success Criteria

Implementation is complete when:
- [ ] All tasks from tasks.md are marked completed
- [ ] All tests pass (unit, integration, contract)
- [ ] `/validate implementation` passes all gates
- [ ] Code review complete with no blockers
- [ ] Documentation updated and accurate
- [ ] Smart commits document the journey
