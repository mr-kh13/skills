---
name: pragmatic-tdd
description: Test-driven development, applied pragmatically. Use when implementing a feature or fixing a bug test-first, or when the user asks for TDD or red-green-refactor. Writes tests that give real confidence, skips low-value ones with a reason, and always verifies the change.
---

# Pragmatic TDD

The goal is confidence, not coverage. A test earns its place only if it would
catch a meaningful failure in the behaviour your code is responsible for; green
means nothing on its own. So not every change needs a new test, but every change
must be checked.

## Self-checks

Run every test you write or keep against these. Each one catches a mistake that
looks fine until it costs you. If a test fails a check, fix it or drop it.

- **Wrong layer.** Would this test still pass if the behaviour you care about —
  access control, validation, business rules — were deleted? If yes, it is
  testing below the layer that matters. Test through the interface real callers
  use, not a lower layer that skips the logic.
- **Loose assertion.** Could this still pass if the result were subtly wrong?
  Assert the specific expected value, not a weaker proxy. `toBeTruthy()`,
  `not.toThrow()`, or "length > 0" in place of the real value is not a real
  assertion.
- **Edge-case value.** How likely is this input through a real caller, and how
  bad is the failure if it happens (wrong data, security, crash, money)? Test it
  when likelihood and severity together justify the cost — including rare inputs
  whose failure is catastrophic. Skip speculative cases with little practical
  value, and disclose the skip (see below).
- **Isolation.** Does this pass run alone, and again run twice in a row? Each
  test creates the state it needs and resets mocks and shared variables
  afterwards. No dependence on order or on another test's leftovers.
- **Determinism.** Does this give the same result every run? Control time,
  dates, randomness, and network. Wait on conditions, not fixed delays. A flaky
  test is worse than no test — it teaches the team to ignore red. Treat flakiness
  as a defect: find the cause, don't silence it with retries.
- **Not your code.** Would this only fail if a framework or third-party library
  broke? Then don't write it. Those are battle-tested; stub them at the boundary
  and test your own logic.

## Before coding

1. Read the project rules, related code, existing tests, conventions, and how
   tests are run.
2. Define the behaviour that should change.
3. Check whether an existing test would fail if that behaviour broke.
4. Resolve unclear requirements before writing a test.

Do not use tests to invent requirements. If details such as inputs, output,
errors, locale, or time zone are unclear, use an existing project convention or
clarify them first.

## Decide whether to test

Write or update a test when it helps define behaviour, reproduce a bug, protect
important logic, or prevent a likely regression.

Usually skip a new test when:

- Existing tests already protect the behaviour
- The change only affects docs, comments, copy, or formatting
- The change is mechanical and has no behaviour change
- Types, linting, compilation, or another check fully verifies it
- The test would be fragile or tied to internal details

When existing tests cover the same behaviour, update them instead of adding
duplicate coverage.

Use your best judgment on what to test, but **never skip silently**. When you
decide not to test something a reasonable reviewer might expect covered — a
plausible edge case, a path you judged low-risk — say so and give the reason.
Don't list the infinite tail of irrelevant inputs; disclose the calls you
actually paused on.

Do not skip a valuable test only because it is difficult to set up.

## Choose the test

Choose the level by coverage, not by habit. Use the narrowest test that still
exercises the real logic — the level where the behaviour lives, with real
internal dependencies. Then:

- **No redundant coverage across levels.** If a higher-level test already
  exercises the behaviour meaningfully, do not add a lower-level test that only
  re-covers the same ground.
- Add a focused unit test only for logic a higher-level test can't meaningfully
  reach — complex branching, tricky calculations, many edge cases.

Mock only true external boundaries: services or libraries that can't run in a
test environment (for example, a third-party payment API). Keep everything
internal real. Over-mocking is the same disease as testing the wrong layer — the
test stops exercising real behaviour.

Keep tests focused and easy to read, and define expected results without copying
production code. Keep the structure flat; group only when it improves
readability.

The agent writes unit and integration tests freely by judgment. Pause and ask
first only when a test requires new infrastructure or dependencies the project
doesn't have yet — installing packages, standing up an e2e harness, or adding a
test framework that isn't set up.

## Red → Green → Refactor

### Red

Write one focused test for one small piece of agreed behaviour. Run it and
confirm that it fails, fails for the expected reason, and would catch the
regression if the behaviour broke again.

For a bug fix, first write a test that reproduces the bug.

If the test passes before the change, check whether the behaviour already works,
the test checks the wrong thing, or existing coverage already protects it. If it
can't run because of the environment or existing failures, record why and do not
claim a confirmed red step.

### Green

Make the smallest sensible change that makes the test pass. Do not add
unrequested behaviour or change unrelated code.

### Refactor

Improve names, structure, and duplication with the tests green. Avoid
abstractions that add complexity only for testing.

A behaviour- and API-preserving refactor that breaks a passing test is a
**signal, not a chore**. Diagnose the cause before touching the test: the
refactor actually broke something, the test was coupled to implementation (low
quality), or the change surfaced a pre-existing weakness like shared state or a
fixture. Never just edit the test until it goes green — that hides real breakage
and weakens the suite.

## Verify and report

Run the focused test first, then the relevant suite and project checks based on
risk. When no test is added, still run the strongest useful checks.

Do not leave new test, type, lint, or build failures behind. Report unrelated
existing failures instead of hiding them. Do not call the work TDD unless a
failing test was written before the code change.

Close with a consistent report:

```text
Test decision:
Tests added or updated:
Skipped (considered but cut): [case] — [likelihood/severity] — [reason]
Verification:
Remaining risk:
```

Keep it proportionate: one line per field, and drop any field that adds nothing.
Keep each skipped entry to a single line. State each fact once — don't repeat a
risk across fields — and don't restate the report in surrounding prose. Include
the reason whenever a test was changed, removed, or skipped; omit the "Skipped"
line when there was nothing worth disclosing.

## Example

A feature: only the owner of a record may update it. Ownership is enforced in
the application layer that sits above the data store.

A tempting but worthless test writes and reads the record **straight through the
data store**:

```text
Red? It passes immediately — the row is written and read back.
Self-check (wrong layer): would it still pass if the ownership check were deleted?
Yes. It never ran the ownership logic. It proves the database works, not that
your rule works.
```

Test through the layer where the rule lives instead:

```text
Test decision: write a test — enforces owner-only update, the behaviour at risk.

Red: update as a non-owner through the application interface
     → expect a permission error, got success
     (fails: ownership check not applied)
```

Green: apply the ownership check and nothing more. Refactor: extract the check
only if it reads better, keeping the test green.

```text
Test decision: wrote a test — owner-only update through the application layer.
Tests added or updated: 1 (rejects non-owner update).
Verification: focused test + suite pass.
Remaining risk: none for this path.
```