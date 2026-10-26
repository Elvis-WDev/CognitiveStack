# Test Instructions

Apply to files under `tests/`.

- Cover risky behavior and cross-boundary contracts, not a coverage number.
- Name tests `Method_Scenario_Result` and structure them as Arrange, Act, Assert.
- Keep tests deterministic: inject `TimeProvider`, use fakes behind ports, and load fixtures.
- Test repositories and migrations against a real SQLite file, never a mock of SQL.
- Test ViewModels without WPF: commands, validation, loading, error, and cancellation.
- Test error results, validation, authorization, and permissions.
- For private data, cover foreign IDs and role checks, with at least two companies in the
  fixtures when the database holds several.
- For critical writes, cover repeated, concurrent, and cross-company uses of the same
  idempotency key, using two real connections for concurrency.
- Keep tests against real external test environments opt-in and segregated, never with real
  customer data.
