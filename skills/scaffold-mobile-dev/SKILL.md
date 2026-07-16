---
name: scaffold-mobile-dev
description: Compatibility entry for repositories or users that explicitly invoke the older scaffold-mobile-dev skill. Use only to migrate that request to the current Skaffold workflow with Portless local routes, public development tunnels, branch isolation, encrypted configuration, and just dev, just tunnel, just logs, and just down.
---

# Scaffold Mobile Dev Compatibility

This entry is retained for callers that installed or named the earlier skill.
Do not run the archived `scripts/scaffold-mobile-dev.py`; its Tailscale-specific
contract is superseded.

Apply `$skaffold` to the target repository and follow its complete discovery,
host, environment, Compose, routing, and verification workflow. Preserve any
existing Tailscale integration only if repository discovery shows it is still
independently required; it is not the default public tunnel provider.

Expose only the current required interface:

- `just dev` for background development and all local routes;
- `just tunnel` for all unauthenticated public development routes;
- `just logs` for streaming labeled logs without stopping services;
- `just down` for stop-only, non-destructive current-instance shutdown.

Tell the user that the compatibility entry delegated to `$skaffold` and report
the new verified commands and routes without secret values.
