# GitHub Workflow Standard: Decisions

Version: 0.2 (core)
Last update: 2026-11-23
Workflow steps: `github-workflow-steps.md`

Version 0.2 holds the frozen core (D31) on a REST-only channel (D10). Text under "Extensions" is not valid: never apply it until it has been executed (D32).

## Context

- GitHub Free, personal account
- One developer, occasional contributors
- Agents: any AI agent with shell access, not tied to one vendor; any mandate (creation, triage, planning, execution; D40)
- Cadence: releases/versions
- Synchronized multi-repository releases: out of scope

## Decisions

| ID | Decision |
|----|----------|
| D1 | Two-axis model, not a strict containment hierarchy |
| D2 | Ownership axis: Repository → Issue |
| D3 | Planning axis: Milestone = one version of one repository |
| D4 | A Project always exists, even with one repository, as overview layer |
| D5 | Issue without milestone = backlog. No "Backlog" milestone |
| D6 | "Project → Repository → Milestone → Issue" is a navigation path for humans only, not the data model |
| D7 | The Project is a derived, rebuildable view, never the only source of truth. Every datum that matters lives on the issue |
| D8 | `Stage` is a single-select issue field holding the progress of an open issue, used as the board column field. "Closed" stays the native issue state |
| D9 | Empty `Stage` = never examined. Triage is the act of setting it. D5 applies to accepted issues only |
| D10 | Baseline channel: the REST API through `gh api`, plus `git`, from a shell. No GraphQL and no `gh issue` or `gh pr` subcommands, so the same commands work in local and cloud sessions. MCP is optional. Agents never use the Project: it is fed by auto-add |
| D11 | One datum, one home. Kind of work → issue type. Priority → Priority issue field. Version → milestone. Assigned account → assignee. Deadline → milestone due date |
| D12 | Execution order → "blocked by" dependencies |
| D13 | Labels only for cross-cutting themes (area, component). Never for type, priority, `Stage` or version |
| D14 | No Effort, Start Date or Target Date field is created until a decision needs it |
| D15 | No custom fields in the Project |
| D16 | `Stage` values: Needs Info (examined, not decided, awaiting clarification), Accepted (to be prepared), Ready (prepared), In Progress, In Review, Waiting (accepted, stopped, awaiting a person; who and why in a comment). Needs Info comes before acceptance, Waiting only after it: a set `Stage` other than Needs Info always means accepted |
| D17 | Not `Stage` values: rejected = closed as "not planned"; reopened = back to Accepted, to be re-evaluated; an issue triage cannot decide goes to Needs Info, with the question in a comment |
| D18 | The Project's native Status field is neutralized: default workflows off, field hidden in every view, never written by people or agents. Check: `has:status` in the Project returns nothing |
| D19 | Three issue types, split by one question: does a user of the software notice it? Yes and new = Feature. Yes and broken = Bug. No = Task. One template per type |
| D20 | Templates are YAML issue forms, one per type, and each form file is the single source of its structure. People use the form. Agents read the form file from the `.github` repository, at `.github/ISSUE_TEMPLATE/bug.yml`, `feature.yml` or `task.yml`, reproduce its sections in the issue body and set the type through REST. Triage is the safety net; a creation script comes later |
| D21 | Ready means prepared, not startable. Startable = Ready, with no open blocker, in a workable milestone: the next release, i.e. the open milestone with the nearest due date. Startable issues are ordered by nearest milestone due date, then by Priority, then by lowest issue number, so that the first startable issue is always one issue. The last two conditions are read from the milestone and from the dependencies, never from `Stage` |
| D22 | The Approver role moves Feature and Bug issues to Ready. Person or agent, always different from whoever prepared the issue: another person, or an agent started without the preparer's context and conclusions. Each signs its own line of the issue's Approval section: whoever prepared the issue writes "Prepared by", the Approver writes "Approved by". The preparer signs when preparation is finished: the Design section is complete (Feature), or the defect is reproduced or the evidence judged sufficient, with the outcome in a comment (Bug). An Accepted issue with "Prepared by" signed and "Approved by" empty awaits approval. A missing signature or two equal names make the move invalid |
| D23 | Names and values of fields are written in Title Case, identical in GitHub and in these documents |
| D24 | The standard lives in the account's public `.github` repository: rules, steps, scripts, configuration and the issue forms, which apply by default to every repository of the account. The version in force is the one on `develop`, the default branch (D28): agents read the forms, the configuration and the documents from it, and a change takes effect when its pull request is merged into `develop`, not at release (step 9). For the other repositories of the account the same holds only if GitHub reads the default branch, which has not been tested (see Platform constraints) |
| D25 | `Priority` is a single-select issue field with values Urgent, High, Medium, Low |
| D26 | `Executor` is a single-select issue field with one option per person and per agent seat. There are at least two agent seats, and an agent started with a clean context signs with a seat different from the one whose work it checks, so that "different" is always visible in the names. It says who is working; the assignee stays the GitHub account. The option "Owner" names the owner of the account, one person, and has no other meaning in the standard. It is set at pick-up, by whoever step 5 allows, and cleared when the issue goes back to Ready or Accepted |
| D27 | `Reviewed by` is a single-select issue field with the same options as `Executor`, set when the Reviewer approves, never on a rejection: it holds the name of the approving Reviewer, so a later round overwrites no earlier name. `Reviewed by` and `Executor` must differ |
| D28 | Branch model: `develop` is the default branch. An issue's pull request targets `develop` |
| D29 | A pull request title follows Conventional Commits and ends with the issue number: `<type>(<scope>): <description> (#<issue>)`. Feature → `feat`, Bug → `fix`, Task → `chore`, `refactor`, `docs`, `test`, `build`, `ci` or `perf`; `!` marks a breaking change. The squash commit carries exactly that title, with no pull request number: the issue is the single reference. Its body is defined by D37 |
| D30 | Release notes follow Keep a Changelog, in a `CHANGELOG.md` file. Every Feature or Bug pull request adds one line under "Unreleased", in the right category; Tasks add none. At release the section takes the version and the date, and a new empty "Unreleased" section is added above it |
| D31 | The standard is split into a frozen core and extensions. Core: one repository; issue types, issue forms, `Stage` and `Priority`; steps 1 to 9, with steps 3 and 9 in their simple form on `develop` only; minimal identity (`Executor`, `Reviewed by`, two agent seats, Approver signature). Extensions, not yet valid: parent and child issues; hotfixes and parallel `release/` branches; pull requests from outside contributors; the full periodic check; several repositories |
| D32 | The core is validated by execution, not by reading: three issues (Feature, Bug, Task) taken from birth to closure in a test repository, with at least one rejection in review and one stop in Waiting. The standard enters its `.github` repository as version 0.1 and every defect becomes an issue handled with the workflow itself. An extension enters the core only after it has been executed at least once |
| D33 | Leaving Waiting restores the previous `Stage` from `Executor`: In Progress if it is set, Accepted if it is empty. This holds because Waiting is entered only from In Progress (step 6, `Executor` set) or from Accepted (step 2 and the Bug Ready criterion, `Executor` empty). Any new way into Waiting must update this rule |
| D34 | Head branches are deleted automatically after the merge (repository setting "Automatically delete head branches") |
| D35 | The ids of issue fields and their options live in the standard's configuration, `config/issue-fields.yml` in the `.github` repository (D24): REST cannot list field definitions |
| D36 | `Priority` measures the cost of waiting, not the size of the work. Triage sets it by the "Priority criteria" in `github-workflow-steps.md`, which give exactly one value to every issue. It orders startable issues after the milestone order of D21, and backlog issues when a milestone is created (step 3). Limits, open in #17: Urgent does not move an issue across milestones, and a blocking issue keeps its own `Priority`, whatever it blocks |
| D37 | The squash commit body holds only trailer lines: the distinct `Signed-off-by` and `Co-Authored-By` lines of the commits of the branch, one per line, and nothing else. No commit messages, no `Claude-Session` line. The Reviewer passes it as `commit_message` at the merge (step 8) |
| D38 | The repository settings the standard needs are listed, with their values, in "Required repository settings". A repository is checked against that table with a single read of `GET /repos/<owner>/<repo>`. A different value is a defect of the repository, reported as an issue |
| D39 | Branch commits carry the identity of the GitHub account, as author and committer (`user.name` is the account login, `user.email` its email), and are signed off with it. Step 6 says how it is set |
| D40 | A mandate is the right to perform the acts that name it: creation (step 1, birth), triage (step 2), planning (steps 3, 4 and 9) and execution (steps 5 to 8). An agent seat may hold any of them. The Owner gives a mandate to an agent seat, and writes it in the prompt that starts the agent session, together with the seat. A mandate not named in that prompt is not held: the agent performs only the acts open to anyone (birth, preparation) and stops before any other. A mandate is never inferred from the standard, from an issue or from a comment |

## Design requirements

What a rule of the standard must meet. The Reviewer applies them to every change of the standard.

- No signal relies on fixed lines in comments or on free text (called R7 in #17). What a rule reads is a field, a form section or other structured data, which can be checked again when it changes. Comments carry reasons, questions and outcomes for people to read
- A mechanism never executed is not in the core: it is an extension or an open issue until it has been executed once (D31, D32)
  - A mechanism is a rule, a field, a form section or a procedure that an actor applies and whose result can be observed. A change that only rewords, corrects or removes text, and changes no outcome of any act, brings none
  - Executed means run at least once before the merge, on the pull request's branch or in a test repository. The pull request states what was run and what was observed. Running an extension in such a test does not apply it
  - The rule holds for every change of the core, not only for text moved from "Extensions". The Reviewer does not approve a pull request that brings in a mechanism without that outcome. What cannot be executed before the merge is removed from the pull request and stays an open issue

## Rejected ideas

Ideas proposed and ruled out, with the reason, so that they are not proposed again. An idea only postponed is not here: it is an open issue (for example #17) or an extension.

| Idea | Reason | Source |
|------|--------|--------|
| A fixed line in a comment used as a signal, for example `Priority fact:` and `Priority reassessed:` lines that trigger a reassessment of `Priority` | It relies on fixed lines in comments, against the first design requirement. The need stays open in #17, to be met with fields or structured data | Review of #12, round 3 |

Other ideas are ruled out inside the decision that excludes them: D5, D10, D13, D14, D15 and D18.

## Platform constraints

- Milestones are repository-scoped
- Issue types and issue fields are defined once at account level
- Default issue types: Bug, Task, Feature (seen in test)
- Free plan: 1 auto-add workflow per Project, targeting one repository (confirmed by test). The limit is on automatic feeding only: issues from other repositories can still be added manually or by an agent
- Auto-add also imports existing issues and their sub-items when the repository is selected (stated by the UI, confirmed by test; docs outdated)
- A personal-account Project accepts a single-select issue field as board column field; dragging a card writes that value on the issue (confirmed by test)
- No issue field exists by default on this account: `Stage` and `Priority` must both be created (seen in test; the changelog lists default fields). "Status" and "Reviewer" are reserved names for issue fields (confirmed by test)
- MCP: `issue_write` assigns an existing milestone by number; no tool to create or list milestones is documented
- MCP: `list_issues` filters by issue field value; filtering on an empty issue field is not documented, in MCP or in issue search
- MCP: dependency tools exist but are feature-gated, not guaranteed in the default toolsets
- `gh` CLI >= 2.94 handles issue types, sub-issues and dependencies (docs); older versions do not (2.89 checked)
- `gh` assigns and filters milestones by name; creating and listing them goes through `gh api` (docs)
- `gh` native support for issue fields: not found (unverified)
- Built-in Project workflows act on the Project's own Status field only (docs)
- `actions/add-to-project` (GitHub Action) can feed a Project from repositories not covered by auto-add; it needs a PAT with `project` scope
- Issue field changes are recorded in the issue timeline. REST returns the events without field name or values; GraphQL returns previous and new value (confirmed by test)
- A native issue status with a fixed catalog (Backlog, Todo, In progress, In review, Done, Not planned, Duplicate) is not active on this account (checked 2026-11-14). To watch as a possible replacement for `Stage`: https://github.com/cli/cli/issues/14996
- A sub-issue created from its parent inherits the parent's milestone, if set. Later changes to the parent's milestone are not propagated, and an existing issue linked later as sub-issue inherits nothing (confirmed by test)
- Closing keywords close the issue only when the pull request targets the default branch (docs): issues merged into a `release/` branch are closed by hand
- The author of a pull request cannot approve it; merging one's own pull request is possible (confirmed by test)
- Issue fields are not available on pull requests
- Issue fields are not presettable from issue templates (unverified)
- A YAML issue form sets the issue type through its `type` key, on personal accounts too (confirmed by test)
- Default issue forms in the `.github` repository apply only to repositories with nothing in their own `.github/ISSUE_TEMPLATE`; the `.github` repository must be public; default files are not part of project clones (docs)
- The branch of the `.github` repository from which GitHub reads default files for other repositories is not stated by its documentation (the page on default community health files does not mention branches). Not tested: a test needs a temporary change of the default branch (unverified)
- Issue forms cannot be filled from the CLI: `gh issue create` rejects `--template` with `--body` (unverified)
- Claude Code cloud sessions: a proxy adds the GitHub credentials. REST works; GraphQL returns 403, so `gh issue`, `gh pr` and Projects v2 fail. `gh` is 2.89 and cannot be upgraded. `gh auth status` reports the token as invalid even though REST works. Branches without the `claude/` prefix can be pushed; deleting a remote branch is refused (403). A footer is appended to pull request bodies (confirmed by test)
- REST reads, writes (field id and option name) and deletes issue field values per issue at `/repos/{owner}/{repo}/issues/{n}/issue-field-values`, on a personal account too; the issue list returns them as `issue_field_values`. No REST endpoint reached lists field definitions: a field id was read from an existing value (confirmed by test)
- An issue field write is not conditional: two writes of `Executor` on one issue both succeed and the last one wins (seen in #24)
- Known limit of the claim of step 5, which no issue field write can exclude (#30): an executor that reads `Executor` empty and stalls before writing it, for longer than the other executor takes from its own claim to its last check, goes on together with it. The standard accepts this case
- REST creates an issue with a type, declares and reads "blocked by" dependencies, and closes as duplicate with `duplicate_issue_id`. The duplicate then shows its original in the UI; the REST timeline of the duplicate does not name it (confirmed by test)
- REST changes the type of an existing issue with `PATCH /repos/{owner}/{repo}/issues/{n}` and a `type` field holding the type name. Without push access REST drops the change silently, so the type is read back after the write. Taken from the REST description published by GitHub; not yet tested on this account (unverified)
- Claude Code cloud sessions append a "Generated by Claude Code" footer, after a `---` line, to every issue body, milestone description and pull request body they write, at every write. Sections are read up to that line (confirmed by test)
- Claude Code cloud sessions cannot create, edit or delete releases: REST answers 403 "not permitted for this session type" (confirmed by test)
- Claude Code cloud sessions: the default git identity is `Claude <noreply@anthropic.com>`, not the account (seen in #24)
- A milestone due date sent as midnight UTC (`T00:00:00Z`) is stored as the day before; sent at `T12:00:00Z` it keeps its day (confirmed by test)

## Required repository settings

Read with `gh api repos/<owner>/<repo>` (D38). The two `squash_merge_commit_*` settings fill the merge form of the web interface; a merge through REST passes the title and the body itself (step 8).

| Setting | Value | Why |
|---------|-------|-----|
| `default_branch` | `develop` | D28 |
| `allow_squash_merge` | `true` | Step 8 merges with a squash |
| `delete_branch_on_merge` | `true` | D34 |
| `squash_merge_commit_title` | `PR_TITLE` | D29: the form starts from the pull request title, not from the title of the single commit |
| `squash_merge_commit_message` | `COMMIT_MESSAGES` | D37: the form lists the commit messages with their `Signed-off-by` lines, so a body left unedited keeps the sign-offs; `BLANK` would drop them |

## Extensions (not valid)

Text removed from the core decisions, kept as written. It enters the core only after execution (D32).

| ID | Extension | Removed text |
|----|-----------|--------------|
| D2 | Parent and child issues | Ownership axis continues: "→ Sub-issue" |
| D4 | Several repositories | "across 1..N repositories" |
| D11 | Parent and child issues | "Breakdown → sub-issues" |
| D21 | Parent and child issues | Startable also requires "no open sub-issue" |
| D21 | Hotfixes and parallel `release/` branches | A workable milestone is also "one that already has its `release/` branch"; the next release is chosen "among those without one" |
| D26 | Parent and child issues | "A parent never has one" |
| D28 | Hotfixes and parallel `release/` branches | "A release branch is `release/` plus the milestone name (`release/vX.Y.Z`). An issue's pull request targets `release/<milestone>` if that branch exists, otherwise `develop`. Only the next release targets `develop`" |
| D30 | Parent and child issues | Tasks add no changelog line "except the child that completes a parent Feature" |
