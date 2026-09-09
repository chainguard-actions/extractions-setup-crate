<!-- markdownlint-disable -->

# Hardening Report: extractions--setup-crate/v2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **extractions--setup-crate/v2** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The workflow uses tag-based (non-SHA-pinned) action references, making it vulnerable to supply-chain attacks if the tag is moved. Found: `actions/checkout@v6` (lines 12 and 47) and `actions/setup-node@v6` (line 13). All should be pinned to full 40-character commit SHAs, e.g. `actions/checkout@<sha> # v6`.

Locations:

- `.github/workflows/build.yaml:12`
- `.github/workflows/build.yaml:13`
- `.github/workflows/build.yaml:47`

### script-injection (severity: high)

Sub-rule (a): The `Test` step directly interpolates the expression `${{ matrix.repo }}` inside a `run:` shell command: `which $(printf ${{ matrix.repo }} | awk -F '/' '{print $2}' | tr '[:upper:]' '[:lower:]')`. The `matrix.repo` value flows through YAML template substitution before the shell parses it, allowing an attacker who controls the matrix value to inject arbitrary shell commands. The value should be passed via an `env:` variable and double-quoted in the shell script instead.

Locations:

- `.github/workflows/build.yaml:55`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key, and neither the `build` job nor the `test` job defines a job-level `permissions:` block. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad. A minimal `permissions:` block (e.g. `contents: read`) should be added at the top level or per job.

Locations:

- `.github/workflows/build.yaml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings in .github/workflows/build.yaml: (1) Pinned actions/checkout@v6 to SHA d23441a48e516b6c34aea4fa41551a30e30af803 and actions/setup-node@v6 to SHA 249970729cb0ef3589644e2896645e5dc5ba9c38, with tag comments preserved; (2) Moved ${{ matrix.repo }} out of the run: shell command into an env: variable MATRIX_REPO, referenced as double-quoted "$MATRIX_REPO" in the shell script with printf '%s' for safe handling; (3) Added top-level permissions: contents: read block to restrict the workflow token to the minimum required permissions.

