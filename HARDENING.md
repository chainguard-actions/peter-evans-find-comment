<!-- markdownlint-disable -->

# Hardening Report: peter-evans--find-comment/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **peter-evans--find-comment/v3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct expression interpolation of user-controlled `workflow_dispatch` inputs inside `run:` shell commands. `${{ github.event.inputs.main_version }}` and `${{ github.event.inputs.target }}` are interpolated directly into `git tag` and `git push` commands, allowing an attacker with workflow_dispatch access to inject arbitrary shell commands.

Locations:

- `.github/workflows/update-major-version.yml:28`
- `.github/workflows/update-major-version.yml:30`

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable tags instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include: actions/checkout@v4, actions/setup-node@v4, actions/upload-artifact@v4, actions/download-artifact@v4, peter-evans/create-pull-request@v6 (ci.yml); peter-evans/enable-pull-request-automerge@v3 (automerge-dependabot.yml); peter-evans/slash-command-dispatch@v4 (slash-command-dispatch.yml); actions/checkout@v4 (update-major-version.yml).

Locations:

- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:26`
- `.github/workflows/ci.yml:27`
- `.github/workflows/ci.yml:35`
- `.github/workflows/ci.yml:155`
- `.github/workflows/automerge-dependabot.yml:9`
- `.github/workflows/slash-command-dispatch.yml:9`
- `.github/workflows/update-major-version.yml:19`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual jobs define job-level `permissions:` keys. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all three finding types across four workflow files:

1. **script-injection** (update-major-version.yml): Moved `${{ github.event.inputs.main_version }}` and `${{ github.event.inputs.target }}` out of `run:` shell commands into `env:` blocks as `MAIN_VERSION` and `TARGET`, then referenced them as `"$MAIN_VERSION"` and `"$TARGET"` in the shell scripts.

2. **unpinned-uses**: Pinned all action references to full 40-character commit SHAs:
   - `actions/checkout@v4` → `@34e114876b0b11c390a56381ad16ebd13914f8d5` (ci.yml ×3, update-major-version.yml ×1)
   - `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020` (ci.yml)
   - `actions/upload-artifact@v4` → `@ea165f8d65b6e75b540449e92b4886f43607fa02` (ci.yml ×2)
   - `actions/download-artifact@v4` → `@d3f86a106a0bac45b974a628896c90dbdf5c8093` (ci.yml ×3)
   - `peter-evans/create-pull-request@v6` → `@c5a7806660adbe173f04e3e038b0ccdcd758773c` (ci.yml)
   - `peter-evans/enable-pull-request-automerge@v3` → `@a660677d5469627102a1c1e11409dd063606628d` (automerge-dependabot.yml)
   - `peter-evans/slash-command-dispatch@v4` → `@13bc09769d122a64f75aa5037256f6f2d78be8c4` (slash-command-dispatch.yml)

3. **missing-permissions**: Added top-level `permissions:` blocks to all four workflow files with minimal required permissions:
   - ci.yml: `contents: read` at top level; `contents: write` + `pull-requests: write` on the `package` job
   - automerge-dependabot.yml: `contents: write` + `pull-requests: write`
   - slash-command-dispatch.yml: `contents: read`
   - update-major-version.yml: `contents: write` (needed for git push)

