<!-- markdownlint-disable -->

# Hardening Report: peter-evans--find-comment/v4.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--find-comment/v4.0.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.inputs.main_version }}` and `${{ github.event.inputs.target }}` are interpolated directly inside `run:` shell commands. These are user-controlled workflow_dispatch inputs that flow through YAML template substitution before the shell sees them, enabling arbitrary command injection. Offending lines:
  - `run: git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}`
  - `run: git push origin ${{ github.event.inputs.main_version }} --force`
Fix: move the values into `env:` variables and reference them as quoted shell variables (e.g., `"$MAIN_VERSION"`).

Locations:

- `.github/workflows/update-major-version.yml:29`
- `.github/workflows/update-major-version.yml:31`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no job within any of these files defines job-level `permissions:`. Without explicit permissions, workflows run with the default (often broad) token permissions. Each workflow should declare the minimal permissions required (e.g., `permissions: read-all` or specific scopes).

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable version tags instead of full 40-character SHA commit hashes. Mutable tags can be moved by the upstream repository, enabling supply-chain attacks. Failing references include:
  - `actions/checkout@v5` (ci.yml, automerge-dependabot.yml — via peter-evans/enable-pull-request-automerge@v3, update-major-version.yml)
  - `actions/setup-node@v5` (ci.yml)
  - `actions/upload-artifact@v4` (ci.yml)
  - `actions/download-artifact@v5` (ci.yml)
  - `peter-evans/create-pull-request@v7` (ci.yml)
  - `peter-evans/enable-pull-request-automerge@v3` (automerge-dependabot.yml)
  - `peter-evans/slash-command-dispatch@v4` (slash-command-dispatch.yml)
  All should be pinned to full SHA digests with a version comment.

Locations:

- `.github/workflows/ci.yml:16`
- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/slash-command-dispatch.yml:10`
- `.github/workflows/update-major-version.yml:19`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all three findings across four workflow files:

1. script-injection (update-major-version.yml lines 29, 31): Moved `${{ github.event.inputs.main_version }}` and `${{ github.event.inputs.target }}` into `env:` blocks and referenced them as quoted shell variables (`"$MAIN_VERSION"`, `"$TARGET"`) in the `run:` commands.

2. missing-permissions: Added top-level `permissions: contents: read` to all four workflow files (ci.yml, automerge-dependabot.yml, slash-command-dispatch.yml, update-major-version.yml). Added job-level permissions where write access is needed: `contents: write` + `pull-requests: write` for the package job in ci.yml; `pull-requests: write` for the automerge job; `contents: write` for the tag job in update-major-version.yml.

3. unpinned-uses: Pinned all 7 action references to full 40-character commit SHAs with version tag comments: actions/checkout@fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09 (v5), actions/setup-node@a0853c24544627f65ddf259abe73b1d18a591444 (v5), actions/upload-artifact@ea165f8d65b6e75b540449e92b4886f43607fa02 (v4), actions/download-artifact@634f93cb2916e3fdff6788551b99b062d0335ce0 (v5), peter-evans/create-pull-request@22a9089034f40e5a961c8808d113e2c98fb63676 (v7), peter-evans/enable-pull-request-automerge@a660677d5469627102a1c1e11409dd063606628d (v3), peter-evans/slash-command-dispatch@13bc09769d122a64f75aa5037256f6f2d78be8c4 (v4).

