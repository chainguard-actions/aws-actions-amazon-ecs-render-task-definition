<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-render-task-definition/v1.8.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-render-task-definition/v1.8.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is directly interpolated inside a `run:` shell command. In package.yml line 17, `run: gh pr checkout ${{ github.event.pull_request.number }}` embeds the pull request number directly into the shell command string. Although a PR number is typically numeric, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it. An attacker controlling the expression value could inject shell metacharacters.

Locations:

- `.github/workflows/package.yml:17`

### unpinned-uses (severity: high)

All `uses:` references across all workflow files are pinned to mutable tags or version strings rather than immutable 40-character SHA commit digests. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved or compromised. Failing references:
- check.yml: `actions/checkout@v4` (line 11), `actions/setup-node@v4` (line 13), `actions/github-script@v7` (line 25)
- codeql-analysis.yml: `actions/checkout@v4` (line 30), `github/codeql-action/init@v3` (line 40), `github/codeql-action/autobuild@v3` (line 46), `github/codeql-action/analyze@v3` (line 55)
- notifications.yml: `actions/github-script@v7` (line 14), `slackapi/slack-github-action@v1.26.0` (lines 44, 57, 70)
- package.yml: `actions/checkout@v4` (line 13), `actions/setup-node@v4` (line 19)

Locations:

- `.github/workflows/check.yml:11`
- `.github/workflows/check.yml:13`
- `.github/workflows/check.yml:25`
- `.github/workflows/codeql-analysis.yml:30`
- `.github/workflows/codeql-analysis.yml:40`
- `.github/workflows/codeql-analysis.yml:46`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/notifications.yml:14`
- `.github/workflows/notifications.yml:44`
- `.github/workflows/notifications.yml:57`
- `.github/workflows/notifications.yml:70`
- `.github/workflows/package.yml:13`
- `.github/workflows/package.yml:19`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` block and no per-job `permissions:` block. Without explicit permissions, workflows inherit the default repository token permissions, which may be overly broad (e.g., write access to contents). Each workflow should declare the minimal permissions required.
- check.yml: no permissions declared at top level or in either job (`check`, `conventional-commits`).
- codeql-analysis.yml: no permissions declared at top level or in the `analyze` job.
- notifications.yml: no permissions declared at top level or in the `issue-notifications` job.

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three finding types across four workflow files:

1. script-injection (package.yml): Moved `${{ github.event.pull_request.number }}` into the step's `env:` block as `PR_NUMBER` and referenced it as `"$PR_NUMBER"` in the shell script.

2. unpinned-uses: Pinned all 13 action references to full 40-char commit SHAs with tag comments preserved: actions/checkout@v4→11d5960a, actions/setup-node@v4→49933ea5, actions/github-script@v7→f28e40c7, github/codeql-action/{init,autobuild,analyze}@v3→f3712979, slackapi/slack-github-action@v1.26.0→70cd7be8.

3. missing-permissions: Added top-level `permissions: {}` and minimal per-job permissions to check.yml (contents: read for check job; pull-requests: read for conventional-commits job), codeql-analysis.yml (actions: read, contents: read, security-events: write for analyze job), and notifications.yml (contents: read for issue-notifications job).

