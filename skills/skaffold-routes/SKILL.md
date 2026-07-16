---
name: skaffold-routes
description: Wire stable Portless local URLs and ephemeral public tunnel URLs for every browser-facing service in a Skaffold project. Use during /skaffold or /skaffold-update to reconcile branch-isolated routes, Cloudflare Quick Tunnels, framework proxy/origin settings, and route logging.
---

# Skaffold Routes

Give each exposed service a readable local route and, on `just tunnel`, one
public unauthenticated route. Keep route ownership scoped to the current
worktree instance.

## Wire Portless locally

- Preserve an existing compatible Portless setup.
- When Portless launches a native process, use
  `portless run --name <instance-service> -- <command>`.
- For Docker or another independently managed listener, discover the actual
  assigned host port and reconcile
  `portless alias <instance-service> <port>`.
- Use explicit, collision-proof instance names even if Portless can infer
  worktree names for processes it launches.
- Persist alias ownership under `.skaffold/runtime/` so `just dev` is
  idempotent and `just down` touches only the current aliases.
- Detect and print the actual Portless scheme, domain, and non-default proxy
  port. Do not hard-code an example URL.

Do not stop the shared Portless proxy from a project lifecycle command.

## Wire public tunnels

Default to one Cloudflare Quick Tunnel per browser-facing service using the
service's loopback Portless or published origin. Prefer to supervise tunnels in
the central runner or a Compose profile so their logs, PIDs, and shutdown share
the normal lifecycle.

For every tunnel:

1. Start `cloudflared` without account credentials or access authentication.
2. Capture the assigned `trycloudflare.com` URL from structured output or logs
   without mixing unrelated tunnel output.
3. Store URL and ownership only in the current `.skaffold/runtime/` directory.
4. Wait for the public URL to return the service's expected health or browser
   response.
5. Print a labeled mapping from service to public URL.

Quick Tunnels do not support Server-Sent Events. If an exposed service needs
SSE, retain a suitable existing provider; otherwise report that exact blocker
instead of claiming the route works.

## Reconcile public origins

After tunnel URLs are known, update only development-time consumers of public
origins, callback URLs, CORS/CSRF trusted origins, proxy headers, secure cookie
behavior, HMR/WebSocket origins, and frontend API bases. Pass ephemeral values
through the runner environment and recreate only affected services. Do not
write tunnel URLs into committed files or ciphertext.

Apply framework-specific trusted-host and forwarded-header settings narrowly
to the development environment. Do not disable host validation globally.

## Make exposure explicit

The requested public URLs intentionally have no authentication. Print one clear
warning when `just tunnel` starts and never present them as private. Do not add
authentication silently. `just down` must stop current tunnel processes and
disable current aliases without deleting persistent project or Compose state.
