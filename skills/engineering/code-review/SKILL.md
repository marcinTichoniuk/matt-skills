---
name: code-review
description: 'Review the diff since a fixed point (commit, branch, tag, or merge-base) on four axes: Standards (code smells), Spec (matches the issue or spec), Tests (tests conventions), and Performance (Vercel React and Next.js rules). Use when the user wants to review a branch, a PR, or work-in-progress changes, or asks to "review since X".'
---

Review the diff between `HEAD` and a fixed point on four axes: **Standards**, **Spec**, **Tests**, and **Performance**. Each axis runs in its own parallel sub-agent, so the axes do not share context.

## Process

### 1. Pin the fixed point

The fixed point is whatever the user named (a commit SHA, branch, tag, `main`, `HEAD~5`). If they named none, ask.

Check that `git rev-parse <fixed-point>` resolves and that `git diff <fixed-point>...HEAD` is non-empty (three-dot, so the comparison is against the merge-base). A bad ref or an empty diff fails here, not inside the sub-agents. Note the commits with `git log <fixed-point>..HEAD --oneline`.

### 2. Find the spec

Look in this order:

1. Issue references in the commit messages (`#123`, `Closes #45`, GitLab `!67`): fetch the relevant ticket from the issue tracker.
2. A path the user passed as an argument.
3. A spec in the issue tracker or the repo (for example under `docs/` or `specs/`) that matches the branch name or feature.
4. Ask the user where the spec is.

Done when you have the spec, or the user says there is none.

### 3. Spawn the sub-agents in parallel

Give each sub-agent the full diff command, the commit list, its brief, and the **shared rules** below, pasted in full (the sub-agent has no other access to them):

> - **The repo overrides.** A convention that many files in the repo share wins over the rules of this review. A defect that other files repeat is still a finding, because a repeated defect is not a convention.
> - Standards and Performance leave out test and story files. The Tests axis owns tests.
> - For each finding, give one verdict and a one-line reason:
>   - **fix**: the finding is correct, and the change is clear from the finding alone.
>   - **skip**: the finding is correct, but the change costs more than it gives, or reaches code outside the diff.
>   - **decide**: the user must choose. Either behaviour that the spec did not ask for (its removal is a product decision), or a Performance fix with impact MEDIUM or higher that reaches outside the diff (a fix now, or a follow-up ticket).
> - If nothing matches, report "no findings". Stay under 400 words.

**Standards**: also give the absolute path of [`smells.md`](smells.md). Brief: "Read `smells.md`. Report, per file or hunk, each smell from it that you spot in the diff: name it and quote the hunk."

**Spec**: also give the path or the fetched contents of the spec. Brief: "Report: (a) requirements the spec asked for that are missing or partial; (b) behaviour in the diff that the spec did not ask for (scope creep); (c) requirements that look implemented, but where the implementation looks wrong. Quote the spec line for each finding. Leave the tests to the Tests axis."

**Tests**: also give the list of test files that the diff adds or changes. The full diff stays in, because the production code shows the missing coverage. The spec goes to the Spec axis only. Brief: "Invoke the `writing-test` skill, then review the tests that this diff adds or changes against it. Done when each behaviour that the diff adds or changes has a test at its seam, or a finding. Do not edit, write or run tests: only report the findings."

**Performance**: brief: "Invoke the `vercel-react-best-practices` skill. Use its Quick Reference as an index: pick the rules whose patterns appear in the diff, and read only those files in its `rules/` directory, never its compiled `AGENTS.md`. Report each place where code that the diff adds or changes breaks a rule: name the rule, quote its `impact:` line and the hunk, and name the cost (an extra request, a larger bundle, more renders). Order the findings by `impact:`, CRITICAL first."

Skip a sub-agent, and note the skip in the final report:

- **Spec**, if there is no spec.
- **Tests**, if the diff changes no test file and no source file.
- **Performance**, if the project does not use React or Next.js, or the diff changes no source file outside tests and stories.

### 4. Aggregate

Present the reports under `## Standards`, `## Spec`, `## Tests`, and `## Performance` headings, verbatim or lightly cleaned. Give each finding an ID for its axis (`S1` Standards, `D1` Spec, `T1` Tests, `P1` Performance), and keep its verdict and reason. Keep each axis apart, in the order its sub-agent gave: a change can pass one axis and fail another, and a merged ranking lets one axis hide another.

End with a one-line summary: the findings per axis, the count of each verdict, and the worst issue within each axis.

### 5. Ask once

If no finding has the verdict **fix** or **decide**, end here.

Ask the user, and wait for the answer before you change any code:

> Reply `ok` to apply every **fix**. To change a verdict, list the IDs: `ok, skip S3, fix T2`. Answer each **decide**, or it counts as **skip**.

### 6. Apply

Apply each approved finding. For a Tests finding, call the Skill tool with "writing-test" unless it is loaded, and follow it. Then run the project's quality checks.

Done when each approved finding is fixed or reported with the reason it is not, and each check is green or reported with its output.

Report each finding as fixed or skipped. If the fixes were large, suggest a new `code-review` run.
