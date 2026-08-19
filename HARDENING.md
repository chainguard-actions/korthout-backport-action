<!-- markdownlint-disable -->

# Hardening Report: korthout--backport-action/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **korthout--backport-action/v4.3.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in release.yml directly interpolate `${{ ... }}` expressions into shell commands, violating rule (a). This allows workflow-controllable values to be parsed as shell syntax before the shell ever sees them.

1. `run: git show ${{ steps.commit-new-release.outputs.commit_sha }} | cat` — `steps.commit-new-release.outputs.commit_sha` is interpolated directly into the shell command.
2. `git tag v${{ steps.version.outputs.major }} --force` and `git tag v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — step outputs interpolated directly.
3. `git push origin v${{ steps.version.outputs.major }} --force` and `git push origin v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — step outputs interpolated directly.
4. `run: git show ${{ steps.commit-next-dev.outputs.commit_sha }} | cat` — step output interpolated directly.

All of these should be moved to `env:` variables and referenced as `"$VAR"` in the shell script.

Locations:

- `.github/workflows/release.yml:39`
- `.github/workflows/release.yml:51`
- `.github/workflows/release.yml:55`
- `.github/workflows/release.yml:65`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions, violating the principle of least privilege.

- `ci.yml`: no permissions block at top level or job level.
- `publish.yml`: no permissions block at top level or job level.
- `release.yml`: no permissions block at top level or job level.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/publish.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in release.yml by moving all four ${{ steps.*.outputs.* }} expressions from run: shell commands into env: blocks, then referencing them as double-quoted shell variables (e.g., "$COMMIT_SHA", "$VERSION_MAJOR"). Added top-level permissions blocks to all three workflow files: ci.yml gets 'contents: read' (build/test only), publish.yml gets 'contents: write' (pushes commits/tags), and release.yml gets 'contents: write' (pushes commits/tags).

