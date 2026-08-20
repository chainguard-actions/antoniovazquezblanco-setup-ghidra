<!-- markdownlint-disable -->

# Hardening Report: antoniovazquezblanco--setup-ghidra/v2.1.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **antoniovazquezblanco--setup-ghidra/v2.1.5** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.release.tag_name }}` is directly interpolated inside a `run:` shell command string in release.yml. This allows an attacker who can create a release with a crafted tag name to inject arbitrary shell commands. The offending line is: `TAG_NAME="${{ github.event.release.tag_name }}"`  — the expression is expanded by the GitHub Actions template engine before the shell ever sees it, bypassing any shell quoting. The value should instead be passed via an `env:` variable and the shell variable should be double-quoted: `env: TAG_NAME: ${{ github.event.release.tag_name }}` then `TAG_NAME="$TAG_NAME"` in the script.

Locations:

- `.github/workflows/release.yml:22`

### broad-permissions (severity: medium)

scorecard.yml sets `permissions: read-all` at the top level. This grants overly broad read access to all GitHub Actions scopes for every job in the workflow. It should be replaced with specific minimal permissions (e.g. `contents: read`, `security-events: write`) scoped to what each job actually requires.

Locations:

- `.github/workflows/scorecard.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, broad-permissions

**Notes:**

1. release.yml (line 22): Moved `${{ github.event.release.tag_name }}` out of the `run:` shell string into an `env:` block as `TAG_NAME`. The shell script now uses `"$TAG_NAME"` (double-quoted env var) instead of the template expression, preventing shell injection via crafted release tag names.
2. scorecard.yml (line 9): Replaced `permissions: read-all` with `permissions: contents: read` at the top level. The job-level permissions (`security-events: write`, `id-token: write`) were already minimal and correctly scoped, so they were left unchanged.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/release.yml line 28: changed `git tag -f v$MAJOR $TAG_NAME` to `git tag -f "v$MAJOR" "$TAG_NAME"`. Both variables are now double-quoted, preventing shell metacharacter injection from an attacker-controlled release tag name. The TAG_NAME value was already correctly isolated in the step's `env:` block rather than being directly interpolated via `${{ }}` in the run script.

