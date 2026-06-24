# Dynamic Harness

A collection of [Claude Code](https://docs.anthropic.com/en/docs/claude-code) skills for disciplined, structured AI-assisted development.

These skills are designed to prevent common failure modes when working with AI coding agents: drift, hallucination, scope creep, and lost context across sessions.

## Skills

### `/harness` — Structured Execution Framework

The core skill. Wraps any implementation task in a disciplined protocol with:

- **Complexity classification** — adjusts ceremony to match task size (Light/Medium/Heavy)
- **Hallucination checkpoints** — verifies files, symbols, and APIs exist before writing code
- **Verification gates** — every step must pass a concrete check before proceeding
- **Course-correction & hard abort** — prevents confidently-wrong streaks and sunk-cost traps
- **Multi-session persistence** — `.harness/` files track progress, decisions, and context across days

Best for: complex implementations, multi-day refactors, tasks where you've been burned by AI drift before.

[Full documentation](skills/harness/README.md)

### `/grill-me` — Design Interrogation

An interview skill that stress-tests your plan or design by walking down every branch of the decision tree, resolving dependencies one-by-one.

Best for: pre-implementation design validation, finding gaps in your thinking, reaching shared understanding before committing to an approach.

## Installation

### Option 1: Copy into your Claude Code skills directory

```bash
# Copy individual skills
cp -r skills/harness ~/.claude/skills/harness
cp -r skills/grill-me ~/.claude/skills/grill-me
```

### Option 2: Clone and symlink

```bash
git clone https://github.com/SumelRattan/dynamic-harness.git
ln -s $(pwd)/dynamic-harness/skills/harness ~/.claude/skills/harness
ln -s $(pwd)/dynamic-harness/skills/grill-me ~/.claude/skills/grill-me
```

## Usage

Once installed, invoke from any Claude Code session:

```
/harness Add pagination to the catalog API
```

```
/grill-me Here's my plan for the auth refactor...
```

The harness skill auto-detects in-progress tasks — if a `.harness/` directory exists with an incomplete task, it offers to resume where you left off.

## How it works

The harness operates in phases:

```
Classify → Understand → Decompose → Construct Plan → Execute (with gates) → Stabilize
```

For multi-day tasks, it adds persistence:

```
... → Construct Plan → Persist to .harness/ file → Execute milestone → Checkpoint → [next session] → Resume → Execute ...
```

Key safety mechanisms:

| Mechanism | Prevents |
|-----------|----------|
| Hallucination checkpoints | Inventing files/symbols/APIs that don't exist |
| Verification gates | Broken code progressing to the next step |
| Course-correction triggers | Confidently wrong for many steps |
| Hard abort protocol | Sunk-cost fallacy on a flawed approach |
| Drift budget (8 steps/session) | Unbounded sessions losing coherence |
| Stale-decision review | Building on expired assumptions |

## Contributing

PRs welcome. If you've built a Claude Code skill that prevents a common AI-agent failure mode, it fits here.

## License

MIT
