---
name: tdd
description: Test-driven development. Use when the user wants to build features or fix bugs test-first, or mentions "red-green-refactor".
---

# Test-Driven Development

TDD is the red → green loop. This skill is the reference that makes that loop produce tests worth keeping: the anti-patterns and the rules of the loop. Every section applies on every cycle: consult them before and during the loop, not after.

Call the Skill tool with "writing-test" before the first test of the session. It owns each test inside the loop: what a good test is, the seams, and mocking. On a conflict about the test itself, `writing-test` wins.

When exploring the codebase, read `GLOSSARY.md` (if it exists) so test names and interface vocabulary match the project's domain language, and respect ADRs in the area you're touching.

## Anti-patterns

Anti-patterns inside one test (implementation-coupled, tautological) are in `writing-test`. This one is about the loop:

- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify _imagined_ behavior: you test the _shape_ of things rather than user-facing behavior, the tests go insensitive to real changes, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test by steps 3–4 of `writing-test`, then only enough code to pass it. Don't anticipate future tests or add speculative features.
- **One slice at a time.** One behaviour, one test at its entry seam, one minimal implementation per cycle.
- **Report at the end.** When the loop is done, report as step 6 of `writing-test` says.
- **Refactoring is not part of the loop.** It belongs to the review stage (see the `code-review` skill), not the red → green implementation cycle.
