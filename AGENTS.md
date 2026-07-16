# Skaffold

Skaffold is a repository of Agent Skills that turns a new or existing project
into a runnable, isolated development environment. The primary entrypoint is
`$skaffold`; `$skaffold-update` reconciles later topology changes.

## Repository scope

- Change only this repository unless the user explicitly selects another target.
- Treat adjacent repositories and examples as read-only evidence. Never modify
  `../apex` as part of Skaffold development.
- Preserve unrelated user changes and established target-repository tooling.
- Never read, print, diff, or persist plaintext secrets or private keys.
- Use `apply_patch` for file content changes.

## Skill architecture

`skills/skaffold` is the dispatcher. It coordinates these focused skills:

1. `skaffold-discover` inventories the real repository.
2. `skaffold-host` provisions macOS or Ubuntu requirements.
3. `manage-varlock` wires Varlock and SOPS with age or SSH recipients.
4. `skaffold-compose` builds the isolated background runtime and hot reload.
5. `skaffold-routes` wires Portless and public Cloudflare Quick Tunnels.
6. `skaffold-verify` runs the repair loop until the environment works.

`skills/skaffold-update` rescans an already configured target and applies only
the affected child workflows. `skills/scaffold-mobile-dev` is a compatibility
entry and must delegate to `$skaffold`.

## Target contract

Every configured target must expose:

- `just dev`: start the complete current instance in the background, wait for
  readiness, and print every local route.
- `just tunnel`: ensure development is running, start every unauthenticated
  public route, wait for readiness, and print every public URL.
- `just logs`: stream labeled logs; exiting the viewer leaves services running.
- `just down`: stop only the current instance and delete nothing.

Prefer Docker Compose for the entire stack when source watch works. Isolate
Compose projects, processes, networks, ports, named volumes, data, cookies,
routes, tunnels, and runtime state by worktree. Keep stable non-secret topology
in `.skaffold/project.yaml` and ephemeral state in ignored
`.skaffold/runtime/`.

For an existing target, inspect and repair autonomously until verification
passes. For an empty target, ask which stack the user already chose; do not
recommend one unless asked.

## Editing skills

- Keep each `SKILL.md` focused and put triggering conditions in its frontmatter
  description.
- Keep `agents/openai.yaml` synchronized and make its default prompt mention the
  skill as `$skill-name`.
- Put detailed shared contracts in `references/`; avoid duplicating them across
  child skills.
- Preserve the required command semantics and unauthenticated-public warning.
- Check current official behavior before encoding version-sensitive tool facts.

After changes, run `quick_validate.py` from the installed `skill-creator` skill
against every directory under `skills/`, parse every `agents/openai.yaml`, and
check that no template `TODO` remains.
