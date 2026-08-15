<!-- markdownlint-disable -->

# Hardening Report: antoniovazquezblanco--setup-ghidra/v2.1.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **antoniovazquezblanco--setup-ghidra/v2.1.4** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ github.event.release.tag_name }}` is directly interpolated inside a `run:` shell command string. The template engine substitutes this value before the shell parses the script, so a malicious release tag name (e.g. containing `;`, `$(...)`, or backticks) could inject arbitrary shell commands. The offending line is: `TAG_NAME="${{ github.event.release.tag_name }}"`

Locations:

- `.github/workflows/release.yml:20`

### unpinned-uses (severity: high)

Multiple `uses:` references across workflow files are pinned to mutable tags or version strings rather than immutable 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

.github/workflows/main.yml:
- `actions/checkout@v7` (lint, build, test jobs)
- `actions/setup-node@v6` (lint, build, test jobs)
- `EndBug/add-and-commit@v10.0.0` (lint and build jobs)
- `ArtiomTr/jest-coverage-report-action@v2.3.1` (test job)

.github/workflows/release.yml:
- `actions/checkout@v7`

.github/workflows/codeql.yml:
- `actions/checkout@v7`
- `github/codeql-action/init@v4`
- `github/codeql-action/analyze@v4`

Locations:

- `.github/workflows/main.yml:13`
- `.github/workflows/main.yml:18`
- `.github/workflows/main.yml:26`
- `.github/workflows/main.yml:34`
- `.github/workflows/main.yml:39`
- `.github/workflows/main.yml:47`
- `.github/workflows/main.yml:57`
- `.github/workflows/main.yml:62`
- `.github/workflows/main.yml:70`
- `.github/workflows/main.yml:78`
- `.github/workflows/release.yml:13`
- `.github/workflows/codeql.yml:21`
- `.github/workflows/codeql.yml:24`
- `.github/workflows/codeql.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection in release.yml by moving `${{ github.event.release.tag_name }}` into the step's `env:` block as `TAG_NAME`, then referencing it as `$TAG_NAME` in the shell script. Fixed all unpinned-uses across main.yml, release.yml, and codeql.yml by pinning every `uses:` reference to its full 40-character commit SHA (with the original tag preserved as a comment): actions/checkout@v7→9c091bb21b7c1c1d1991bb908d89e4e9dddfe3e0, actions/setup-node@v6→249970729cb0ef3589644e2896645e5dc5ba9c38, EndBug/add-and-commit@v10.0.0→290ea2c423ad77ca9c62ae0f5b224379612c0321, ArtiomTr/jest-coverage-report-action@v2.3.1→262a7bb0b20c4d1d6b6b026af0f008f78da72788, github/codeql-action/init@v4→7188fc363630916deb702c7fdcf4e481b751f97a, github/codeql-action/analyze@v4→7188fc363630916deb702c7fdcf4e481b751f97a.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted $TAG_NAME expansions in .github/workflows/release.yml. Changed `$TAG_NAME` to `"$TAG_NAME"` in both locations: (1) the [[ ]] conditional test on line 22, and (2) the `git tag` command on line 26. The variable was already correctly sourced from the env: block rather than inline ${{ }} interpolation, so only the quoting needed to be added.

