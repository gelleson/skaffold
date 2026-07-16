---
name: skaffold-compose
description: Build or update the branch-isolated Docker Compose development runtime for a Skaffold project. Use when /skaffold or /skaffold-update must wire apps, frontends, workers, preparation jobs, databases, queues, Temporal, hot reload, readiness, logs, and non-destructive lifecycle commands.
---

# Skaffold Compose

Turn the discovered topology into a complete background development runtime.
Prefer one Compose model for applications and third-party dependencies when it
can preserve the framework's development watch behavior.

## Select a watch strategy

Choose independently for each executable service:

1. Preserve a proven development Dockerfile and framework watcher.
2. Otherwise bind-mount source and run the framework's native watch command.
3. Use Compose `develop.watch` when it improves performance or correctness:
   `sync` source, `sync+restart` configuration, and `rebuild` manifests or
   native dependency changes. Use `initial_sync` when required.
4. Put container-owned dependency directories, build caches, and generated
   artifacts in named volumes so host and container architectures do not mix.
5. Fall back to a host-native process only when a correct container reload loop
   is not practical. Keep it under the same central lifecycle and logs.

Test the selected watch behavior; a configuration that starts but ignores
source changes is not complete.

## Isolate every instance

Derive one stable instance ID from repository identity plus the absolute worktree path
and branch or detached commit. Use a readable slug plus a short hash and apply
it consistently to:

- Compose project, containers, networks, host processes, and runtime paths;
- named volumes for databases, queues, Temporal, caches, and dependencies;
- published ports and Portless aliases;
- cookie names/domains and any framework development state;
- tunnel processes and public-origin overrides.

Bind host-published ports to loopback. Let Docker assign ports and query the
result, or allocate them with collision checking; never assume a branch is the
only running instance. Use Compose service DNS and internal ports for
service-to-service traffic.

## Build the runtime

- Add health checks for required dependencies and apps. Express startup
  ordering with health-aware dependencies where possible.
- Model migrations, asset generation, or bootstrap work as idempotent one-shot
  jobs that finish before dependent apps become ready.
- Run development containers as an appropriate non-root user when the stack
  permits it, and keep Dockerfiles cache-efficient and multi-architecture.
- Keep stable configuration in committed files and ephemeral state in the
  current instance's `.skaffold/runtime/` directory.
- Wrap the central runner with
  `varlock run --inject vars -- <runner>`. Pass only declared variables to
  Compose processes. Never materialize a decrypted env file, include resolved
  values in `docker compose config` output, or mount a host private key into a
  container.

Use a small repository-native runner behind `just` when orchestration needs
port discovery, Portless alias reconciliation, tunnel URL capture, or mixed
native and Compose processes. Keep it portable across macOS and Ubuntu.

## Provide lifecycle commands

Implement these exact public recipes:

- `just dev`: reconcile the full current instance in the background, enable
  watch behavior, run preparation, wait for required health, register routes,
  and print every labeled local URL. It must be idempotent.
- `just tunnel`: ensure `just dev` is healthy, reconcile tunnels for every
  exposed route, wait for public readiness, and print every labeled public URL.
- `just logs`: stream consistently labeled app, worker, dependency,
  preparation, native-process, and tunnel output. Ctrl-C exits only the viewer.
- `just down`: stop apps, workers, dependencies, native processes, routes, and
  tunnels belonging to the current instance. Use Compose stop semantics so it
  deletes no container, network, named volume, data, image, or cache, and never
  affects another instance.

Optional `just doctor` and `just urls` recipes are useful but cannot replace
the four required recipes.

## Check the result

Parse Compose configuration without printing secrets, build changed images,
start the stack, wait for readiness, inspect resolved host ports, and verify a
safe source edit triggers reload. Then hand off to `$skaffold-routes` and
`$skaffold-verify`.
