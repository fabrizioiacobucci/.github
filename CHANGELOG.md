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
