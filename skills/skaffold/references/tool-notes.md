# Tool notes

Use established repo versions when compatible. Check installed command help
before relying on flags, and consult current official documentation when a
version-sensitive detail matters.

## Docker Compose

- Compose Watch requires Docker Compose 2.22 or later.
- Watch actions include `sync`, `rebuild`, and `sync+restart`; `initial_sync`
  can populate a container before watching.
- `docker compose up -d --wait` backgrounds the stack and waits for health or
  running state. Use the repository-compatible watch invocation afterward.
- `docker compose down` preserves named volumes unless volume deletion is
  explicitly requested.
- Docs: <https://docs.docker.com/compose/how-tos/file-watch/> and
  <https://docs.docker.com/reference/cli/docker/compose/up/>.

## Portless

- Current upstream is Vercel Labs Portless: <https://github.com/vercel-labs/portless>.
- Use `portless run --name <name> -- <command>` when Portless owns a native
  process. Use `portless alias <name> <port>` for Docker or an independently
  managed process.
- Portless can derive worktree prefixes for processes it launches. Docker
  aliases still need an explicit collision-proof instance name.
- Keep the shared proxy alive when stopping a single project instance.

## Cloudflare Quick Tunnels

- `cloudflared tunnel --url http://localhost:<port>` creates a random
  `trycloudflare.com` URL without an account.
- Quick Tunnels are for development, have a 200-concurrent-request limit, and
  do not support Server-Sent Events. Detect incompatible exposed services and
  preserve a suitable existing provider or report the limitation.
- Docs: <https://developers.cloudflare.com/tunnel/setup/>.

## SOPS and Varlock

- SOPS supports age recipients and SSH `ssh-ed25519` or `ssh-rsa` recipients.
  Each host should keep its own private key and be added through a public
  recipient; update ciphertext recipients without copying keys.
- Prefer in-memory `sops exec-env`, `sops exec-file`, or an equivalent Varlock
  resolver. Never generate a persistent decrypted env file.
- Use `varlock load --agent` for redacted validation and
  `varlock run --inject vars -- <runner>` to pass declared variables to the
  central runner.
- Docs: <https://getsops.io/docs/> and
  <https://varlock.dev/reference/cli-commands/>.
