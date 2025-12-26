---
description: Decompose parent feature spec into atomic capabilities for manageable PRs
scripts:
  sh: scripts/bash/decompose-feature.sh --json
  ps: scripts/powershell/decompose-feature.ps1 -Json
---

# Decompose - Break Feature into Atomic Capabilities

Given a parent feature specification, decompose it into independently-implementable capabilities.

## Pre-Decomposition Validation

1. **Verify parent spec exists**:
   - Run `{SCRIPT}` from repo root
   - Parse JSON for SPEC_PATH, CAPABILITIES_FILE, SPEC_DIR
   - Confirm parent spec.md is complete (no [NEEDS CLARIFICATION] markers)

2. **Load parent specification**:
   - Read parent spec.md
   - Extract all functional requirements (FR-001, FR-002, ...)
   - Identify key entities and user scenarios
   - Understand dependencies and constraints

3. **Load constitution** at `/memory/constitution.md` for constitutional requirements

## Decomposition Process

### Phase 1: Analyze & Group Requirements

**Consider decomposition strategy:**
- What are the natural bounded contexts in this feature?
- How can requirements be grouped to maximize independence?
- Which capabilities are foundational (enable others)?
- What dependencies exist between capability groups?

**Grouping Strategies:**
1. **By Entity Lifecycle**: User CRUD, Project CRUD, Report CRUD
2. **By Workflow Stage**: Registration → Auth → Profile → Settings
3. **By API Cluster**: 3-5 related endpoints that share models/services
4. **By Technical Layer**: Data layer → Service layer → API layer (vertical slices preferred)

**Analyze functional requirements:**
- Group related FRs into bounded contexts
- Identify foundation requirements (infrastructure, base models)
- Map dependencies between requirement groups
- Estimate complexity per group

### Phase 2: Estimate Scope Per Capability

**For each identified group, assess:**
- Models: [count] entities with validation
- Services: [count] operations/use cases
- API/CLI: [count] endpoints/commands
- Tests: Contract + integration coverage

**Target per capability:**
Aim for capabilities that can be reviewed in a single PR session.

**Sizing guidance:**
- **Small:** 1-2 components, focused scope, straightforward review
- **Medium:** 3-4 components, clear boundaries, manageable review
- **Too small:** Consider merging with related capability
- **Too large:** Consider further decomposition

### Phase 3: Order Capabilities

**Ordering criteria:**
1. **Infrastructure dependencies**: Database/storage → Services → APIs
2. **Business value**: High-value capabilities first (demonstrate value early)
3. **Technical risk**: Foundation/risky components early (de-risk fast)
4. **Team parallelization**: Independent capabilities can be developed concurrently

**Create dependency graph:**
- Identify foundation capabilities (no dependencies)
- Map capability-to-capability dependencies
- Validate no circular dependencies
- Identify parallel execution opportunities

### Phase 4: Generate Capability Breakdown

1. **Fill capabilities.md template using UTF-8 encoding**:
   - Load CAPABILITIES_FILE (already created by script)
   - Fill each capability section:
     - Cap-001, Cap-002, ... Cap-00X
     - Scope description
     - Dependencies
     - Business value
     - Component breakdown (qualitative sizing)
     - Size assessment (Small | Medium)
   - Generate dependency graph
   - Document implementation strategy

2. **Create capability subdirectories**:
   ```bash
   For each capability (Cap-001 to Cap-00X):
     - Create directory: specs/[feature-id]/cap-00X-[name]/
     - Copy capability-spec-template.md to cap-00X-[name]/spec.md
     - Populate with scoped requirements from parent spec
   ```

3. **Populate scoped specs using UTF-8 encoding**:
   - For each capability directory, fill spec.md:
     - Link to parent spec
     - Extract relevant FRs from parent
     - Define capability boundaries (what's IN, what's OUT)
     - List dependencies on other capabilities
     - Scope user scenarios to this capability
     - Estimate component breakdown
     - Validate scope appropriate for single PR

### Phase 5: Validation

**Decomposition quality checks:**
- [ ] All capabilities: Reviewable PR size (small or medium)
- [ ] Each capability independently testable
- [ ] No circular dependencies
- [ ] All parent FRs assigned to a capability (no orphans)
- [ ] Total capabilities ≤10 (prevent over-decomposition)
- [ ] Foundation capabilities identified
- [ ] Parallel execution opportunities documented

**Capability independence checks:**
- [ ] Each capability delivers vertical slice (contract + model + service + tests)
- [ ] Each capability has clear interfaces with other capabilities
- [ ] Each capability can be merged independently (given dependencies met)
- [ ] Each capability has measurable acceptance criteria

## Output Artifacts

After decomposition, the feature directory should contain:

```
specs/[jira-123.feature-name]/
├── spec.md                      # Parent feature spec (unchanged)
├── capabilities.md              # Decomposition breakdown (NEW)
├── cap-001-[name]/              # First capability (NEW)
│   └── spec.md                 # Scoped to Cap-001
├── cap-002-[name]/              # Second capability (NEW)
│   └── spec.md                 # Scoped to Cap-002
├── cap-00X-[name]/              # Additional capabilities (NEW)
│   └── spec.md                 # Scoped to Cap-00X
```

## Next Steps

**For each capability (can be done in parallel where dependencies allow):**

1. **Plan**: `/plan --capability cap-001` → generates cap-001/plan.md (scoped for reviewable PR)
2. **Tasks**: `/tasks` → generates cap-001/tasks.md (8-15 tasks)
3. **Implement**: `/implement` → atomic PR
4. **Repeat** for cap-002, cap-003, etc.

## Example Workflow

```bash
# Step 1: Create parent spec (on branch: username/proj-123.user-system)
/specify "Build user management system with auth, profiles, and permissions"
→ Creates branch: username/proj-123.user-system
→ specs/proj-123.user-system/spec.md

# Step 2: Decompose into capabilities (on parent branch)
/decompose
→ specs/proj-123.user-system/capabilities.md
→ specs/proj-123.user-system/cap-001-auth/spec.md
→ specs/proj-123.user-system/cap-002-profiles/spec.md
→ specs/proj-123.user-system/cap-003-permissions/spec.md

# Step 3: Implement Cap-001 (creates NEW branch)
/plan --capability cap-001 "Use FastAPI + JWT tokens"
→ Creates branch: username/proj-123.user-system-cap-001
→ cap-001-auth/plan.md

/tasks
→ cap-001-auth/tasks.md (10 tasks)

/implement
→ Implement on cap-001 branch

# Create atomic PR to main
gh pr create --base main --title "feat(auth): Cap-001 authentication capability"
→ PR #1: cap-001 branch → main ✓ MERGED

# Step 4: Back to parent, sync, implement Cap-002
git checkout username/proj-123.user-system
git pull origin main

/plan --capability cap-002 "Use FastAPI + Pydantic models"
→ Creates branch: username/proj-123.user-system-cap-002
→ cap-002-profiles/plan.md

/tasks → /implement
→ PR #2: cap-002 branch → main ✓ MERGED

# Step 5: Repeat for cap-003...
# Each capability = separate branch + atomic PR
```

**Key Points:**
- Parent branch holds all capability specs
- Each capability gets its own branch from parent
- PRs go from capability branch → main (not to parent)
- After merge, sync parent with main before next capability

## Troubleshooting

**"Too many capabilities (>10)":**
- Validate each is truly independent
- Consider merging tightly-coupled capabilities
- Review if feature scope is too large (might need multiple parent features)

**"Capabilities too small":**
- Merge with related capabilities
- Ensure not over-decomposing simple features

**"Circular dependencies detected":**
- Review capability boundaries
- Extract shared components to foundation capability
- Reorder dependencies to break cycles

**"Cannot estimate scope accurately":**
- Start with component counts (will refine during /plan)
- Use similar features as reference
- Document uncertainty, adjust during planning phase

## Success Criteria

Decomposition is complete when:
- [ ] capabilities.md fully populated
- [ ] All capability directories created (cap-001/ through cap-00X/)
- [ ] Each capability has scoped spec.md
- [ ] All validation checks passed
- [ ] Dependency graph is acyclic
- [ ] Team understands implementation order
- [ ] Ready to run `/plan --capability cap-001`
