# Release principles

## Identities

- worktree: filesystem checkout, possibly containing uncommitted state
- branch: movable pointer to a commit chain
- commit SHA: source history identity
- image tag: movable label
- image digest: immutable artifact identity
- container ID: one running instance

`fetch` updates remote knowledge; `pull` integrates it locally. Rebase and squash can change commit IDs while preserving a patch.

## Independent evidence

```text
focused test -> changed behavior
local fence -> intended patch and nearby behavior
CI -> clean reproducible environment
artifact check -> packaged software can run
staging -> production-like integration
production check -> real environment runs the intended artifact
```

Do not repeat an unchanged check without a new boundary or relevant state change.

## Promotion invariant

```text
merged commit -> build once -> immutable digest -> staging -> same digest in production
```

Rollback selects a previously built artifact. Server-side state should record current, previous, actual artifact ID, and the artifact withdrawn by rollback. Keep durable artifacts in the registry.

Directories alone do not isolate staging and production. Check processes, networks, ports, configuration, secrets, data, permissions, hosts, and traffic.
