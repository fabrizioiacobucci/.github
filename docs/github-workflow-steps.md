# GitHub Workflow Standard: Steps

Version: 0.1 (core)
Last update: 2026-11-20
Decisions and constraints: `github-workflow.md`

Version 0.1 holds the frozen core (D31) on a REST-only channel (D10). Text under "Extensions" is not valid: never apply it until it has been executed (D32).

"Step" names a point of the workflow. `Stage` names the issue field. Never mix the two.

The step numbers are not a strict sequence. Steps 1, 2, 4, 5, 6, 7, 8 are the life of an issue. Steps 3 and 9 are the life of a version and happen whenever a version is decided or shipped.

## Steps

| # | Step | What happens | Detail |
|---|------|--------------|--------|
| 1 | Birth | The issue is created | done |
| 2 | Triage | Accepted, rejected, duplicate or Needs Info | done |
| 3 | Version opening | The milestone is created, with name and due date | done |
| 4 | Planning | Version, dependencies, preparation, approval to Ready | done |
| 5 | Pick-up | An executor claims the first startable issue | done |
| 6 | Work | On a branch named after the issue, with possible stops in Waiting | done |
| 7 | Review | Pull request opened, judged against the completion criterion | done |
| 8 | Closure | The Reviewer merges, the issue closes | done |
| 9 | Release | Tag, changelog, milestone closed | done |

## Step details

### 1. Birth

- Who: anyone (developer, agent, outside contributor)
- Does: creates the issue with the sections of its type. People use the issue form. Agents read the form file, reproduce its sections in the body and create the issue through REST with its type (`gh api repos/<owner>/<repo>/issues -f type=<Type>`)
- Leaves untouched: `Stage`, milestone, Approval section
- Result: the issue is in the triage queue (empty `Stage`)

### 2. Triage

- Who: whoever holds the triage mandate (person or agent)
- Does: takes an issue with empty `Stage` and picks one outcome
  - Accept: fixes the type if needed, sets Priority by the Priority criteria, sets `Stage` to Accepted. A Task with a clear "Done when" goes straight to Ready
  - Reject: closes as "not planned", with the reason in a comment
  - Duplicate: copies anything new into a comment on the surviving issue, then closes it through REST with `state_reason` duplicate and `duplicate_issue_id` set to the surviving issue. The more complete issue survives
  - Suspend: sets `Stage` to Needs Info and asks the question in a comment
- Leaves untouched: milestone, assignee, Approval section
- Result: the issue is accepted, closed, or waiting for clarification
- A wanted Task with an unclear "Done when" is Accepted, not Ready: the triager writes the criterion, or the issue goes to Waiting until the author answers. Needs Info is for an issue that cannot be decided at all; Waiting is for work already wanted
- Triage queue, oldest first: open issues with empty `Stage`, then Needs Info issues commented since they entered it. Until a script exists: people read it from the board (the column with no value and the Needs Info column); agents list the open issues through REST, which returns their field values, and filter locally

### 3. Version opening

- Who: whoever holds the planning mandate
- Does: creates the milestone in the repository with a name (`vX.Y.Z`, the same as the release tag), a due date (mandatory) and a one-line goal as description
- Leaves untouched: the issues
- Result: an open milestone exists, ready to receive issues
- A milestone may be opened well in advance: it is the only placeholder for a future version. Its issues become startable only when it is the next release
- A version that can be named gets its milestone at once, even with a provisional due date
- A group of issues with no version yet is tied together by a theme label. With nothing known, issues stay in the backlog and are chosen by Priority when a milestone is created

### 4. Planning

- Who: whoever holds the planning mandate. Anyone can prepare. The move to Ready belongs to the Approver
- Does: takes an Accepted issue and performs up to four acts, in any order except the last
  - Version: assigns the milestone if the target version is known; otherwise the issue stays in the backlog
  - Order: declares "blocked by" dependencies
  - Preparation: writes the Design section (Feature) or reproduces the defect (Bug)
  - Approval: the Approver writes its own line (D22) and sets `Stage` to Ready. This act closes preparation
- Leaves untouched: assignee
- Result: the issue is Ready, with or without a milestone

### 5. Pick-up

- Who: whoever holds the execution mandate (person or agent main session, never a subagent)
- Does: chooses the first startable issue, then sets `Executor` to its own name, assigns the issue and sets `Stage` to In Progress
- Leaves untouched: milestone, Priority, type
- Result: the issue is In Progress, with an assignee and an `Executor`
- Claim order: read the issue just before the claim and skip it if `Executor` is set, since another executor holds it. Write `Executor` first, read the issue again, go on only if `Executor` still carries your own name. Then assign the issue, set `Stage`, and read it once more with the same test
- Lost claim: an executor whose test fails withdraws and writes nothing more on that issue. `Executor`, assignee and `Stage` are the winner's, and what the loser already wrote in assignee and `Stage` carries the same values. It takes the next issue
- An executor with at least one issue In Progress picks up nothing new. Waiting and In Review do not count, so an issue coming back from Waiting may leave an executor with more than one In Progress

### 6. Work

- Who: the executor named in `Executor`
- Does: works on the issue's branch until the completion criterion is met
  - Needs a person: sets `Stage` to Waiting, with who is awaited and why in a comment. `Executor` and assignee stay
  - Gives up: sets `Stage` back to Ready and clears `Executor` and assignee
- Leaves untouched: milestone, Priority
- Result: the work is complete on the branch, ready for a pull request
- One branch per issue, created with `git` from `develop` and pushed, with the flat name `<number>-<short-title>`. No prefix for type, version or theme. No branch without an issue. The pull request links it to the issue (step 7). The branch is deleted automatically after the merge (D34)
- Claim check: before the first push of the branch, the executor reads the issue once more. If `Executor` no longer carries its own name, the claim is lost and it withdraws as in step 5
- Identity: before the first commit, the executor reads `git var GIT_COMMITTER_IDENT`. If it is not the identity of D39, the executor sets `user.name` and `user.email` in the clone's git config. Every commit is made with `git commit -s`
- Work found along the way: what the completion criterion needs is done inside the issue; everything else becomes a new issue that goes through triage. If the new issue blocks the current one, the executor pushes the branch, declares "blocked by", sets `Stage` back to Ready and clears `Executor` and assignee
- The executor never edits the Design or the completion criterion of its own issue. Any change to the completion criterion is substantial; a Design change is substantial when the approach or the parts touched change. A substantial change sends the issue back to Accepted, with `Executor` and assignee cleared: on a Feature or Bug the Approval section is emptied and a new approval is needed; a Task is set back to Ready by someone other than the executor
- When both rules apply, the Design or criterion rule comes first

### 7. Review

- Who: the executor opens the pull request. The Reviewer judges and is always different from the executor, with the same meaning of "different" as for the Approver
- Does:
  - The executor opens the pull request through REST from the issue's branch to `develop`, with the title `<type>(<scope>): <description> (#<issue>)` and a description containing `Closes #<issue>`, and sets the issue's `Stage` to In Review
  - On a Feature or Bug, the pull request also adds one line to `CHANGELOG.md` under "Unreleased", in the right Keep a Changelog category. A breaking change is marked in that line
  - The Reviewer judges the work against the issue's completion criterion, changelog line included
  - Approve: go to step 8
  - Reject: the reasons go in the pull request and `Stage` goes back to In Progress
- Leaves untouched: Design, completion criterion, milestone
- Result: the work is approved, or back with the executor with reasons
- The Reviewer writes its own name in `Reviewed by` when it approves (D27)
- Work that leaves no file: the executor writes the outcome in a comment and sets `Stage` to In Review; the Reviewer judges the comment and closes the issue by hand as completed. Work that produces a document goes through a pull request like code

### 8. Closure

- Who: the Reviewer who approved. The executor never merges its own work
- Does: merges the pull request with a squash merge, so that one issue is one commit, passing the pull request title as commit title, to keep the pull request number out of it, and the trailer lines of D37 as commit message (REST `PUT /repos/<owner>/<repo>/pulls/<n>/merge` with `merge_method` squash, `commit_title` and `commit_message`). The issue closes as completed (by hand if the link does not close it) and the branch is deleted automatically (D34)
- Leaves untouched: `Stage`, `Executor`, `Reviewed by`, which stay on the closed issue as a record
- Result: the issue is closed as completed, inside its milestone

### 9. Release

- Who: whoever holds the planning mandate. The tag and the release are made by a person: a cloud agent session cannot create releases (REST answers 403), and only a person making them has been executed (D32)
- Does: once every issue of the milestone is closed or moved as a leftover, in this order
  - Creates in the milestone the Task "Release vX.Y.Z", then accepts it and sets it to Ready with a Priority. From there it goes through steps 5 to 8 like any other. Its pull request, into `develop`, renames the "Unreleased" section of `CHANGELOG.md` to the version with its date and adds a new empty "Unreleased" section above it
  - A person creates the tag `vX.Y.Z` on the commit of that merge
  - A person publishes the release with that changelog section as notes
  - Closes the milestone
- Leaves untouched: the closed issues
- Result: the version is shipped and its milestone is closed
- Leftovers are moved before the milestone is closed
- Reopening: while the milestone is open, a reopened issue goes back to Accepted, with `Executor`, `Reviewed by` and assignee cleared. Once the milestone is closed, the issue is not reopened: a new issue citing it is created (a Bug with "Observed in" set to the shipped version, or a Feature or Task for something missing)
- An issue found open inside a closed milestone is closed again by the triager, who creates the new issue

## Issue forms

| Type | Required sections | Optional sections |
|------|-------------------|-------------------|
| Feature | Goal; Acceptance criteria | Design (required before Ready); Out of scope; Approval (last, filled at Ready) |
| Bug | What happens; What should happen; Steps to reproduce; Observed in | Approval (last, filled at Ready) |
| Task | What to do; Done when | Context |

- "Observed in" is free text: a release, or a branch and commit
- "Design" holds at least the approach and the parts of the software it touches
- "Approval" holds two lines, "Prepared by" and "Approved by", left empty at birth and filled with names taken from the `Executor` options

## Ready criteria

| Type | An issue becomes Ready when |
|------|-----------------------------|
| Feature | Its Design section is complete |
| Bug | It has been reproduced, or the attached evidence (log, trace) is judged sufficient. The outcome goes in a comment |
| Task | Its "Done when" section is clear. Set directly at triage, with no Approver |

- A Bug neither reproduced nor backed by evidence goes to Waiting, with a question to the reporter. With no answer it is closed as "not planned"

## Priority criteria

| Priority | Criterion, read top to bottom: the first that holds gives the value |
|----------|---------------------------------------------------------------------|
| Urgent | A shipped version cannot be used at all, or a step of this standard (steps 1 to 9) cannot be completed, with no workaround; or data is lost in normal use; or a security flaw has a known way to be exploited |
| High | An existing function cannot be used, with no workaround |
| Medium | What the issue fixes or adds is reached today only through a workaround; or what it adds has a concrete use named in the issue (who needs it and for what); or data loss or a security flaw is possible, with no normal-use path and no known exploit |
| Low | Any other issue |

- A workaround counts only if it is written in the issue or in a comment (structured datum: #15)
- Triage sets the value from the issue alone: "blocked by" dependencies never change it
- Limits, open in #17: Urgent does not move an issue across milestones (D21 still orders by milestone first), and a blocking issue keeps its own Priority, whatever it blocks (D36)

## Rules

- Creating and accepting are two distinct acts, even for the same actor. Whoever creates does not set `Stage`; with a triage mandate, triage is a separate act
- No draft items in the Project: every piece of work is born as an issue in a repository
- Priority is mandatory for every accepted issue. Startable issues are taken in the order of D21
- Board views filter on open issues (`is:open`): a closed issue keeps its last `Stage` value
- Abandoned work: the issue goes back to Ready; assignee and `Executor` are cleared
- Leftovers at release: an unfinished issue that is In Progress or In Review goes back to Ready, with assignee and `Executor` cleared; any other keeps its `Stage`. All move to the next milestone or to none
- Leaving Waiting: restore the previous `Stage` from `Executor` (D33): In Progress if it is set, Accepted if it is empty. Who is awaited and why stay in the entry comment
- Silence clock: 30 days from entering Needs Info, or from the last comment, whichever is later. The same clock applies to an issue in Waiting for an outside answer

## Extensions (not valid)

Text removed from the core steps, kept as written. It enters the core only after execution (D32).

### Parent and child issues

| Source | Removed text |
|--------|--------------|
| Step 3 | A group of issues with no version yet may also be tied together "by a parent issue" |
| Step 4 | Version act: "On a parent, the same act sets the same milestone on every child" |
| Step 4 | Act "Breakdown: creates sub-issues where the work is too large for one issue" (the core counts four acts instead of five) |
| Step 4 | "Sub-issues created here skip the triage queue: whoever breaks down an accepted parent creates them and accepts them right after, as a separate, declared act. A Task child with a clear 'Done when' goes straight to Ready. Without a triage mandate, the children keep an empty `Stage` and go to the queue" |
| Step 5 | "On the first child started, sets the parent to In Progress too" |
| Step 6 | One branch per "leaf" issue |
| Step 7 | "For a Feature made of sub-issues, the line is added by the pull request of the child that completes it" |
| Step 7 | "A parent In Review is checked by someone who executed none of its children, or by an agent started without their context, against the parent's own acceptance criteria" |
| Step 8 | "and if it was the last open child the parent goes to In Review" |
| Rules | Leftovers: "An unfinished parent moves together with its open children" |
| Rules | "Ready on a Feature with sub-issues means its sub-issues can be worked. An issue with open sub-issues is never picked up itself: work happens on the leaves" |
| Rules | "A parent follows its children: Ready to In Progress when the first child is picked up; In Progress to In Review when the last child is closed, to check the parent's own acceptance criteria; closed after that check. Whoever touches the first and the last child updates the parent. A parent never has an `Executor`; whoever checks its criteria writes its own name in `Reviewed by`" |
| Rules | "Children carry their parent's milestone, or none if the parent has none. A child meant for another version means the parent must be split. Whoever links an existing issue as a child sets its milestone by hand" |

### Hotfixes and parallel `release/` branches

| Source | Removed text |
|--------|--------------|
| Step 3 | "A hotfix milestone on a shipped version gets its `release/vX.Y.Z` branch when it is opened, cut from the tag of the version to fix" |
| Step 6 | Branch base "`--base <target branch>` (the target branch of D28)" (the core uses `--base develop`) |
| Step 8 | "An issue merged into a `release/` branch is closed by hand: the automatic close works only for the default branch" |
| Step 8 | "Anything merged into a `release/` branch, the release Task included, is carried, by the same Reviewer in the same act, to `develop` and to every later `release/` branch still open" |
| Step 9 | First release act: "Cuts `release/vX.Y.Z` from `develop`, unless it already exists"; the release Task's pull request goes "into the release branch" (the core uses `develop`) |

### Pull requests from outside contributors

| Source | Removed text |
|--------|--------------|
| Step 7 | "Pull request from an outside contributor with no issue: the triager creates the issue from it, links them, accepts the issue and sets it straight to In Review. Declared exception: it skips Ready" |

### Full periodic check

The whole section below is removed from the core.

- Who: whoever holds the triage mandate
- When: weekly
- The check only lists. Every remedy stays an act of whoever holds the mandate, never automatic

Data consistency:

1. The Project's native Status field is empty everywhere (`has:status` returns nothing)
2. On an open issue, `Executor` is set if and only if the issue was picked up and has not gone back to Ready or Accepted: In Progress, In Review, or Waiting entered after pick-up. Parents and issues created from an outside pull request never have one
3. `Reviewed by` and `Executor` differ
4. Every Feature or Bug in Ready has its Approval section signed, with two different names
5. Children carry their parent's milestone
6. Parents are in step with their children: none Ready with a child already started, none In Progress with every child closed
7. No open issue sits inside a closed milestone

Stalled work:

8. Needs Info issues, in two lists: commented since entry (to re-examine), silent for 30 days (to close as "not planned")
9. Waiting issues awaiting an outside answer, silent for 30 days
10. Issues left in Accepted and not prepared for 30 days
11. Issues In Progress or In Review with no activity for 7 days
12. Milestones past their due date with open issues

### Several repositories

No text removed from the steps: they already work on one repository.
