<!-- markdownlint-disable -->

# Hardening Report: korthout--backport-action/v4.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **korthout--backport-action/v4.4.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in release.yml directly interpolate `${{ ... }}` expressions into shell commands (sub-rule a), allowing expression values to be parsed by the shell before quoting can occur. Affected steps:
- Line 47: `run: git show ${{ steps.commit-new-release.outputs.commit_sha }} | cat`
- Line 67: `git tag v${{ steps.version.outputs.major }} --force`
- Line 68: `git tag v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force`
- Line 71: `git push origin v${{ steps.version.outputs.major }} --force`
- Line 72: `git push origin v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force`
- Line 82: `run: git show ${{ steps.commit-next-dev.outputs.commit_sha }} | cat`
These should be moved to `env:` variables and referenced as quoted `"$VAR"` in the shell.

Locations:

- `.github/workflows/release.yml:47`
- `.github/workflows/release.yml:67`
- `.github/workflows/release.yml:68`
- `.github/workflows/release.yml:71`
- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:82`

### missing-permissions (severity: medium)

Workflow files ci.yml, publish.yml, and release.yml have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. This means the workflows inherit the repository's default token permissions, which may be overly broad (e.g. write access to contents and pull-requests). Explicit minimal permissions should be declared.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/publish.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed all 6 script-injection locations in release.yml by moving ${{ }} expressions into env: blocks and referencing them as quoted shell variables. Added top-level permissions blocks to all three workflow files: ci.yml gets contents: read (read-only CI), publish.yml and release.yml get contents: write (both push commits and tags).

