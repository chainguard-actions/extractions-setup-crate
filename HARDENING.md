<!-- markdownlint-disable -->

# Hardening Report: extractions--setup-crate/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **extractions--setup-crate/v1.4.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression is interpolated directly inside a `run:` shell command string. On line 46 of build.yaml: `run: which $(printf ${{ matrix.repo }} | awk -F '/' '{print $2}' | tr '[:upper:]' '[:lower:]')`. The `${{ matrix.repo }}` value is substituted into the shell command before the shell parses it, allowing an attacker who controls matrix values (e.g. via a fork or workflow_dispatch) to inject arbitrary shell commands. The fix is to pass the value through an `env:` variable and double-quote it: `env: { REPO: "${{ matrix.repo }}" }` then `run: which $(printf '%s' "$REPO" | awk -F '/' '{print $2}' | tr '[:upper:]' '[:lower:]')`.

Locations:

- `.github/workflows/build.yaml:46`

### unpinned-uses (severity: high)

Multiple `uses:` references in build.yaml are pinned to mutable version tags rather than immutable 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references: `actions/checkout@v4` (lines 12 and 38), `actions/setup-node@v4` (line 13). Each should be pinned to a full SHA, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/build.yaml:12`
- `.github/workflows/build.yaml:13`
- `.github/workflows/build.yaml:38`

### missing-permissions (severity: medium)

build.yaml has no top-level `permissions:` key, and neither the `build` job nor the `test` job defines its own `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be `write-all`), granting broader access than necessary. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or to each job.

Locations:

- `.github/workflows/build.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/build.yaml: (1) script-injection on line 46 — moved `${{ matrix.repo }}` into an `env:` block as `REPO` and referenced it as `"$REPO"` with `printf '%s'` for safe shell expansion; (2) unpinned-uses — pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 (both occurrences) and actions/setup-node@v4 to SHA 49933ea5288caeca8642d1e84afbd3f7d6820020, preserving the tag in a comment; (3) missing-permissions — added top-level `permissions: contents: read` block to restrict the GITHUB_TOKEN to the minimum required.

