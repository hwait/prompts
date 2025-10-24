
# Feature implementation flow
## 1) Planning phase
### 1.1. Plan
```
[CONTEXT]
[PROJECT]=<Linear project key or name>
[REPO_ROOT]=<path or brief structure>

Goal: Turn this into a precise, build-ready plan.

Do:
1) Summarize the feature in 5–8 sentences.
2) Ask up to 10 clarifying questions. If unclear, propose safe defaults.
3) Define Acceptance Criteria as bullet points (testable).
4) Provide a 1-page ADR: Decision, Alternatives, Risks, Rollback.
5) Produce/patch data contracts (proto/OpenAPI/schemas), topic names, key fields, versions.
6) List exact files to create/change (paths), functions to touch, and the algorithms (step-by-step).
7) Observability plan: tracing spans, metrics, logs, dashboards, alerts.
8) Risk & rollout: feature flag, migrations, staged rollout, backout.

Then:
- Output a JSON "plan" with keys:
  {
    "summary": "...",
    "questions": [...],
    "acceptance_criteria": [...],
    "adr": "...",
    "contracts": {"proto": "...", "openapi": "...", "schemas": "..."},
    "changes": [{"path":"...", "action":"add|modify", "why":"..."}],
    "algorithms": ["..."],
    "observability": {"tracing": [...], "metrics": [...], "logs": [...]},
    "rollout": {"flag": "...", "migrations": "...", "stages": ["..."], "backout": "..."},
    "linear": {
      "story_title": "...",
      "story_description": "...",
      "subtasks": [
        {"title":"Phase 1: ...","description":"...","dod":[...],"labels":["..."]},
        {"title":"Phase 2: ...","description":"...","dod":[...],"labels":["..."]}
      ]
    }
  }

Constraints:
- Be specific. Name modules, crates, functions.
- Prefer Rust idioms (Tokio/Axum/Tonic, tracing, anyhow/thiserror).
- Don’t write code yet.
```
### 1.2. Follow-up
```
Using the JSON "plan.linear", create/update the Story and Subtasks in Linear under [PROJECT].
Set priority by impact. Link ADR and contract files. Return created issue IDs.
```
### 1.3. Another plan
Make a same prompt as 1.1. for another hogh-thinking AI

### 1.4. Review and concat 
```
 I need you to:
- review Linear ticket [Ticket] with subtasks [Subtask1], [Subtask2] and [Subtask3],
- adopt ideas/suggestions mentioned there into your plan,
- adjust those subtasks with your suggestions,
- add new subtasks if necessary, or just rewrite existing ones
```

### 1.5. Last review by first AI
```
Please review Linear ticket [Ticket] with subtasks [Subtask1], [Subtask2] and [Subtask3] for consistency, completeness, and the absence of logical errors so that those are ready for SDD tasks package formation.
```

## 2) Implementation preparation
### 2.1 Implementer plan (short + deterministic)
```
[StoryId]=<...> [SubtaskId]=<...>
Tasks:
1) Read both issues + comments. Mark [SubtaskId] In Progress.
2) Create branch: get the name as Linear suggestsed
3) Produce a step-by-step TODO plan for this subtask only. Include:
   - Files + functions to touch
   - Tests to add/run
   - Logging/tracing/metrics to add
   - CLI/env/config flags
   - Edge cases & error handling
   - Migration/compat plan (if any)
4) Post the plan as the FIRST comment of [SubtaskId] (overwrite if exists). Return the final checklist.
```
### 2.2. Reviewer (thinker) critique of plan
```
Scope: [StoryId], [SubtaskId]
Read both Linear MCP issues and FIRST comment in [SubtaskId] (the TODO plan).

Review for:
- Completeness vs Acceptance Criteria
- Correctness vs contracts/ADR
- Risks (concurrency, Kafka ordering, backpressure, timeouts)
- Observability sufficiency
- Test coverage (unit/integration/E2E)
- Performance and failure modes
- Simplicity (avoid over-engineering)

Output:
- "verdict": approve|changes_requested
- "mandatory_changes": [...]
- "suggestions": [...]
- Updated TODO plan (edited, concise)
```
### 2.3 Update the first comment (idempotent)
```
Update FIRST comment of [SubtaskId] with the "Updated TODO plan". Keep it as the canonical checklist.
```
### 2.4. Another AI review and combine
- run the same 2.2. prompt but for current agent
- make request:
```
Based on [Subtask] comments with plan and your suggestions make the fully prepared complete plan corresponding all mentioned issues and suggestions, with maximum consistency and clearness. Plan should be fully ready to be implemented by AI agent with no prior knowledge of context. Save structured plan as a comment to CRI-70
```
- keep the last comment as the only one for the next stage
## 3) Implementation
```
Scope: [SubtaskId]
Action: Implement ONLY Phase 1 of the FIRST comment checklist.
Rules:
- Small commits. Keep feature flagged OFF by default.
- Add/extend tests as specified. Ensure `cargo fmt`, `clippy -D warnings` pass.
- Add tracing spans, structured logs, metrics named as in the plan.
```
## 4) Completion
```
Commit format: "<SubtaskId> <Subtask short caption>"
Then comment in [SubtaskId]:
- What changed
- Tests added/updated and results
- Observability added (metrics/spans)
- Any deviations from plan and why
```
## 5) Code review (deep)
```
Scope: Linear MCP [StoryId], [SubtaskId] with comments
Commits:
[list or range]

Review:
- Match to TODO plan & acceptance criteria
- Bugs (incl. data shape misalignments snake_case vs camelCase, nested envelopes {data:{}})
- Concurrency/async issues (Tokio task leaks, blocking calls, backpressure)
- Contracts honored (proto/OpenAPI/topic schemas)
- Errors & retries (thiserror/anyhow, idempotency)
- Observability (tracing spans meaningful, metrics useful)
- Simplicity & file size; refactor suggestions

Output a comment with:
- "severity": ["blocker", "major", "nit"] items
- "examples": code pointers
- "fix_plan": a concise checklist
```
## 6) Implement review findings
```
Apply the "fix_plan" for [SubtaskId].
```
If fixes are large, regenerate the TODO for fixes (2.1), get quick review (2.2), then implement.
