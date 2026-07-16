---
name: skaffold-update
description: Reconcile an already Skaffold-configured repository after apps, workers, dependencies, routes, environment variables, commands, or watch behavior change. Use for explicit /skaffold-update requests to rescan the real repo, update only affected managed wiring, preserve user content and data, and re-verify the isolated development environment.
---

# Skaffold Update

Update a configured target from current repository evidence, not from the old
manifest alone. If `.skaffold/project.yaml` is absent or structurally invalid,
use `$skaffold` for a full setup instead.

## Reconcile

1. Read the main `$skaffold` project contract and the target's instructions.
2. Use `$skaffold-discover` to rescan manifests, lockfiles, executable apps,
   workers, preparation commands, dependencies, routes, variable names, health
   checks, and watch behavior.
3. Compare discovery with `.skaffold/project.yaml`. Classify additions,
   removals, renames, and changed wiring without reading secret value files.
4. Apply only the affected skills:
   - `$skaffold-host` when tool requirements changed.
   - `$manage-varlock` when declared environment names or providers changed.
   - `$skaffold-compose` when processes, dependencies, health, or watch changed.
   - `$skaffold-routes` when exposed services or origin settings changed.
5. Update the manifest after the wiring is complete. Keep stable user-owned
   additions and refresh only Skaffold-managed files or marked blocks.
6. Use `$skaffold-verify` to run the complete behavior checks, repair failures,
   and repeat until they pass.

## Preserve state

- Never modify a nearby reference repository or another worktree.
- Do not recreate healthy services merely to normalize style.
- Preserve named volumes, database and queue data, images, caches, ciphertext,
  route identity where still valid, and unrelated runtime instances.
- Remove obsolete runtime resources only through the current instance's normal
  non-destructive lifecycle. Do not delete persistent data.
- Keep `just dev`, `just tunnel`, `just logs`, and `just down` semantics
  unchanged even when the internal topology changes.

Report the detected topology delta, managed files changed, commands verified,
and routes. Do not expose environment values.
