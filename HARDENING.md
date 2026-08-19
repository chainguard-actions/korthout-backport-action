<!-- markdownlint-disable -->

# Hardening Report: korthout--backport-action/v4.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **korthout--backport-action/v4.5.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

release.yml contains multiple run: blocks that directly interpolate ${{ }} expressions into shell commands (rule a). This allows template substitution to inject arbitrary shell metacharacters before the shell ever sees the string.

- Line 39: `run: git show ${{ steps.commit-new-release.outputs.commit_sha }} | cat` — steps output interpolated directly.
- Line 53: `git tag v${{ steps.version.outputs.major }} --force` — steps output interpolated directly.
- Line 54: `git tag v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — steps output interpolated directly.
- Line 58: `git push origin v${{ steps.version.outputs.major }} --force` — steps output interpolated directly.
- Line 59: `git push origin v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — steps output interpolated directly.
- Line 70: `run: git show ${{ steps.commit-next-dev.outputs.commit_sha }} | cat` — steps output interpolated directly.

Fix: move each value into an env: variable and reference it as a quoted shell variable (e.g. `"$SHA"`) instead of using `${{ }}` directly inside the run: script.

Locations:

- `.github/workflows/release.yml:39`
- `.github/workflows/release.yml:53`
- `.github/workflows/release.yml:54`
- `.github/workflows/release.yml:58`
- `.github/workflows/release.yml:59`
- `.github/workflows/release.yml:70`

### missing-permissions (severity: medium)

These workflow files have no top-level permissions: key and no job-level permissions: key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) access. Each file should declare minimal required permissions.

- ci.yml: no permissions defined at top-level or job level.
- pr-info.yml: no permissions defined at top-level or job level.
- publish.yml: no permissions defined at top-level or job level.
- release.yml: no permissions defined at top-level or job level.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/pr-info.yml:1`
- `.github/workflows/publish.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 6 script injection locations in release.yml by moving ${{ steps.*.outputs.* }} expressions into env: blocks and referencing them as quoted shell variables ($COMMIT_SHA, $MAJOR, $MINOR). Added top-level `permissions: {}` to all four workflow files (ci.yml, pr-info.yml, publish.yml, release.yml) since they all use either SSH deploy keys or PATs for privileged operations rather than GITHUB_TOKEN write access.

