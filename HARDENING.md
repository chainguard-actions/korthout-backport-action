<!-- markdownlint-disable -->

# Hardening Report: korthout--backport-action/v4.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **korthout--backport-action/v4.6.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in release.yml directly interpolate `${{ }}` expressions into shell commands. These values flow through YAML template substitution before the shell sees them, enabling script injection.

1. Line 57: `run: git show ${{ steps.commit-new-release.outputs.commit_sha }} | cat` — step output interpolated directly into shell.
2. Line 75: `git tag v${{ steps.version.outputs.major }} --force` — step output interpolated directly.
3. Line 76: `git tag v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — step outputs interpolated directly.
4. Line 80: `git push origin v${{ steps.version.outputs.major }} --force` — step output interpolated directly.
5. Line 81: `git push origin v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — step outputs interpolated directly.
6. Line 93: `run: git show ${{ steps.commit-next-dev.outputs.commit_sha }} | cat` — step output interpolated directly.

Fix: route each value through an `env:` variable and reference it as a quoted shell variable, e.g. `env: COMMIT_SHA: ${{ steps.commit-new-release.outputs.commit_sha }}` then `git show "$COMMIT_SHA" | cat`.

Locations:

- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:75`
- `.github/workflows/release.yml:76`
- `.github/workflows/release.yml:80`
- `.github/workflows/release.yml:81`
- `.github/workflows/release.yml:93`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without an explicit permissions block, workflows inherit the repository's default token permissions, which may be overly broad (write access to all scopes). Each file should declare the minimal permissions required.

- ci.yml: read-only access to contents would suffice.
- pr-info.yml: needs `issues: write` (to post comments) and `pull-requests: read` at most.
- publish.yml: needs `contents: write` for pushing commits/tags.
- release.yml: needs `contents: write` for tagging and pushing.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/pr-info.yml:1`
- `.github/workflows/publish.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed 6 script injection locations in release.yml by moving all ${{ steps.*.outputs.* }} expressions into step-level env: blocks and referencing them as quoted shell variables ($COMMIT_SHA, $MAJOR, $MINOR). Added top-level permissions blocks to all four workflow files: ci.yml (contents: read), pr-info.yml (pull-requests: read, issues: write), publish.yml (contents: write), release.yml (contents: write).

