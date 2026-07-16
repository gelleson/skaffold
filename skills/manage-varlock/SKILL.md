---
name: manage-varlock
description: Set up, migrate, validate, and maintain secret-safe project environment configuration with Varlock and committed SOPS ciphertext. Use for .env.schema, environment variable typing, Docker Compose injection, API keys or credentials, SOPS age or SSH recipients, macOS and Ubuntu host identities, CI leak checks, and Skaffold development runners.
---

# Manage Varlock

Treat `.env.schema` as the readable, committed contract. Keep values in an
ignored host override, secure provider, or committed SOPS ciphertext and inject
resolved configuration only into the process that needs it.

## Protect secrets

- Read and edit `.env.schema`, `.sops.yaml`, and intentionally public recipient
  metadata only.
- Do not read, print, diff, or edit plaintext `.env`, `.env.local`, decrypted
  SOPS output, private keys, or provider responses.
- Do not use `echo`, `printenv`, shell tracing, `varlock printenv`, raw JSON/env
  output, or commands that place resolved values in logs or chat.
- Use `varlock load --agent` for redacted inspection and validation.
- Never weaken `@sensitive`, requiredness, or validation to make a check pass.
- Ask the user to enter secrets and manage private key material in their own
  trusted session. Never ask them to paste a value into chat.

If plaintext is discovered, stop displaying it, report only the file and
variable name, and recommend rotation. Do not repeat the value.

## Inspect the project

1. Read repository instructions and preserve unrelated changes.
2. Identify the package manager, installed Varlock form, schema, tracked and
   ignored environment files, `.sops.yaml`, encrypted sources, task runner,
   Compose wiring, and CI.
3. Search variable names and access patterns with `rg`; do not open ignored
   value files or encrypted payloads that may contain intentionally unencrypted
   application fields.
4. Invoke a local Varlock dependency through its package manager. Use a
   standalone binary only when that is established project practice.
5. Check `varlock <command> --help` and `sops <command> --help` before relying on
   unfamiliar or version-sensitive flags.

For a monorepo, prefer a root schema for shared values and explicit imported
schemas at package boundaries.

## Set up or migrate

When setup is in scope:

1. Run `varlock init --agent` from the intended schema root.
2. Treat generated declarations as a draft and review each item against code.
3. Replace example-file semantics with complete schema declarations without
   copying real values from plaintext files.
4. Track `.env.schema` and SOPS ciphertext; keep local overrides, private keys,
   and `.skaffold/runtime/` ignored.
5. Add an official framework integration when one exists. Otherwise wrap the
   central runner with Varlock.
6. Review manifest, lockfile, generated types, Compose/task files, and ignore
   changes before keeping them.

Do not install packages, plugins, or host tools unless setup or installation is
part of the parent request.

## Model each value

- Public value with a safe default: store the default in `.env.schema`.
- Sensitive value: declare it empty with `@sensitive`, correct type and required
  rules, then resolve it from SOPS, a provider, or an ignored override.
- Machine-local secret: let the user create it interactively, then validate only
  with agent-safe output.
- Shared development or production secret: use the repository's encrypted
  source or a pinned provider plugin.
- Branch-derived ports, URLs, cookies, and tunnel origins: compute them in the
  central runner and pass process overrides; never commit them.

Example schema:

```dotenv
# @defaultSensitive=false @defaultRequired=false
# @currentEnv=$APP_ENV
# ---

# @required @type=enum(development,test,production)
APP_ENV=development

# @type=url
API_URL=http://127.0.0.1:8000

# @sensitive @required
API_KEY=
```

Keep a blank line between item blocks so decorators attach to the intended
variable. Omit `@type=string` unless string constraints are useful.

## Integrate SOPS identities

Preserve an existing SOPS format. For a new setup, use one encrypted development
file and public recipients in `.sops.yaml`; JSON is convenient for bulk schema
resolution. SOPS may use age recipients or supported `ssh-ed25519`/`ssh-rsa`
public recipients.

Each macOS workstation or Ubuntu VPS owns its private identity. Never copy a
private age or SSH key, decrypted file, or plaintext override between hosts.
When a host cannot decrypt, report the host's public recipient and ask the user
to add it out of band, then update recipients with SOPS without exposing values.

Keep decryption in memory through a Varlock resolver, for example:

```dotenv
# @setValuesBulk(exec("sops decrypt --output-type json secrets/development.sops.json"), format=json)
```

Have the user edit encrypted values through SOPS in their own interactive
session. Commit only ciphertext, schema, and public recipients.

## Integrate the Skaffold runner

Put Varlock outside Compose so secrets enter only declared child processes:

```bash
varlock run --inject vars -- scripts/skaffold dev
```

Use the same wrapper for `tunnel`, `logs` only when it needs declared values,
and other environment-consuming operations. Do not write a decrypted env file,
pass secrets as command arguments, print unredacted `docker compose config`, or
mount the host's private identity into a container.

## Change existing configuration

1. Find every producer and consumer of the variable.
2. Update the schema together with code, Compose, CI, and documentation.
3. Preserve sensitivity, requiredness, types, comments, and environment rules.
4. For rename or deletion, remove every obsolete consumer only after the new
   path works.
5. Run `varlock audit`; use `@auditIgnore` only for genuine external consumers.

## Verify safely

Use the narrowest applicable checks:

```bash
varlock load --agent
varlock audit
varlock scan --staged
varlock run --inject vars -- sh -c 'test -n "${VARIABLE_NAME:-}"'
sops decrypt secrets/development.sops.json >/dev/null
```

Use the established package-manager prefix when needed. Verify only presence
and successful decryption, then run the affected application's smoke test.
Report schema names, recipient status, integrations, and check outcomes only.
