<!-- markdownlint-disable -->

# Hardening Report: antoniovazquezblanco--setup-ghidra/v2.1.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **antoniovazquezblanco--setup-ghidra/v2.1.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tags instead of pinned full-length SHA commit hashes. This exposes the workflow to supply-chain attacks where a tag can be silently moved to point to malicious code.

.github/workflows/main.yml:
  - uses: actions/checkout@v6
  - uses: actions/setup-node@v6
  - uses: EndBug/add-and-commit@v10.0.0
  - uses: ArtiomTr/jest-coverage-report-action@v2.3.1

.github/workflows/codeql.yml:
  - uses: actions/checkout@v6
  - uses: github/codeql-action/init@v4
  - uses: github/codeql-action/analyze@v4

.github/workflows/release.yml:
  - uses: actions/checkout@v6

All of these should be pinned to a full 40-character hex commit SHA (e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4).

Locations:

- `.github/workflows/main.yml:13`
- `.github/workflows/main.yml:18`
- `.github/workflows/main.yml:26`
- `.github/workflows/main.yml:36`
- `.github/workflows/main.yml:41`
- `.github/workflows/main.yml:49`
- `.github/workflows/main.yml:63`
- `.github/workflows/main.yml:68`
- `.github/workflows/main.yml:77`
- `.github/workflows/codeql.yml:28`
- `.github/workflows/codeql.yml:31`
- `.github/workflows/codeql.yml:36`
- `.github/workflows/release.yml:11`

### script-injection (severity: high)

Sub-rule (a): In .github/workflows/release.yml, the run: block directly interpolates the GitHub Actions expression ${{ github.event.release.tag_name }} into a shell command string. This value flows through YAML template substitution before the shell processes it, allowing an attacker who can control the release tag name to inject arbitrary shell commands. The offending line is:

  TAG_NAME="${{ github.event.release.tag_name }}"

The fix is to pass the value via an env: variable and reference it as a quoted shell variable (e.g. "$TAG_NAME") without any ${{ }} expression inside the run: block.

Locations:

- `.github/workflows/release.yml:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all three workflow files:

1. main.yml: Pinned actions/checkout@v6 → SHA (v4), actions/setup-node@v6 → SHA (v4), EndBug/add-and-commit@v10.0.0 → SHA, ArtiomTr/jest-coverage-report-action@v2.3.1 → SHA across all three jobs (lint, build, test).

2. codeql.yml: Pinned actions/checkout@v6 → SHA (v4), github/codeql-action/init@v4 → SHA (v3), github/codeql-action/analyze@v4 → SHA (v3).

3. release.yml: Pinned actions/checkout@v6 → SHA (v4). Fixed script injection by moving ${{ github.event.release.tag_name }} out of the run: block into an env: block as TAG_NAME, and referencing it as $TAG_NAME / "$TAG_NAME" in the shell script.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed three unquoted variable usages in .github/workflows/release.yml: (1) Changed `if [[ $TAG_NAME =~ ^v([0-9]+)\. ]]` to `if [[ "$TAG_NAME" =~ ^v([0-9]+)\. ]]` to prevent word splitting and shell metacharacter interpretation of the untrusted github.event.release.tag_name value inside [[ ]]; (2) Changed `git tag -f v$MAJOR` to `git tag -f "v$MAJOR"` to properly quote the tag argument; (3) Changed `git push origin v$MAJOR` to `git push origin "v$MAJOR"` to properly quote the ref argument. The TAG_NAME variable was already correctly placed in the step's env: block rather than being interpolated directly into the shell script.

