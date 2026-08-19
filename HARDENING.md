<!-- markdownlint-disable -->

# Hardening Report: korthout--backport-action/v4.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **korthout--backport-action/v4.5.1** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: steps in release.yml directly interpolate ${{ steps.*.outputs.* }} expressions into shell commands (sub-rule a). These values flow through YAML template substitution before the shell sees them, enabling command injection if the step outputs contain shell metacharacters. Affected steps and lines:
- Line 49: `run: git show ${{ steps.commit-new-release.outputs.commit_sha }} | cat`
- Line 63: `git tag v${{ steps.version.outputs.major }} --force`
- Line 64: `git tag v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force`
- Line 68: `git push origin v${{ steps.version.outputs.major }} --force`
- Line 69: `git push origin v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force`
- Line 79: `run: git show ${{ steps.commit-next-dev.outputs.commit_sha }} | cat`
Fix: move the values into env: variables and reference them as quoted shell variables (e.g. "$COMMIT_SHA").

Locations:

- `.github/workflows/release.yml:49`
- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:64`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:69`
- `.github/workflows/release.yml:79`

### missing-permissions (severity: medium)

ci.yml has no top-level permissions: key and its only job (build) also has no job-level permissions: key. Without an explicit permissions block, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/ci.yml:1`

### missing-permissions (severity: medium)

pr-info.yml has no top-level permissions: key and its only job (comment) also has no job-level permissions: key. This workflow uses pull_request_target (a privileged trigger) and a PAT, making the absence of explicit permissions especially risky.

Locations:

- `.github/workflows/pr-info.yml:1`

### missing-permissions (severity: medium)

publish.yml has no top-level permissions: key and its only job (publish) also has no job-level permissions: key. Without an explicit permissions block, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/publish.yml:1`

### missing-permissions (severity: medium)

release.yml has no top-level permissions: key and its only job (release) also has no job-level permissions: key. Without an explicit permissions block, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 5 findings across 4 workflow files:

1. release.yml - script-injection: Moved all 6 ${{ steps.*.outputs.* }} expressions into env: blocks and referenced them as quoted shell variables. Specifically: COMMIT_SHA for the two 'git show' steps, and MAJOR/MINOR for the tag creation and push steps.

2. release.yml - missing-permissions: Added `permissions: contents: write` at the top level (required for git tag/push operations).

3. ci.yml - missing-permissions: Added `permissions: contents: read` at the top level (CI only reads code and runs tests).

4. pr-info.yml - missing-permissions: Added `permissions: contents: read` at the top level (comment posting uses a PAT, so GITHUB_TOKEN only needs read access).

5. publish.yml - missing-permissions: Added `permissions: contents: write` at the top level (required for git push operations via the EndBug/add-and-commit action and the tag push step).

