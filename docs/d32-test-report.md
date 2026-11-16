# D32 Validation Test: Report

Standard: version 0.1 (core, REST-only)
Repository: `fabrizioiacobucci/test`, Project `test-project`
Date: 2026-11-16
Actors: Owner (person, mostly from a phone), Agent 1 and Agent 2 (cloud sessions), one GitHub account for all

## Outcome

D32 is met. A Feature, a Bug and a Task went from birth to closure, with one rejection in review and one stop in Waiting. Version `v0.1.0` was released and its milestone closed.

| Issue | Type | Path | Closed by |
|-------|------|------|-----------|
| #7 | Task | Birth → Ready at triage → pick-up → PR #9 → merge | PR #9, squash `docs(changelog): add CHANGELOG.md (#7)` |
| #8 | Feature | Birth → Accepted → Design, blocked by #7, signed → Ready → not startable while #7 open → pick-up → PR #10 → merge | PR #10, squash `feat(greet): add greet script (#8)` |
| #11 | Bug | Birth → Accepted → reproduced, signed → Ready → pick-up → Waiting → In Progress (D33) → PR #12 → rejected → fixed → merge | PR #12, squash `fix(greet): keep every word of the name (#11)` |
| #13 | Task | Release Task, created and accepted as two acts → pick-up → PR #14 → merge | PR #14, squash `chore(release): v0.1.0 (#13)` |
| #2, #3 | Bug | Rejected at triage, reason in a comment | Closed as not planned |

## Validated by execution

- Steps 1 to 9, steps 3 and 9 in their simple form on `develop`
- Birth through REST with the type set; forms read from the `.github` repository
- Triage queue built by the agent from the REST issue list; outcomes Accept (straight to Ready for a Task, Accepted for a Bug and a Feature) and Reject
- D21: an approved Feature blocked by an open issue was not picked up; it became startable when the blocker closed
- Claim order at pick-up (write `Executor`, read again, go on)
- Approval: two different names; a Bug reproduced and its outcome in a comment
- D33: leaving Waiting restored In Progress from `Executor`, with no timeline read
- Review: rejection with the reason in the pull request, `Stage` back to In Progress, fix on the same branch, second review
- Closure: squash title without the pull request number (merged from a phone, title edited by hand); issue closed by `Closes #n`; branch deleted automatically (D34)
- Changelog: one line per Feature and Bug; renamed at release with a new empty "Unreleased"
- `Reviewed by` different from `Executor` on every closed issue

## Not exercised

- Triage outcomes Duplicate and Suspend (Needs Info); duplicate closing was checked only in the REST checks
- A Bug not reproduced (to Waiting with a question to the reporter)
- Giving up, a substantial change of Design or criterion, reopening
- Leftovers at release
- Two executors racing for the same issue
- Work that leaves no file
- Silence clock

## Defects of the standard (to open as issues in `.github`)

1. The paths of the issue forms and of the configuration file in the `.github` repository are not fixed. The first agent stopped on a wrong path
2. Who signs "Prepared by" is unclear: D22 says both names are signed, step 4 says the Approver signs. The test used: each signs its own line
3. Whether `Reviewed by` is set on a rejection is unclear; if another person reviews the second round, the first name is overwritten
4. Step 9 cannot be completed from a cloud session: releases are refused (403). The test used a person. Preferred direction: a GitHub Action that tags and publishes when the release Task is merged
5. The `Executor` option "Owner" collides with D11, where "Owner" means the assignee, and with D26, which asks for one option per person
6. `Priority` is mandatory but has no criteria; the test used ad hoc ones

## Usage risks (for the scripts, O15)

- A person doing a two-part act (body edit, then field) twice forgot the field: `Reviewed by` on #7, `Stage` Ready on #11. A single command per act removes the risk
- Rewriting an issue body in a cloud session needs the environment footer removed first, or footers pile up
- "Blocked by" takes the internal issue id, not the number
- A milestone due date must be sent at 12:00 UTC

## Platform constraints learned (already in 0.1)

Cloud sessions: REST only, `gh` 2.89, `gh auth status` false negative, footer on every body, no remote branch deletion, no releases. REST: issue field values per issue and in the list, no endpoint for field definitions, types, dependencies, duplicates. Milestone due date shift.

## Changes made before the test

Draft → 0.1, after the REST checks and before the first test issue: REST-only channel (D10, D20), leaving Waiting from `Executor` (D33), automatic branch deletion (D34), field ids in configuration (D35), "containing" `Closes #n`, new platform constraints.
