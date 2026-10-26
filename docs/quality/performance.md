# Performance

## Measure Before Optimizing

Performance work without measurement is guessing, and guessing produces complexity that
costs more than the speed it buys. Profile first, find the actual bottleneck, fix that one
thing, measure again.

Do not optimize before there is evidence of a problem. Do fix the known anti-patterns below
on sight, because they are defects rather than optimizations.

## Workflow

1. **Measure** a baseline from representative data and hardware.
2. **Identify** the actual bottleneck, not the assumed one.
3. **Fix** that bottleneck only.
4. **Verify** by measuring again, then keep or revert.
5. **Guard** with a budget, a benchmark, or a test so the regression cannot return silently.

## Budgets

Defaults until the project sets its own numbers or an ADR changes them:

| Signal                                                   | Target                        |
| -------------------------------------------------------- | ----------------------------- |
| Cold start to a usable shell                             | 3s or less                    |
| Longest UI-thread block during any interaction           | under 50ms; no visible freeze |
| Visible feedback after a click or key press              | within 100ms                  |
| Typical query, 95th percentile, on the reference dataset | under 100ms                   |
| Working set after a one-hour work session                | stable, no unbounded growth   |
| Grid scrolling with virtualization                       | 60 frames per second          |

Measure budgets on representative Windows hardware, including the slowest machine the
product supports, never in Wine or a fast development machine. Define the reference dataset
in the project, sized like the largest real customer. Record the project's real budgets and
guard the measurable ones with benchmarks or tests.

## Where To Start

Let the symptom choose the first measurement:

| Symptom                     | Measure first                                                         |
| --------------------------- | --------------------------------------------------------------------- |
| Slow cold start             | Startup trace: host build, migrations, first query, first window      |
| The window freezes          | UI-thread work during the interaction with `dotnet-trace` or PerfView |
| One screen slow to load     | Its query count and plans, then the amount of data bound              |
| Grid scrolling stutters     | Virtualization, row template complexity, and converters               |
| Memory grows over a session | Heap snapshots before and after repeating the workflow                |
| Every operation slow        | Disk, CPU, antivirus scanning, and database size before any query     |
| Intermittent slowness       | Write-lock contention, `busy_timeout` waits, and external calls       |

## Data Access Anti-Patterns

- **N+1 queries**: loading related rows inside a loop. Fetch the relation in one query.
- **Unbounded reads**: every list query has an explicit limit and a deterministic order.
- **Unindexed critical queries**: every hot query maps to an index confirmed with
  `EXPLAIN QUERY PLAN`. See `../architecture/database.md`.
- **Oversized rows**: select the columns the screen uses, not every column of every table.
- **Long write transactions**: SQLite has one writer; keep transactions short.
- **Serial external calls**: run independent calls in parallel, each with a timeout.

## Interface Anti-Patterns

- **Materializing whole tables** into an `ObservableCollection` instead of paging.
- **No virtualization** on grids and lists, or virtualization disabled by grouping or by a
  `ScrollViewer` around the control.
- **Layout thrash** from deeply nested panels and auto-sized grids inside auto-sized grids.
- **Synchronous I/O on the UI thread**: disk, database, or network work run without `await`.
- **Binding errors**, which cost time on every update as well as hiding defects.
- **Heavy converters** that compute, query, or allocate on every binding update.
- **Large images** not decoded to display size; set `DecodePixelWidth`.
- **Logging in hot loops**, or building log messages that the active level discards.
- **Blocking the window for a local refresh**. See `../architecture/feedback-and-states.md`.

## Tools

- `dotnet-counters` for CPU, allocation rate, GC, and thread-pool trends.
- `dotnet-trace` and PerfView for CPU sampling, startup, and UI-thread stalls.
- The Visual Studio profiler and WPF timeline, on Windows, for layout and render cost.
- `EXPLAIN QUERY PLAN` for every critical query.
- BenchmarkDotNet for isolated hot paths, when a benchmark is worth keeping.

## Verify: Keep Or Revert

A fix is a hypothesis until it is re-measured. Re-measure the same way as the baseline:
same command, same machine, same dataset, same fixed number of runs. Change one thing at a
time; three optimizations shipped together produce one number that cannot be attributed to
any of them. Compare the delta against run-to-run variance, not against the mean alone.

| Result against baseline          | Action                                                |
| -------------------------------- | ----------------------------------------------------- |
| Beats the threshold, tests green | Keep, with the before and after numbers in the commit |
| Inside run-to-run noise          | Revert                                                |
| Worse                            | Revert                                                |
| Better, but a test went red      | Revert; it is a regression wearing a win's clothing   |

**Neutral is a revert, not a keep.** This is the step that gets skipped: the change is
already written, discarding it feels wasteful, so it lands unmeasured and the codebase
accumulates complexity that never paid for itself. Code that is kept is maintained forever.

**Correctness gates the metric.** A win obtained by dropping work the product needed, such
as skipping a validation, caching something that must be fresh, lowering `synchronous`
without a recorded decision, or removing an await that was load-bearing, is a regression.

## Attempt Ledger

Reverted work leaves no trace in history, which is why the same dead idea gets retried.
Record every attempt, kept and reverted alike, in the OpenSpec change's design or a project
performance log:

| Attempt       | Baseline to result | Verdict  | Why                               |
| ------------- | ------------------ | -------- | --------------------------------- |
| `{{ATTEMPT}}` | `{{BEFORE→AFTER}}` | kept     | Reason the number moved.          |
| `{{ATTEMPT}}` | `{{BEFORE→AFTER}}` | reverted | Inside noise; not the bottleneck. |

Read the ledger before proposing an experiment so a failed idea is not run twice.

## Rationalizations To Refuse

| Claim                                             | Reality                                                                |
| ------------------------------------------------- | ---------------------------------------------------------------------- |
| "We will optimize later."                         | Anti-patterns compound. Fix those now; defer only micro-optimizations. |
| "It is fast on my machine."                       | Profile on representative Windows hardware with the reference dataset. |
| "It runs fine in Wine."                           | Wine is not a performance reference. Measure on Windows.               |
| "This optimization is obvious."                   | If it was not measured, it is not known.                               |
| "It did not help much, but it does not hurt."     | Neutral changes are reverts. Maintenance is paid forever.              |
| "We already wrote it, so we may as well keep it." | Sunk cost. The measurement does not care how long it took to write.    |

## Reject The Change When

- an optimization has no profiling data behind it;
- a list query has no limit, or a grid binds a whole table;
- data access introduces an N+1 pattern;
- disk, database, or network work runs synchronously on the UI thread;
- several optimizations share one measurement, so none can be attributed;
- a "win" required a test to be changed, skipped, or deleted;
- a neutral result was kept instead of reverted.

## Verification

- [ ] Before and after measurements exist as specific numbers.
- [ ] The result was measured the same way as the baseline, on Windows.
- [ ] The improvement exceeds run-to-run variance.
- [ ] Results that did not beat the baseline were reverted.
- [ ] Attempts were logged, reverted ones included.
- [ ] The specific bottleneck is named and addressed.
- [ ] No new N+1 pattern, unbounded query, or UI-thread I/O was introduced.
- [ ] Budgets still pass.
- [ ] The existing test suite passes.
