# Changelog

All notable changes to this project are documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [Unreleased]

### Fixed

- Criteria to choose a `Priority` value among Urgent, High, Medium and Low (#12)
- Step 9 states that the tag and the release are made by a person, since a cloud agent session cannot create releases (#10)
- The squash commit body holds only `Signed-off-by` and `Co-Authored-By` trailer lines, and the repository settings the standard needs are listed with their values (#19)
- The standard states the paths of the issue forms (`.github/ISSUE_TEMPLATE/`) and of the field configuration (`config/issue-fields.yml`) in the `.github` repository (#7)
- Each signer writes its own line of the Approval section: the preparer "Prepared by", the Approver "Approved by", the same in D22 and in step 4 (#8)
- Startable issues with the same milestone and `Priority` are ordered by lowest issue number, so the first startable issue is always one issue (#23)
- Branch commits carry the identity of the GitHub account and are signed off with it, and the executor checks and sets the identity before its first commit (#24)
- `Reviewed by` is written only when the Reviewer approves, never on a rejection, so a later round overwrites no earlier name (#9)
- `Executor` and `Reviewed by` options each name one person or one agent seat: the option "Owner" is the owner of the account, and D11 no longer uses the word for the assignee (#11)
- The claim of step 5 skips an issue whose `Executor` is set, is checked after its last write and again before the first push of the branch, and a lost claim leaves the issue to the winner, so that two executors working at the same time do not both go on with the same issue, except in one case that issue field writes cannot exclude, which the standard states as a known limit (#30)
- The standard states its design requirements (no signal relies on fixed lines in comments or on free text; no mechanism enters the core before it has been executed) and the ideas already rejected, with the reason of each (#18)
- The standard states that the forms, the configuration and the documents in force are those on `develop` of the `.github` repository, and that a change takes effect when merged into `develop`, not at release (#13)
