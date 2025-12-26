---
description: Identify underspecified areas in the current feature spec by asking up to 5 highly targeted clarification questions and encoding answers back into the spec.
scripts:
   sh: scripts/bash/check-prerequisites.sh --json --paths-only
   ps: scripts/powershell/check-prerequisites.ps1 -Json -PathsOnly
---

## User Input

```text
$ARGUMENTS
```

Consider the user input before proceeding (if not empty).

## Outline

Goal: Detect and reduce ambiguity or missing decision points in the active feature specification and record the clarifications directly in the spec file.

Note: This clarification workflow is expected to run (and be completed) BEFORE invoking `/plan`. If the user explicitly states they are skipping clarification (e.g., exploratory spike), you may proceed, but should note that downstream rework risk increases.

Execution steps:

1. Run `{SCRIPT}` from repo root **once**. Parse minimal JSON payload fields:
   - `FEATURE_DIR`
   - `FEATURE_SPEC`
   - If JSON parsing fails, abort and instruct user to re-run `/specify` or verify feature branch environment.

2. Load the current spec file. Perform a structured ambiguity & coverage scan using this taxonomy. For each category, mark status: Clear / Partial / Missing.

   **Functional Scope & Behavior:**
   - Core user goals & success criteria
   - Explicit out-of-scope declarations
   - User roles / personas differentiation

   **Domain & Data Model:**
   - Entities, attributes, relationships
   - Identity & uniqueness rules
   - Lifecycle/state transitions
   - Data volume / scale assumptions

   **Interaction & UX Flow:**
   - Critical user journeys / sequences
   - Error/empty/loading states
   - Accessibility or localization notes

   **Non-Functional Quality Attributes:**
   - Performance (latency, throughput targets)
   - Scalability (horizontal/vertical, limits)
   - Reliability & availability
   - Observability (logging, metrics, tracing)
   - Security & privacy
   - Compliance / regulatory constraints

   **Integration & External Dependencies:**
   - External services/APIs and failure modes
   - Data import/export formats
   - Protocol/versioning assumptions

   **Edge Cases & Failure Handling:**
   - Negative scenarios
   - Rate limiting / throttling
   - Conflict resolution

   **Constraints & Tradeoffs:**
   - Technical constraints
   - Explicit tradeoffs or rejected alternatives

   **Terminology & Consistency:**
   - Canonical glossary terms
   - Avoided synonyms / deprecated terms

   **Completion Signals:**
   - Acceptance criteria testability
   - Measurable Definition of Done indicators

   **Misc / Placeholders:**
   - TODO markers / unresolved decisions
   - Ambiguous adjectives lacking quantification

3. Generate a prioritized queue of candidate clarification questions (maximum 5). Apply these constraints:
    - Each question must be answerable with EITHER:
       - A short multiple-choice selection (2-5 options), OR
       - A one-word / short-phrase answer (<=5 words)
    - Only include questions whose answers materially impact architecture, data modeling, task decomposition, test design, UX behavior, operational readiness, or compliance validation
    - Favor clarifications that reduce downstream rework risk

4. Sequential questioning loop (interactive):
    - Present EXACTLY ONE question at a time
    - For multiple-choice questions:
       - Analyze all options and determine the most suitable option
       - Present your recommended option prominently with clear reasoning
       - Format as: `**Recommended:** Option [X] - <reasoning>`
       - Then render all options as a Markdown table
       - After the table, add: `You can reply with the option letter, accept the recommendation by saying "yes", or provide your own short answer.`
    - For short-answer style:
       - Provide your suggested answer based on best practices
       - Format as: `**Suggested:** <your proposed answer> - <brief reasoning>`
    - After the user answers:
       - If the user replies with "yes" or "recommended", use your previously stated recommendation
       - Validate the answer fits constraints
       - Record it in working memory and move to the next question
    - Stop asking when:
       - All critical ambiguities resolved, OR
       - User signals completion ("done", "good", "no more"), OR
       - You reach 5 asked questions

5. Integration after EACH accepted answer:
    - Ensure a `## Clarifications` section exists
    - Under it, create `### Session YYYY-MM-DD` subheading for today
    - Append: `- Q: <question> → A: <final answer>`
    - Apply the clarification to the most appropriate section(s)
    - Save the spec file AFTER each integration

6. Validation (performed after EACH write):
   - Clarifications session contains exactly one bullet per accepted answer
   - Total asked questions ≤ 5
   - Updated sections contain no lingering vague placeholders
   - Markdown structure valid

7. Write the updated spec back to `FEATURE_SPEC`.

8. Report completion:
   - Number of questions asked & answered
   - Path to updated spec
   - Sections touched
   - Coverage summary table
   - Suggested next command

Behavior rules:

- If no meaningful ambiguities found, respond: "No critical ambiguities detected." and suggest proceeding
- If spec file missing, instruct user to run `/specify` first
- Never exceed 5 total asked questions
- Respect user early termination signals ("stop", "done", "proceed")

Context for prioritization: {ARGS}
