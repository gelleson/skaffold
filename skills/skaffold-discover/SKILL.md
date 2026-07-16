---
name: skaffold-discover
description: Inspect a new or existing repository to identify every runnable app, frontend, worker, preparation job, third-party dependency, route, health check, watch loop, command, and environment-variable name. Use during /skaffold and /skaffold-update before wiring or changing the development environment.
---

# Skaffold Discover

Build a factual topology from the repository. Discovery is read-only except
when the parent workflow asks this skill to write `.skaffold/project.yaml`.

## Inspect safely

1. Read applicable instruction files first.
2. Inventory files with `rg --files`, then inspect package manifests,
   lockfiles, workspace definitions, language build files, Dockerfiles,
   Compose files, `Justfile`, process files, scripts, CI, and development docs.
3. Follow command references into code and configuration far enough to
   distinguish servers, frontends, workers, schedulers, one-shot preparation,
   and third-party services.
4. Search for environment variable names and access sites. Read schemas and
   encrypted-file metadata only; never open plaintext `.env*`, decrypted SOPS
   output, private keys, or provider responses.
5. Preserve an existing Compose, Portless, tunnel, SOPS, or Varlock integration
   when it can satisfy the parent contract.

For every executable unit, determine:

- `kind`, repository path, runtime, package manager, and lockfile;
- install/build/preparation commands and the development start command;
- internal listen address and port, inter-service dependencies, and working
  directory;
- existing Docker build and hot-reload behavior;
- health or readiness probe and expected startup ordering;
- whether browsers or external callbacks must reach it;
- local route label, public origin consumers, and HMR/WebSocket/SSE needs;
- declared environment names and which process consumes them, never values.

For dependencies, record the image or established host command, health check,
data volume, internal consumers, and whether host exposure is truly required.

## Handle repository state

If no application or stack-defining file exists, ask the user which stack they
already selected. Do not infer or recommend one unless asked. After the parent
creates it, rescan normally.

In an existing repository, do not ask the user questions that code and docs can
answer. State uncertainty in the inventory, test the likely command safely, and
refine the result from evidence.

## Maintain the manifest

Write the stable inventory to `.skaffold/project.yaml` using the main
`$skaffold` project contract. Record Skaffold-owned files and marked blocks in
`managed`, and record inspected user-owned files in `managed.inspect_only`.
Exclude values and ephemeral runtime identifiers. Treat the manifest as a
reproducible discovery snapshot, not permission to overwrite user content.
