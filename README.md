# Skaffold

Skaffold is a set of Agent Skills for making any new or existing repository run
as a complete development environment with one agent command. It investigates
the actual stack instead of imposing a language template, so the same workflow
can adapt to Go, Rust, Ruby, JavaScript, mixed frontends, workers, databases,
queues, Temporal, and other services.

## Install

Add this skill repository with your Skills client:

```sh
npx skills add <repository-url-or-path>
```

Install the Skaffold skill set and use `skaffold` as the primary entrypoint.
The dispatcher uses `skaffold-discover`, `skaffold-host`, `manage-varlock`,
`skaffold-compose`, `skaffold-routes`, and `skaffold-verify` to complete the
workflow.

## Use

In the target repository, invoke:

```text
/skaffold
```

For an existing repository, Skaffold inspects its instructions, manifests,
commands, services, environment contract, and routes, then wires and repairs the
environment until it works. For an empty repository, it asks which stack you
already chose before creating anything.

After adding or changing apps, workers, dependencies, routes, environment
variables, or watch behavior, invoke:

```text
/skaffold-update
```

## Result in a configured repository

| Command | Behavior |
| --- | --- |
| `just dev` | Starts the complete isolated stack in the background, waits for readiness, and prints every Portless local route. |
| `just tunnel` | Ensures development is running, starts every public route, waits for readiness, and prints every public URL. |
| `just logs` | Streams labeled app, worker, dependency, preparation, and tunnel logs without owning service lifetime. |
| `just down` | Stops only the current worktree instance and deletes nothing. |

Each branch or worktree receives isolated Compose resources, ports, named
volumes and data, cookies, runtime state, Portless routes, and tunnels. Docker
Compose is preferred for the whole stack when hot reload can be verified.

## Environment and access

- Varlock provides the readable environment schema and process injection.
- SOPS stores committed ciphertext using per-host age or SSH recipients. Private
  keys and plaintext environment files are never copied between macOS and VPS
  hosts.
- Portless provides local development routes.
- Cloudflare Quick Tunnels provide ephemeral public routes by default.

Public tunnel URLs intentionally have **no authentication**. Anyone with a URL
can access its service until `just down` stops that instance.

First-class hosts are macOS and Ubuntu, including Ubuntu VPS development.

## Skill layout

| Skill | Responsibility |
| --- | --- |
| `skaffold` | Primary dispatcher and end-to-end policy |
| `skaffold-update` | Reconcile a configured repository after topology changes |
| `skaffold-discover` | Inventory apps, commands, dependencies, routes, health, watch, and environment names |
| `skaffold-host` | Check and provision host tools |
| `manage-varlock` | Maintain Varlock, SOPS, recipients, and secret-safe injection |
| `skaffold-compose` | Build the isolated Compose runtime and lifecycle |
| `skaffold-routes` | Wire Portless and public tunnels |
| `skaffold-verify` | Exercise, diagnose, repair, and prove the contract |

The stable target-repository format is documented in
[`skills/skaffold/references/project-contract.md`](skills/skaffold/references/project-contract.md).
