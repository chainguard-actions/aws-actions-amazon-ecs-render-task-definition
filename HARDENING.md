<!-- markdownlint-disable -->

# Hardening Report: aws-actions--amazon-ecs-render-task-definition/v1.8.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **aws-actions--amazon-ecs-render-task-definition/v1.8.5** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: A ${{ }} expression is directly interpolated inside a run: shell command. In package.yml, the step 'Checkout PR' runs: `gh pr checkout ${{ github.event.pull_request.number }}`. The pull request number is attacker-controllable and is injected directly into the shell command string before the shell ever sees it, enabling command injection via a crafted PR number.

Locations:

- `.github/workflows/package.yml:18`

### unpinned-uses (severity: high)

Multiple workflow files reference Actions using mutable version tags instead of pinned 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references: check.yml — actions/checkout@v4, actions/setup-node@v4, actions/github-script@v7; codeql-analysis.yml — actions/checkout@v4, github/codeql-action/init@v3, github/codeql-action/autobuild@v3, github/codeql-action/analyze@v3; notifications.yml — actions/github-script@v7, slackapi/slack-github-action@v1.26.0 (×3); package.yml — actions/checkout@v4, actions/setup-node@v4.

Locations:

- `.github/workflows/check.yml:12`
- `.github/workflows/check.yml:14`
- `.github/workflows/check.yml:24`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:47`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/notifications.yml:17`
- `.github/workflows/notifications.yml:44`
- `.github/workflows/notifications.yml:57`
- `.github/workflows/notifications.yml:70`
- `.github/workflows/package.yml:14`
- `.github/workflows/package.yml:20`

### missing-permissions (severity: medium)

Three workflow files have no top-level permissions: key and no per-job permissions: blocks. Without explicit permissions, workflows inherit the repository's default token permissions (often write-all), violating the principle of least privilege. Affected files: check.yml (two jobs: 'check' and 'conventional-commits', neither has permissions), codeql-analysis.yml (job 'analyze' has no permissions), notifications.yml (job 'issue-notifications' has no permissions). Note: package.yml passes because its single job has a job-level permissions block.

Locations:

- `.github/workflows/check.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/notifications.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across 4 workflow files:

1. script-injection (package.yml line 18): Moved `${{ github.event.pull_request.number }}` out of the run: shell string into an env var `PR_NUMBER: ${{ github.event.pull_request.number }}` and referenced it as `"$PR_NUMBER"` in the shell command.

2. unpinned-uses: Pinned all 13 action references to full commit SHAs:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - actions/github-script@v7 → @f28e40c7f34bde8b3046d885e986cb6290c5673b
   - github/codeql-action/init@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26
   - github/codeql-action/autobuild@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26
   - github/codeql-action/analyze@v3 → @b7351df727350dca84cb9d725d57dcf5bc82ba26
   - slackapi/slack-github-action@v1.26.0 → @70cd7be8e40a46e8b0eced40b0de447bdb42f68e (×3)

3. missing-permissions: Added top-level `permissions: {}` to check.yml, codeql-analysis.yml, and notifications.yml. Added job-level permissions blocks: check job gets `contents: read`, conventional-commits gets `{}`, codeql analyze job gets `actions: read / contents: read / security-events: write` (required for CodeQL), issue-notifications gets `{}`.

