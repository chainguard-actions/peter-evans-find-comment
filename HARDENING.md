<!-- markdownlint-disable -->

# Hardening Report: peter-evans--find-comment/v3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **peter-evans--find-comment/v3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference actions using mutable tags instead of pinned full-length SHA commit hashes, making them vulnerable to supply-chain attacks if the referenced tag is moved or compromised.

.github/workflows/automerge-dependabot.yml:
  - uses: peter-evans/enable-pull-request-automerge@v3

.github/workflows/ci.yml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
  - uses: actions/upload-artifact@v4 (×2)
  - uses: actions/checkout@v4
  - uses: actions/download-artifact@v4 (×2)
  - uses: peter-evans/create-pull-request@v6

.github/workflows/slash-command-dispatch.yml:
  - uses: peter-evans/slash-command-dispatch@v4

.github/workflows/update-major-version.yml:
  - uses: actions/checkout@v4

Locations:

- `.github/workflows/automerge-dependabot.yml:8`
- `.github/workflows/ci.yml:16`
- `.github/workflows/ci.yml:17`
- `.github/workflows/ci.yml:24`
- `.github/workflows/ci.yml:27`
- `.github/workflows/ci.yml:36`
- `.github/workflows/ci.yml:39`
- `.github/workflows/ci.yml:43`
- `.github/workflows/ci.yml:155`
- `.github/workflows/ci.yml:158`
- `.github/workflows/ci.yml:163`
- `.github/workflows/slash-command-dispatch.yml:9`
- `.github/workflows/update-major-version.yml:19`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` block, and no job within them defines job-level permissions either. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/automerge-dependabot.yml:1`
- `.github/workflows/ci.yml:1`
- `.github/workflows/slash-command-dispatch.yml:1`
- `.github/workflows/update-major-version.yml:1`

### script-injection (severity: high)

update-major-version.yml directly interpolates user-controlled `workflow_dispatch` inputs into `run:` shell commands via ${{ }} expressions (sub-rule a). An attacker with permission to trigger the workflow can inject arbitrary shell commands.

Line 27: `run: git tag -f ${{ github.event.inputs.main_version }} ${{ github.event.inputs.target }}`
Line 28: `run: git push origin ${{ github.event.inputs.main_version }} --force`

Both `github.event.inputs.main_version` and `github.event.inputs.target` are workflow_dispatch inputs that flow directly into shell without quoting or sanitization. These should be moved to `env:` variables and referenced as quoted shell variables (e.g., `"$MAIN_VERSION"`) instead.

Locations:

- `.github/workflows/update-major-version.yml:27`
- `.github/workflows/update-major-version.yml:29`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all four workflow files: (1) Pinned all 7 action references to full commit SHAs using lookup_action_sha, preserving original tags as comments. (2) Added minimal permissions blocks to all workflows — automerge-dependabot.yml gets pull-requests:write; ci.yml gets contents:read at top level with the package job overriding to contents:write + pull-requests:write; slash-command-dispatch.yml gets contents:read; update-major-version.yml gets contents:write. (3) Fixed script injection in update-major-version.yml by moving github.event.inputs.main_version and github.event.inputs.target into step env: blocks and referencing them as quoted shell variables.

