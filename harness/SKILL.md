---
name: harness
description: Dynamically generates a structured agent harness for a problem statement — with hallucination guardrails, incremental checkpoints, and course-correction gates. Use when tackling complex implementation tasks, when you want a disciplined execution plan, or when you say "harness this", "create a harness", or "structured approach".
---

# Dynamic Agent Harness

Generate and execute a disciplined harness for the given problem. The harness prevents drift, catches hallucination early, and ensures incremental progress toward the goal.

---

## Complexity Classification (do this FIRST)

Before entering Phase 0, classify the task:

| Level | Signal | Ceremony |
|-------|--------|----------|
| **Light** | 1-3 steps, single file area, well-understood change | Skip Phases 1 and 2.5. Abbreviated Phase 0 (confirm goal in 1 exchange). Execute directly with gates. |
| **Medium** | 4-9 steps, 2-4 files, some unknowns | Full Phase 0-4. No persistence file needed unless it stalls. |
| **Heavy** | 10+ steps, multiple systems/files, multi-day, unknowns dominate | Full Phase 0-4 + Phase 2.5 (persistence). Milestones mandatory. |

State the classification and your reasoning. If the user disagrees, adjust.

**Batching rule:** Steps that are trivial AND tightly coupled (add import + add type + use type) may be batched into one logical step at Light/Medium complexity. At Heavy complexity, never batch — the overhead is worth the safety.

---

## Phase 0: Problem Understanding (MANDATORY — do not skip)

Before doing anything else, reach full clarity on the problem. Ask targeted clarifying questions one at a time until you can answer ALL of the following:

1. **What is the desired end state?** — What does "done" look like concretely?
2. **What are the constraints?** — Technology, timeline, scope boundaries, things NOT to change.
3. **What's the success criteria?** — How will the user know it works? What should they be able to do/see?
4. **What's the context?** — Why is this needed? What broke, or what's missing?
5. **What's off-limits?** — Are there areas of the codebase, patterns, or approaches to avoid?

### How to ask

- Ask 2-3 questions per round (not all at once — that overwhelms).
- For each question, offer your best guess or recommendation so the user can just confirm or correct.
- If a question can be answered by reading the codebase, read the codebase instead of asking.
- Stop asking when you can restate the full problem back to the user and they confirm it.

### Gate: Restate and Confirm

Once you have enough clarity, restate the problem in structured form:

```
GOAL: [one sentence]
CONSTRAINTS: [bullets]
SUCCESS CRITERIA: [testable statements]
SCOPE: [what's in / what's out]
APPROACH HINT: [your initial read on how to solve it, 1-2 sentences]
COMPLEXITY: [Light / Medium / Heavy — with reasoning]
```

**Do NOT proceed past Phase 0 until the user confirms this restatement.**

---

## Phase 1: Problem Decomposition

Now explore the codebase and build a grounded mental model:

1. **Identify knowns** — what exists in the codebase right now that's relevant. Read files, grep symbols, check tests. Do NOT assume structure — verify it.
2. **Identify unknowns** — what you'd need to look up, what APIs/patterns you're unsure about.
3. **Identify hallucination risks** — where are you most likely to invent something that doesn't exist? Flag these explicitly:
   - API signatures you haven't verified
   - File paths you haven't confirmed
   - Patterns you're inferring but haven't seen in this codebase
   - Library features you recall but haven't checked the version for

Output the decomposition to the user. If anything is unclear or risky, ask before proceeding.

---

## Phase 2: Harness Construction

Build an execution plan with these mandatory components:

### A. Steps (ordered, small, testable)

Break work into the smallest steps where each one:
- Changes at most 1-2 files
- Can be verified independently (build, test, or manual check)
- Has a clear "done" signal

For Heavy tasks, group steps into **milestones** (3-5 steps each). Each milestone should produce a committable, non-breaking state. Plan one milestone per session as the default scope.

### B. Verification Gates (one per step)

Each step gets a gate — a concrete check that MUST pass before moving on:
- **Build gate**: does it compile/parse without errors?
- **Test gate**: do existing tests still pass?
- **Behavior gate**: does the new behavior work as intended?
- **Consistency gate**: is the change consistent with surrounding code patterns?

### C. Hallucination Checkpoints

At each step, before writing code, explicitly verify:
- [ ] The file I'm about to edit exists at the path I think
- [ ] The function/class/symbol I'm referencing actually exists (grep it)
- [ ] The API/method signature matches what I'm about to call
- [ ] The pattern I'm using matches how this codebase does it elsewhere

If ANY checkpoint fails — STOP, re-research, and update the plan.

### D. Course-Correction Triggers

The harness must pause and reassess if:
- A verification gate fails twice on the same step
- A checkpoint reveals the codebase structure differs from assumption
- The user flags something as wrong
- The approach requires touching 3+ unexpected files (scope creep signal)

On trigger: summarize what went wrong, propose an adjusted approach, and get confirmation before continuing.

### E. Hard Abort Protocol

If course-correction has fired **3 times** on the same task, or if you realize the fundamental approach is wrong (not a detail — the architecture):

1. STOP all execution immediately
2. State clearly: "ABORT — the current approach is fundamentally flawed. Here's why: [reason]"
3. Discard the current step list
4. Return to Phase 1 with the new understanding
5. Rebuild the harness from scratch with user confirmation

Do NOT try to patch a broken approach incrementally. The sunk-cost of completed steps is not a reason to continue down a wrong path.

---

## Phase 2.5: Persistence (for Heavy tasks)

When the task is classified Heavy (>10 steps, multi-day):

### Harness File

Create `.harness/<task-slug>.md` in the project root with:

```markdown
# <Task Title>

## Restatement
GOAL: ...
CONSTRAINTS: ...
SUCCESS CRITERIA: ...
COMPLEXITY: Heavy

## Milestones

### Milestone 1: <name>
- [ ] Step 1: description
- [x] Step 2: description (done 2026-06-23)
- [~] Step 3: description (in-progress)
- [-] Step 4: description (skipped — reason)

### Milestone 2: <name>
- [ ] Step 5: description
- [ ] Step 6: description

## Decision Log
| Date | Decision | Alternatives | Why | Still valid? |
|------|----------|-------------|-----|--------------|
| 2026-06-23 | Used React context over Redux | Redux, Zustand | Only 2 components need this state, context is simpler | ✓ |
| 2026-06-24 | Skipped Lambda target type | Keep it hidden, remove entirely | Product says no Lambda support planned for 6mo | ✓ |

## Blockers / Open Questions
- Waiting on API team for schema confirmation
- Unclear: should validation run client-side or server-side?

## Session Log
### Session 1 (2026-06-23)
- Completed: Milestone 1 (steps 1-4)
- Resume from: Milestone 2, Step 5
- Context for next session: The API returns 404 for agent types — may need a mock until backend deploys
- Decisions made: 2 (see log)
- Aborts/corrections: 0
```

### Session-End Protocol

At the end of each session:
1. Update the harness file with current step progress
2. Mark the exact step to resume from
3. Note any context the next session will need that isn't obvious from the code or harness file
4. Commit the harness file alongside any code changes

### Session-Start Protocol

At the start of each session:
1. Read the harness file
2. Verify the last "done" step still holds (quick gate re-run — build/test)
3. State: "Resuming from step N. Last session completed X."
4. **Stale-decision review**: for each decision older than 2 days that the current step depends on — re-validate it (check the code, re-read the constraint). If invalid, flag before proceeding.
5. Check if any blockers have resolved

### Drift Budget

- Maximum **8 steps** per session before forcing a progress checkpoint (update harness file, commit, assess drift)
- At milestone boundaries: commit, run full test suite, update harness file
- If a session ends mid-milestone, note what remains and why
- If drift is detected at checkpoint: return to the restatement, compare current trajectory vs. stated goal, and correct or abort

---

## Phase 3: Execution

Execute the plan step by step:

```
For each step:
  1. State what you're about to do (1 sentence)
  2. Run hallucination checkpoints (verify before writing)
  3. Make the change (minimal, focused)
  4. Run the verification gate
  5. Report: PASS → next step | FAIL → diagnose and retry once | FAIL×2 → course-correct | FAIL×3 across task → hard abort
```

### Execution Rules

- **Never skip a gate.** If you can't run the gate (e.g., no test exists), say so and ask the user how to verify.
- **Batch only when trivial.** Tightly coupled trivial changes (import + type + usage) can be one step at Light/Medium. At Heavy, one step = one change.
- **Prefer reading over assuming.** If unsure about anything, read the file first. The cost of reading is zero; the cost of a wrong assumption cascades.
- **Surface uncertainty early.** If you're 70% sure about something, say "I'm assuming X based on Y — correct?" rather than proceeding silently.
- **Track drift.** After every 3 steps, do a quick self-check: "Am I still solving the original problem, or have I drifted into tangential work?"
- **Respect the drift budget.** After 8 steps in a single session, STOP and checkpoint — update the harness file, commit progress, and assess whether to continue or pause.
- **Log non-obvious decisions.** Any time you choose between two viable approaches, add it to the Decision Log with reasoning. Future sessions depend on this.
- **Announce milestone completions.** When a milestone is done, explicitly state it and confirm with the user before moving to the next one.

---

## Phase 4: Stabilization

After all steps complete:

1. **Run the full verification suite** — build, tests, lint (whatever the project uses)
2. **Diff review** — look at the total diff and check for:
   - Unintended changes (files you didn't mean to touch)
   - Leftover debug code
   - Inconsistencies with the original plan
3. **Goal check** — re-read the original problem statement. Does the implementation actually solve it? Not a related problem, not a superset — the actual stated goal.
4. **Report** — summarize what was done, what was verified, and any remaining risks or TODOs.
5. **Clean up** — if a `.harness/` file exists for this task, mark it complete or delete it per user preference.

---

## Invocation

### New task

When this skill is invoked, begin by classifying complexity, then enter Phase 0. Ask the user:

> What's the problem you want me to harness?

Then ask clarifying questions until you reach full understanding, confirm the restatement, and proceed through the appropriate phases for the complexity level.

If the user provides the problem in the same message as invoking the skill, go directly into clarifying questions (Phase 0) — do NOT skip to decomposition.

### Resuming a task

If a `.harness/` directory exists in the project, check for an in-progress harness file. If one exists:

1. Read it and present a summary: "Found in-progress harness: **<title>**. Milestone N, step M. Last session completed X."
2. Ask: "Resume this, or start a new harness?"
3. If resuming, follow the Session-Start Protocol (Phase 2.5) before continuing execution.
