---
name: skaffold-host
description: Check and provision a development host for a Skaffold-managed repository. Use during /skaffold or when /skaffold-update changes host requirements, with first-class support for macOS and Ubuntu and tools such as Docker Compose, just, Portless, cloudflared, SOPS, age, and Varlock.
---

# Skaffold Host

Make the current macOS or Ubuntu machine capable of running the discovered
project. Other operating systems are best effort.

## Check before installing

1. Detect operating system, architecture, shell, and whether the session is a
   local workstation or remote VPS.
2. Derive required tools from discovery. Normally check Docker with Compose,
   `just`, Portless, `cloudflared`, SOPS, age or SSH support, and Varlock.
3. Verify versions and actual command behavior. Require Docker Compose 2.22 or
   later when the selected stack uses Compose Watch.
4. Reuse compatible installed tools. Do not replace a working package source
   solely for consistency.

## Provision safely

- On macOS, prefer Homebrew and Docker Desktop or the repo's established Docker
  provider.
- On Ubuntu, prefer official distribution packages or the tool vendor's
  official repository when the distribution version is insufficient.
- Install automatically when setup is in scope, requesting approval when
  package installation or privilege escalation requires it.
- Do not open firewall ports, create public DNS, authenticate external
  accounts, modify SSH access, or weaken host security as a side effect.
- Do not copy a private SOPS, age, or SSH key between hosts. Check that this host
  has its own usable recipient identity and tell the user only what public
  recipient must be added out of band.

Portless is a shared host service. Start or repair its proxy when required, but
do not make a project-level `down` command own or stop the shared proxy.

## Verify

Confirm each required command is discoverable in a normal non-interactive shell
and can perform its safe version or status check. Confirm Docker can reach its
daemon and Compose can parse the project later. Report installed versions and
manual blockers without printing credentials or environment values.
