<!-- markdownlint-disable -->

# Hardening Report: antoniovazquezblanco--setup-ghidra/v2.1.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **antoniovazquezblanco--setup-ghidra/v2.1.6** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: `${{ github.event.release.tag_name }}` is directly interpolated inside a `run:` shell block. The offending line is: `TAG_NAME="${{ github.event.release.tag_name }}"`. A release tag name is attacker-controlled (anyone who can create a GitHub release can set the tag name), so this allows shell command injection. The value should be passed via an `env:` variable and the shell variable should be double-quoted: `env: TAG_NAME: ${{ github.event.release.tag_name }}` and then `TAG_NAME="$TAG_NAME"` in the script.

Locations:

- `.github/workflows/release.yml:22`

### broad-permissions (severity: medium)

The workflow file sets `permissions: read-all` at the top level. This grants overly broad read access across all scopes and should be replaced with specific minimal permissions (e.g., `contents: read`, `security-events: write`, `id-token: write`) scoped only to what each job actually needs.

Locations:

- `.github/workflows/scorecard.yml:9`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, broad-permissions

**Notes:**

1. release.yml (line 22): Moved `${{ github.event.release.tag_name }}` from the run: shell block into an `env:` block as `TAG_NAME`. Updated the script to use `"$TAG_NAME"` (double-quoted shell variable) instead of the inline GitHub expression, preventing shell command injection via attacker-controlled release tag names.
2. scorecard.yml (line 9): Replaced `permissions: read-all` with specific minimal `permissions: contents: read` at the top level. The job-level already scopes the necessary write permissions (security-events: write, id-token: write) for the Scorecard analysis.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in `.github/workflows/release.yml` 'Update major version tag' step: added double quotes around `v$MAJOR` and `$TAG_NAME` in the `git tag -f` command. The `TAG_NAME` env var was already properly populated from the `env:` block; the only missing piece was quoting the variables in the shell command to prevent metacharacter injection from an attacker-controlled release tag name.

