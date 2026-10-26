---
name: optimize-performance
description: Use when a performance requirement exists, when startup, screen load, query times, or interface responsiveness regress, or when profiling is needed. Enforces measuring before optimizing and reverting changes that do not beat the baseline.
---

# Optimize Performance

## Context To Load

Read:

- `docs/quality/performance.md`
- Relevant architecture document for the layer being measured: `docs/architecture/dotnet-architecture.md` or `docs/architecture/wpf-mvvm.md`.
- `docs/architecture/database.md` for query and index work.
- The project's performance ledger or the OpenSpec change's design, to avoid repeating a failed attempt.

## Workflow

1. State the symptom and the user-visible cost. Refuse to optimize without one.
2. Measure a baseline on representative Windows hardware with the reference dataset, never in Wine, and record the exact tool, command, and conditions.
3. Identify the bottleneck from the measurement, not from assumption.
4. Change one thing.
5. Re-measure the same way as the baseline.
6. Keep only if the result beats run-to-run variance and the suite stays green. Otherwise revert.
7. Record the attempt with its numbers and verdict, including reverted attempts.
8. Add a budget, monitor, or test so the regression cannot return unnoticed.

## Guardrails

- Do not optimize without evidence of a problem.
- Do not combine several optimizations into one measurement.
- Do not keep a change that landed inside the noise; neutral is a revert.
- Do not accept a win that required changing, skipping, or deleting a test.
- Do not gain speed by dropping work the product needs.
- Do not add memoization, caching, or indirection that no profile justified.
- Do not leave a list query unbounded, a grid unvirtualized, or a relation loaded inside a loop.
- Do not move work off the UI thread by blocking on it with `.Result` or `.Wait()`.
