# /harness — Dynamic Agent Harness

A structured execution framework for Claude Code that prevents drift, catches hallucination early, and enables multi-day task continuity.

## What it does

When invoked, the harness skill wraps your task in a disciplined protocol:

1. **Classifies complexity** — adjusts ceremony to match task size (Light/Medium/Heavy)
2. **Forces problem clarity** — won't start coding until the goal is confirmed
3. **Decomposes with verification** — every step gets a gate that must pass before proceeding
4. **Catches hallucination** — explicit checkpoints verify files, symbols, and APIs exist before writing code
5. **Persists across sessions** — for multi-day work, maintains a `.harness/` file with progress, decisions, and context
6. **Self-corrects or aborts** — course-correction triggers prevent confidently-wrong streaks; hard abort prevents sunk-cost traps

## When to use it

- Complex implementation tasks (new features, refactors, migrations)
- Work that will span multiple sessions or days
- Tasks where you've been burned by LLM drift or hallucination before
- Any time you want a structured approach with checkpoints

## When NOT to use it

- Quick questions or lookups
- One-line fixes where the answer is obvious
- Exploratory work where you don't know the goal yet (use `/grill-me` first)

## How to invoke

```
/harness
```

Or include the problem inline:

```
/harness Refactor the auth middleware to use OAuth-only flow across all routes
```

To resume a previous multi-day task, just invoke `/harness` — it auto-detects in-progress `.harness/` files and offers to resume.

## Complexity levels

| Level | Steps | Sessions | Ceremony |
|-------|-------|----------|----------|
| Light | 1-3 | 1 | Confirm goal, execute with gates, done |
| Medium | 4-9 | 1 | Full phases, no persistence file |
| Heavy | 10+ | Multi-day | Full phases + persistence file + milestones |

## The `.harness/` file (Heavy tasks only)

For multi-day work, creates `.harness/<task-slug>.md` in your project root containing:

- **Restatement** — the confirmed goal and constraints
- **Milestones & steps** — with status markers (`[ ]`, `[x]`, `[~]`, `[-]`)
- **Decision log** — why non-obvious choices were made (prevents re-litigation across sessions)
- **Session log** — what was done, where to resume, context for next time

## Safety mechanisms

| Mechanism | What it prevents |
|-----------|-----------------|
| Hallucination checkpoints | Inventing files, symbols, or APIs that don't exist |
| Verification gates | Broken code progressing to the next step |
| Course-correction triggers | Confidently wrong for many steps |
| Hard abort protocol | Sunk-cost fallacy — patching a fundamentally flawed approach |
| Drift budget (8 steps/session) | Unbounded sessions that lose coherence |
| Stale-decision review | Building on assumptions that expired days ago |

## Example session

```
User: /harness Add pagination to the catalog API

Claude: Classifying complexity... Medium (4-6 steps, 2-3 files, patterns exist in codebase).

Let me ask a couple of questions:
1. Is this cursor-based or offset-based pagination? I see the existing /search endpoint uses cursor-based — should we match that?
2. What's the default page size? 20? 50?

User: Cursor-based, default 25.

Claude:
GOAL: Add cursor-based pagination to the catalog API endpoint
CONSTRAINTS: Match existing /search endpoint pattern, default page size 25
SUCCESS CRITERIA: GET /catalog?cursor=X&limit=25 returns paginated results with nextCursor
SCOPE: In — API handler, response type, tests. Out — frontend changes.
COMPLEXITY: Medium

Confirm?

User: yes

Claude: [proceeds through decomposition → construction → execution with gates]
```

## Tips

- **Trust the abort.** If the harness triggers a hard abort, let it restart. Forcing a broken plan forward wastes more time than rebuilding.
- **Confirm milestones.** The harness pauses between milestones — use that moment to sanity-check direction.
- **Decision log is for future-you.** Write "why" not "what" — the code shows what; the log explains why you chose it over alternatives.
- **Don't fight the complexity classification.** If a "Light" task turns out harder, say so — the harness will upgrade.
