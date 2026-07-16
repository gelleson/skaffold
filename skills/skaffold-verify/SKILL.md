---
name: skaffold-verify
description: Exercise and repair a Skaffold-managed development environment end to end. Use after /skaffold or /skaffold-update to prove static configuration, secret-safe injection, background startup, hot reload, local and public routes, streaming logs, worktree isolation, non-destructive shutdown, and restart persistence.
---

# Skaffold Verify

Verification is a repair loop, not a checklist-only review. Run the applicable
checks, diagnose failures from labeled logs and state, patch the managed wiring,
and repeat until the contract passes or an external blocker is proven.

## Validate statically

- Validate `.skaffold/project.yaml`, task-runner syntax, scripts, Dockerfiles,
  and Compose configuration.
- Ensure every discovered executable and dependency is represented, every
  exposed service has a unique route, and every stateful dependency has an
  instance-scoped named volume.
- Ensure `.skaffold/runtime/` and plaintext override files are ignored.
- Search managed files for fixed project names, branch-global ports, committed
  tunnel URLs, plaintext secret values, and destructive volume or cache flags.
- Use `$manage-varlock` agent-safe checks for schema, SOPS decryption ability,
  and variable presence. Never display resolved values or unredacted Compose
  environment output.

## Exercise development

1. Run `just dev` and confirm it returns after backgrounding the complete stack.
2. Confirm preparation succeeds, dependencies and apps become healthy, workers
   remain running, and every printed local URL resolves to the intended service.
3. Start `just logs`, confirm labels include every service class, then exit the
   viewer and prove the stack is still running.
4. Make a small reversible source-only change in each distinct watch strategy
   and prove the target reloads or rebuilds. Restore the exact source content.
5. Call `just dev` again and prove it reconciles instead of duplicating
   resources.

## Exercise tunnels

Run `just tunnel`; confirm it ensures development is healthy, prints one public
URL per exposed service, and each URL responds correctly without authentication.
Check public-origin, callback, CORS/CSRF, proxy, cookie, and HMR behavior when
the framework uses them. Detect SSE and enforce the provider rule from
`$skaffold-routes`. Confirm tunnel logs appear in `just logs`.

## Exercise isolation and shutdown

When Git supports it, create or use a disposable second worktree or equivalent
instance without altering another repository. Run both instances together and
prove their Compose projects, ports, routes, cookies, runtime directories,
tunnels, and named volumes do not collide.

Record current named volumes and service identity, then run `just down`. Confirm
only the selected instance stopped, no container/network/volume/image/cache was
deleted, the shared Portless proxy and second instance still work, and logs can
be reattached after restart. Run `just dev` again and confirm state persists.

## Report evidence

Summarize checks by command and outcome, include labeled local and public URLs,
and name any remaining external blocker. Never include secret values, decrypted
payloads, private keys, or credential-bearing commands.
