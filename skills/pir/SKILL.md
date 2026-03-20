---
name: pir
description: |
  PIR (Plan-Implement-Review) — structured workflow for non-trivial coding tasks.
  Six phases: Plan, Plan Review (Codex), Revise, Implement, Code Review (Codex), Final Fix.
  Enforces review gates between phases — no implementation without a reviewed plan,
  no completion without a code review pass. Plan review and code review are delegated
  to Codex by default for an independent perspective.
  Use when asked to "implement this", "build this feature", or for any multi-file
  change that benefits from upfront planning.
---

# Implement — Structured Plan-Review-Implement Workflow

Enforces a disciplined workflow: **Plan → Codex Review → Revise → Implement → Codex Review → Fix**.
Every phase has a clear deliverable and exit criteria. No skipping phases.

## Ground Rules

- **You implement directly.** Do NOT delegate implementation to sub-agents, Codex, or Claude Code sessions unless the user explicitly asks.
- **Reviews are delegated to Codex by default.** Both plan review (Phase 2) and code review (Phase 5) are sent to Codex for an independent perspective. If the user says "no codex", "skip codex", "review yourself", or similar — do the reviews yourself instead.
- **No phase skipping.** Each phase must complete before the next begins.
- **Ask before proceeding** at each gate (marked with 🚦) — the user decides whether to continue, revise, or abort.
- **Keep plans concrete.** File paths, function names, data structures — not hand-wavy descriptions.
- **Follow existing patterns.** Read the codebase first. Match conventions already in use.

## Codex Integration

Reviews are delegated to Codex via:

```bash
~/.claude/skills/codex/scripts/ask_codex.sh "<review prompt>" \
  --workspace <project-root> \
  --read-only \
  --reasoning high
```

- Always use `--read-only` — Codex reviews but does NOT modify files.
- Always use `--reasoning high` — reviews need deep analysis.
- Save `session_id` from the output — use `--session <id>` if follow-up review is needed after revisions.
- Read the output file and present findings to the user.

---

## Phase 1: PLAN

**Goal:** Produce a concrete, reviewable implementation plan.

1. Read all relevant source files, tests, configs, and documentation.
2. Identify every file that needs to change and what changes are needed.
3. Note any risks, edge cases, or open questions.
4. Output a structured plan:

```
## Implementation Plan

### Summary
[1-2 sentence description of what we're building/changing and why]

### Changes
- `path/to/file.py` — [what changes and why]
- `path/to/other.py` — [what changes and why]
- ...

### New Files (if any)
- `path/to/new.py` — [purpose]

### Risks & Edge Cases
- [risk 1]
- [risk 2]

### Open Questions
- [anything that needs user input before proceeding]

### Test Strategy
- [how we'll verify correctness]
```

Do NOT present the plan to the user yet — send it to Codex for review first (Phase 2).

---

## Phase 2: PLAN REVIEW (Codex)

**Goal:** Get an independent review of the plan before presenting it to the user.

Send the full plan to Codex with this prompt structure:

```
Review this implementation plan for a codebase at <workspace>.
Focus on:
1. Are there files or dependencies the plan missed?
2. Will existing tests break? Are new tests needed?
3. Are there database/schema/migration implications not addressed?
4. Is there a simpler approach that achieves the same result?
5. Any risks or edge cases not covered?
6. Does the plan follow existing codebase patterns?

Here is the plan:
<paste the full plan>
```

After receiving Codex's review:
1. If Codex found issues — fix the plan and note what was caught.
2. If the plan is clean — note that Codex approved it.
3. Present BOTH the plan AND Codex's review summary to the user.

🚦 **Gate 1:** Present to the user:
> Here's the implementation plan. Codex reviewed it and [found N issues which I've addressed / approved it]. Want me to proceed, or would you like changes?

Wait for user approval before continuing.

---

## Phase 3: REVISE

**Goal:** Incorporate user feedback into the plan.

If the user requests changes at Gate 1:
1. Update the plan based on feedback.
2. For significant changes, re-send to Codex for another review pass (use `--session <id>` to continue the conversation).
3. For minor changes, note the update and skip re-review.
4. Present the updated plan.

🚦 **Gate 2:** Ask the user to confirm the revised plan. Repeat until approved.

---

## Phase 4: IMPLEMENT

**Goal:** Execute the approved plan.

1. Work through the plan file by file, in dependency order.
2. After each file edit, run any relevant tests immediately if a test command is available.
3. If a test fails, fix it before moving to the next file. Do NOT accumulate broken state.
4. If you discover the plan needs adjustment mid-implementation:
   - For minor adjustments: note them and continue.
   - For significant deviations: stop and inform the user before proceeding.

**Implementation rules:**
- Make the minimal changes needed. Do not refactor surrounding code.
- Do not add comments, docstrings, or type hints to code you didn't change.
- Match the existing code style exactly (naming, formatting, patterns).
- Run the full test suite (if available) after all changes are complete.

---

## Phase 5: CODE REVIEW (Codex)

**Goal:** Independent code review before presenting to the user.

After implementation is complete, send the diff to Codex for review:

```
Review the following code changes in <workspace>.
The changes implement: <one-line summary from the plan>.

Focus on:
1. Correctness — do the changes implement the plan accurately?
2. Missed side effects — imports, broken references, unintended behavioral changes
3. Test coverage — are the changes adequately tested?
4. Security — injection, credential exposure, trust boundary issues
5. Style — do changes match existing codebase conventions?
6. Leftover artifacts — debug code, TODO comments, dead code

Run `git diff` in the workspace to see all changes.
```

After receiving Codex's review:
1. Fix any valid issues Codex found.
2. Note any findings you disagree with (present to user for decision).
3. Compile the final summary.

🚦 **Gate 3:** Present to the user:

```
## Implementation Complete

### Changes Made
- `path/to/file.py` — [summary of change]
- ...

### Codex Review
- [finding 1 — fixed]
- [finding 2 — fixed]
- [finding 3 — disagree, reason: ...]
  (or "Codex approved — no issues found")

### Test Results
- [pass/fail summary]

### Notes
- [any deviations from the original plan and why]
```

---

## Phase 6: FINAL FIX

**Goal:** Address any remaining issues from Codex review or user feedback.

1. Fix any issues raised by Codex or the user.
2. Re-run tests after fixes.
3. If fixes are significant, optionally re-send to Codex for a final pass.
4. Confirm all clear.

---

## When the User Provides Arguments

If invoked as `/pir <description>`, treat the description as the task and start Phase 1 immediately by reading the relevant codebase.

If invoked as just `/pir`, ask:
> What would you like me to implement?

## Opting Out of Codex Reviews

If the user says "no codex", "skip codex", "review yourself", or similar at any point:
- Switch to self-review for the remainder of the workflow.
- Use the same review checklists but execute them yourself instead of delegating.

## Workflow Diagram

```
Plan → Codex Plan Review → 🚦 User Approves? → Revise (loop) → Implement → Codex Code Review → 🚦 User Approves? → Final Fix → Done
```
