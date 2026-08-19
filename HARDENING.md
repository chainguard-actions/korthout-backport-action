<!-- markdownlint-disable -->

# Hardening Report: korthout--backport-action/v4.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **korthout--backport-action/v4.5.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: Multiple `run:` blocks in release.yml directly interpolate `${{ ... }}` expressions inside shell commands, bypassing shell quoting. Affected lines:
- Line 58: `run: git show ${{ steps.commit-new-release.outputs.commit_sha }} | cat` — step output injected directly into shell command.
- Line 76: `git tag v${{ steps.version.outputs.major }} --force` — step output injected directly.
- Line 77: `git tag v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — step outputs injected directly.
- Line 82: `git push origin v${{ steps.version.outputs.major }} --force` — step output injected directly.
- Line 83: `git push origin v${{ steps.version.outputs.major }}.${{ steps.version.outputs.minor }} --force` — step outputs injected directly.
- Line 96: `run: git show ${{ steps.commit-next-dev.outputs.commit_sha }} | cat` — step output injected directly into shell command.
All `${{ ... }}` expressions are substituted by the Actions runner before the shell parses the command, allowing shell metacharacters in the values to be interpreted.

Locations:

- `.github/workflows/release.yml:58`
- `.github/workflows/release.yml:76`
- `.github/workflows/release.yml:77`
- `.github/workflows/release.yml:82`
- `.github/workflows/release.yml:83`
- `.github/workflows/release.yml:96`

### missing-permissions (severity: medium)

Four workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions. Each file should declare minimal required permissions.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/pr-info.yml:1`
- `.github/workflows/publish.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions

**Notes:**

Fixed script-injection in release.yml by moving all 6 ${{ }} expressions from run: shell commands into env: blocks and referencing them as quoted environment variables. Fixed missing-permissions in all 4 workflow files (ci.yml, pr-info.yml, publish.yml, release.yml) by adding top-level 'permissions: {}' and minimal job-level permissions: contents:read for ci, pull-requests:write for pr-info, contents:write for publish and release.

