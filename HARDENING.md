<!-- markdownlint-disable -->

# Hardening Report: antoniovazquezblanco--setup-ghidra/v2.1.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **antoniovazquezblanco--setup-ghidra/v2.1.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable tags or version strings instead of immutable 40-character SHA commit digests. This exposes the workflow to supply-chain attacks if the referenced tag is moved or the repository is compromised.

.github/workflows/main.yml:
  - uses: actions/checkout@v6
  - uses: actions/setup-node@v6
  - uses: EndBug/add-and-commit@v10.0.0
  - uses: ArtiomTr/jest-coverage-report-action@v2.3.1

.github/workflows/release.yml:
  - uses: actions/checkout@v6

.github/workflows/codeql.yml:
  - uses: actions/checkout@v6
  - uses: github/codeql-action/init@v4
  - uses: github/codeql-action/analyze@v4

Locations:

- `.github/workflows/main.yml:13`
- `.github/workflows/main.yml:18`
- `.github/workflows/main.yml:24`
- `.github/workflows/main.yml:36`
- `.github/workflows/main.yml:41`
- `.github/workflows/main.yml:47`
- `.github/workflows/main.yml:62`
- `.github/workflows/main.yml:67`
- `.github/workflows/main.yml:75`
- `.github/workflows/release.yml:13`
- `.github/workflows/codeql.yml:20`
- `.github/workflows/codeql.yml:23`
- `.github/workflows/codeql.yml:27`

### script-injection (severity: high)

Sub-rule (a): In .github/workflows/release.yml, the run: block directly interpolates the GitHub Actions expression `${{ github.event.release.tag_name }}` into a shell command string. A release tag name is attacker-controlled (any user with permission to publish a release can set it to an arbitrary string), so this allows shell command injection. The value is assigned to TAG_NAME via: `TAG_NAME="${{ github.event.release.tag_name }}"`. The fix is to pass the value through an environment variable and quote it: set `env: TAG_NAME: ${{ github.event.release.tag_name }}` and reference `"$TAG_NAME"` in the script.

Locations:

- `.github/workflows/release.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

Fixed all unpinned action references across main.yml, release.yml, and codeql.yml by pinning each to its full 40-character SHA commit digest (with original tag preserved as a comment). Fixed script injection in release.yml by moving `${{ github.event.release.tag_name }}` into the step's `env:` block as `TAG_NAME` and referencing it as `"$TAG_NAME"` in the shell script.

