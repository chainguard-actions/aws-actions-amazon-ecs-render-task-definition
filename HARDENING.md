<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-render-task-definition/v1.9.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-render-task-definition/v1.9.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The `run:` block in the 'Checkout PR' step directly interpolates `${{ github.event.pull_request.number }}` into a shell command (`gh pr checkout ${{ github.event.pull_request.number }}`). Any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted by the template engine before the shell ever sees it. The fix is to pass the value via an `env:` variable and reference it as a quoted shell variable: `env: PR_NUMBER: ${{ github.event.pull_request.number }}` then `run: gh pr checkout "$PR_NUMBER"`.

Locations:

- `.github/workflows/package.yml:18`

### unpinned-uses (severity: high)

All `uses:` references across the workflow files use mutable tag or version refs instead of immutable 40-character SHA commit digests, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references:
- check.yml: `actions/checkout@v4`, `actions/setup-node@v4`, `actions/github-script@v7`
- codeql-analysis.yml: `actions/checkout@v4`, `github/codeql-action/init@v3`, `github/codeql-action/autobuild@v3`, `github/codeql-action/analyze@v3`
- notifications.yml: `actions/github-script@v7`, `slackapi/slack-github-action@v1.26.0` (×3)
- package.yml: `actions/checkout@v4`, `actions/setup-node@v4`

Locations:

- `.github/workflows/check.yml:11`
- `.github/workflows/check.yml:13`
- `.github/workflows/check.yml:22`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:37`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:49`
- `.github/workflows/notifications.yml:14`
- `.github/workflows/notifications.yml:43`
- `.github/workflows/notifications.yml:56`
- `.github/workflows/notifications.yml:67`
- `.github/workflows/package.yml:14`
- `.github/workflows/package.yml:21`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g., `write` to contents). Each workflow should declare the minimal permissions required.
- check.yml: jobs `check` and `conventional-commits` both lack permissions.
- codeql-analysis.yml: job `analyze` lacks permissions.
- notifications.yml: job `issue-notifications` lacks permissions.

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (package.yml): Moved `${{ github.event.pull_request.number }}` into an `env:` block as `PR_NUMBER` and referenced it as `"$PR_NUMBER"` in the shell command.

2. unpinned-uses: Pinned all 13 `uses:` references to full 40-char SHA digests with tag comments preserved:
   - actions/checkout@v4 → 34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → 49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/github-script@v7 → f28e40c7f34bde8b3046d885e986cb6290c5673b
   - github/codeql-action/{init,autobuild,analyze}@v3 → b7351df727350dca84cb9d725d57dcf5bc82ba26
   - slackapi/slack-github-action@v1.26.0 → 70cd7be8e40a46e8b0eced40b0de447bdb42f68e (×3)

3. missing-permissions: Added minimal permissions blocks to all jobs lacking them:
   - check.yml `check`: contents: read
   - check.yml `conventional-commits`: contents: read, pull-requests: read
   - codeql-analysis.yml `analyze`: actions: read, contents: read, security-events: write
   - notifications.yml `issue-notifications`: contents: read, pull-requests: read, issues: read

