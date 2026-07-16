# Project contract

Commit `.skaffold/project.yaml` as the stable, non-secret description of what
Skaffold discovered and manages. Use this shape as a guide, extending it only
when the repository needs more detail:

```yaml
version: 1
project:
  name: example
  runtime: compose
services:
  api:
    kind: app
    path: services/api
    start: bundle exec rails server -b 0.0.0.0 -p 3000
    internal_port: 3000
    depends_on: [db]
    health: /up
    expose: true
    route: api
    watch:
      mode: native
      paths: [services/api]
  db:
    kind: dependency
    image: postgres:17
    expose: false
environment:
  schema: .env.schema
  encrypted: secrets/development.sops.json
  provider: sops-age
managed:
  files: []
  blocks: []
  inspect_only: []
```

Record apps, frontends, workers, one-shot preparation jobs, dependencies,
commands, health checks, declared variable names, route labels, and watch
strategy. Never record resolved values, host-assigned ports, PIDs, container
IDs, tunnel URLs, or other ephemeral state.

Add `.skaffold/runtime/` to the repository ignore file. Runtime data belongs
there in an instance-specific subdirectory. It must be safe to recreate and
must never contain decrypted environment files.

## Required commands

`just dev` starts or reconciles the complete current instance in the
background, waits until required services are ready, registers local routes,
and prints every labeled local URL. Repeated calls are idempotent.

`just tunnel` first ensures `just dev` is healthy, then starts one public,
unauthenticated route for every browser-facing service. It waits until the
routes respond and prints every labeled public URL. Repeated calls reconcile
missing tunnels rather than duplicating healthy ones.

`just logs` attaches a labeled, follow-mode view of app, worker, dependency,
preparation, and tunnel logs. Exiting the viewer leaves all services running.

`just down` stops only the current instance's apps, workers, dependencies,
routes, and tunnels. Prefer Compose stop semantics: do not delete containers,
networks, named volumes, images, build caches, secret ciphertext, or runtime
data needed to restart. It must not stop the shared Portless proxy or another
worktree's resources.

## Managed edits

Prefer whole generated files when the repo does not own them. In shared files,
use stable markers:

```text
# skaffold:start <purpose>
...
# skaffold:end <purpose>
```

List generated files and blocks in `managed.files` and `managed.blocks`. List
user-owned files that Skaffold only inspected in `managed.inspect_only`.
`/skaffold-update` may replace managed content but must merge around user-owned
content.
