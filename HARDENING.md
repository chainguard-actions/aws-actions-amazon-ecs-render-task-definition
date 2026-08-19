<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-render-task-definition/v1.8.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-render-task-definition/v1.8.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command. In the 'Checkout PR' step, `${{ github.event.pull_request.number }}` is embedded directly in the shell command `gh pr checkout ${{ github.event.pull_request.number }}`. Although `pull_request.number` is an integer and lower risk than string fields, any `${{ ... }}` expression inside a `run:` block is a script-injection finding per the check rules. The value should be passed via an `env:` variable and referenced as a quoted shell variable instead.

Locations:

- `.github/workflows/package.yml:18`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references:

.github/workflows/check.yml:
  - actions/checkout@v4 (line 11)
  - actions/setup-node@v4 (line 13)
  - actions/github-script@v7 (line 25)

.github/workflows/codeql-analysis.yml:
  - actions/checkout@v4 (line 29)
  - github/codeql-action/init@v3 (line 40)
  - github/codeql-action/autobuild@v3 (line 47)
  - github/codeql-action/analyze@v3 (line 58)

.github/workflows/notifications.yml:
  - actions/github-script@v7 (line 15)
  - slackapi/slack-github-action@v1.26.0 (lines 47, 62, 77)

.github/workflows/package.yml:
  - actions/checkout@v4 (line 14)
  - actions/setup-node@v4 (line 20)

Locations:

- `.github/workflows/check.yml:11`
- `.github/workflows/codeql-analysis.yml:29`
- `.github/workflows/notifications.yml:15`
- `.github/workflows/package.yml:14`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad). Each workflow should declare minimal required permissions.

- check.yml: no permissions declared at top-level or on the 'check' or 'conventional-commits' jobs.
- codeql-analysis.yml: no permissions declared at top-level or on the 'analyze' job.
- notifications.yml: no permissions declared at top-level or on the 'issue-notifications' job.

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (package.yml): Moved `${{ github.event.pull_request.number }}` from the `run:` shell command into an `env:` block as `PR_NUMBER`, referenced as `"$PR_NUMBER"` in the shell.

2. unpinned-uses: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v4 → 11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → 49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/github-script@v7 → f28e40c7f34bde8b3046d885e986cb6290c5673b
   - github/codeql-action/{init,autobuild,analyze}@v3 → f3712979fa5f215279b101dd0a2e3bdfb4353324
   - slackapi/slack-github-action@v1.26.0 → 70cd7be8e40a46e8b0eced40b0de447bdb42f68e

3. missing-permissions: Added top-level permissions blocks:
   - check.yml: permissions: {} (no special permissions needed)
   - codeql-analysis.yml: actions: read, contents: read, security-events: write
   - notifications.yml: contents: read, pull-requests: read, issues: read

