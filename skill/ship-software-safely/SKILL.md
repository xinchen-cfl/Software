---
name: ship-software-safely
description: Apply a traceable software delivery workflow from local implementation through Git, pull requests, CI, immutable artifacts, staging, production, rollback, and cleanup. Use when Codex is asked to implement, merge, release, deploy, promote, rollback, or diagnose delivery state for a repository, especially with GitHub, containers, registries, worktrees, servers, or multiple environments.
---

# Ship Software Safely

Ship the smallest safe change. Adapt commands to the repository; do not assume a language, CI provider, registry, or runtime.

## Rules

- Inspect repository instructions, branch, worktrees, dirty files, remotes, CI, build, and deploy conventions before changing state.
- Preserve unrelated and unknown work.
- Do not infer authorization to merge, deploy, delete remote state, or change production.
- Build once and promote the same immutable artifact through environments.
- Use a full digest when available; do not deploy `latest`.
- Verify resulting state, not only command success.
- Re-run checks only after relevant state changes or at an independent boundary.
- Record the rollback target before replacing production.

## Workflow

### 1. Develop

Start from the intended base commit on a feature branch. Use a separate worktree only for concurrent or isolated work.

1. Add or identify a failing check for the intended behavior.
2. Implement the smallest coherent change.
3. Run focused checks, then broader repository-required checks.
4. Inspect status and diff; commit only the intended patch.

### 2. Integrate

Push and open a PR when the repository uses PRs. Let CI reproduce checks in a clean environment. Rebase when required; after published history is rewritten, use `--force-with-lease`.

After merge, fetch the target branch and identify its actual commit. Squash and rebase may preserve the patch while changing commit IDs.

### 3. Build

Build the deployable artifact from the merged target-branch commit. Record:

```text
source commit:
artifact reference:
artifact digest:
platform:
```

CI success is not deployment. A movable tag is not artifact identity.

### 4. Stage

Capture the current staging artifact, deploy the new digest, then verify:

- process/container health and readiness
- version signal
- configured artifact and actual artifact ID
- critical behavior changed by the release
- production remained unchanged

### 5. Produce

Before production, state the target digest, current artifact, rollback target, affected services, and verification commands. Deploy the exact staging digest and repeat the applicable checks.

Check colocated services when ports, proxy, networks, storage, or host resources are shared.

### 6. Roll back

Rollback by redeploying a known previous artifact, not by undoing Git or rebuilding old source. Verify version, identity, health, critical behavior, and unaffected environments. Treat database rollback separately.

### 7. Clean up

Fetch and prune, confirm the patch exists on the target branch, inspect every worktree for dirty or untracked files, remove obsolete worktrees, then delete obsolete branches. Never discard uncommitted, unpushed, unknown work.

## Completion evidence

Do not claim completion until applicable fields are known:

```text
source commit:
CI/check result:
artifact digest:
staging artifact / version / health:
production artifact / version / health:
rollback target and rollback result:
branch/worktree cleanup state:
```

Read [references/release-principles.md](references/release-principles.md) only when Git identity, test boundaries, artifact identity, or environment identity is unclear.

