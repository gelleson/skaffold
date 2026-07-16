---
name: skaffold
description: Turn a new or existing repository into a one-command, branch-isolated development environment. Use for explicit /skaffold requests and when a repo must run on macOS or Ubuntu with Docker Compose hot reload, Portless local routes, optional unauthenticated Cloudflare Quick Tunnels, SOPS-encrypted configuration, and the required just dev, just tunnel, just logs, and just down commands.
---

# Skaffold

Inspect the actual repository, wire its complete development topology, and keep
repairing it until the documented commands work. Do not assume a fixed language
or framework: support Go, Rust, Ruby, JavaScript, mixed frontends, workers, and
other executable stacks discovered in the repository.

Read [project-contract.md](references/project-contract.md) and
[tool-notes.md](references/tool-notes.md) before changing a target repository.

## Protect the target

- Follow every applicable repository instruction file and preserve unrelated
  changes.
- Treat paths outside the selected target repository as read-only unless the
  user explicitly puts them in scope. A nearby example repository is evidence,
  not an edit target.
- Never read, print, diff, or persist plaintext secret values. Delegate the
  environment contract to `$manage-varlock`.
- Commit stable topology in `.skaffold/project.yaml`; keep PIDs, assigned ports,
  logs, and tunnel URLs only under ignored `.skaffold/runtime/`.
- Prefer managed files or marked blocks. Do not replace established tooling
  when it already satisfies the contract.

## Dispatch the workflow

Apply the child skills in this order:

1. Use `$skaffold-discover` to inspect the repo and build a factual service,
   dependency, route, command, health, watch, and environment inventory.
2. If the repository is empty, ask which already-chosen stack to create. Do not
   recommend a stack unless asked. Create the minimum runnable app, then run
   discovery again.
3. Use `$skaffold-host` to check or provision the current macOS or Ubuntu host.
4. Use `$manage-varlock` to establish a readable schema and SOPS-backed,
   host-specific age or SSH decryption without exposing values.
5. Use `$skaffold-compose` to wire the full stack, hot reload, readiness,
   lifecycle commands, and branch/worktree isolation.
6. Use `$skaffold-routes` to give every browser-facing service a Portless route
   and an ephemeral public route.
7. Use `$skaffold-verify` to exercise the commands, isolation, reload, routes,
   logs, tunnel access, shutdown, and persistence. Repair failures and repeat.

Do not stop after writing a plan or configuration. Execute safe checks and
iterate until the environment works. Ask only when the repo is empty and the
stack is unknown, a secret must be entered out of band, privilege approval is
required, or an external system blocks further progress.

## Enforce the result

- Prefer Docker Compose for applications, workers, and third-party services
  when source changes reload correctly. Use a host-native process only for a
  service that cannot provide a sound container watch loop.
- Isolate Compose project names, containers, networks, ports, volumes, database
  state, queue state, Temporal state, cookies, runtime files, routes, and
  tunnels for each worktree or branch instance.
- Make `just dev` background the complete stack, wait for readiness, and print
  every local route.
- Make `just tunnel` ensure development is running, start unauthenticated public
  routes for every exposed service, wait for them, and print every public URL.
- Make `just logs` stream labeled logs without stopping the stack when the
  viewer exits.
- Make `just down` stop only the current instance and delete no persistent
  data, named volume, image, or cache.
- State clearly that public tunnel URLs have no authentication. Do not silently
  add authentication the user did not request.

Finish with the commands that were verified, all local and public route labels,
the files changed, and any unavoidable limitation. Never include secret values.
