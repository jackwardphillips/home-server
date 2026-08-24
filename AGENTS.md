# Control-Room Guidance

These instructions apply to work launched from `/home/jack/server`.

This is the WSL control room for several machines that host long-running
services. Prioritize reliability over speed.

## General

- Don't assume. Don't hide confusion. Surface tradeoffs. If uncertain, ask.
- If multiple interpretations exist, present them. Don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## Principles

- Make the smallest change that solves the problem.
- Explain the plan before making significant changes.
- Ask before performing destructive actions.
- Match each host's existing project structure and tooling.
- Record fleet-wide operational changes in `/home/jack/server/CHANGELOG.md`.

## Scope and topology

- `waxwing` is the WSL control machine. Application workloads generally run on
  remote hosts, not in this directory.
- The short SSH aliases `mac`, `spectre`, and `surface` use the owner's
  passphrase-protected interactive key and permit PTYs. For unattended control-
  room commands, use `mac-codex`, `spectre-codex`, and `surface-codex`; those
  aliases use the restricted no-PTY automation key.
- Consult the ignored `inventory.local.yaml` when present, otherwise use the
  public example in `inventory.yaml`; always verify roles and live state before
  acting.
- If work affects more than one host, state exactly which hosts will be touched.
- Do not silently move services or data between hosts.

## Safety

- Inspect before modifying. Prefer read-only discovery first.
- Do not restart, recreate, stop, remove, or prune Docker resources without
  understanding their role and obtaining approval for disruptive work.
- Do not migrate services, change networking or Docker configuration, or install
  remote software without explicit approval.
- Before a risky or disruptive change, explain the command, affected host and
  service, expected impact, and rollback approach; obtain approval.
- Never print, copy here, or commit passwords, tokens, private keys, `.env`
  contents, webhook URLs, or other secrets.
- Treat Compose files, repositories, and native service definitions as more
  authoritative than ad-hoc container state when possible. Compare definitions
  with live state before acting.
- Review `git status` and `git diff` in the repository being changed before any
  commit. Do not commit automatically unless explicitly asked.

Never, without explicit approval:

- Delete data, volumes, databases, or runtime state.
- Reset or recreate containers if data may be lost.
- Modify `.env` files containing secrets.
- Open firewall ports or weaken security.
- Install system packages or remote software.

## SSHFS browsing views

- `/home/jack/server/hosts/*` contains live SSHFS mounts intended primarily for
  browsing and inspection in VS Code.
- Files beneath `hosts/` are live remote files. Editing or deleting them affects
  the corresponding server immediately.
- Prefer SSH commands against the appropriate host, or VS Code Remote-SSH, for
  remote administration and development work.
- Do not modify remote files through these mounts unless explicitly asked.
- Do not treat the three mounts as one shared filesystem.

## Docker

- Prefer `docker compose` over manual `docker run`.
- Preserve persistent volumes and bind-mounted runtime data.
- After approved changes, verify the affected service is healthy.
- Check definitions, live state, and logs when troubleshooting instead of
  guessing.

## Git

- Prefer `git pull --ff-only`.
- Never force-push or rewrite history unless requested.
- Keep commits focused on the requested change.
- Do not assume `/home/jack/server` itself is a Git repository; verify first.

## Discovered conventions

- The Mac's main operations repository is `/Users/jack/Server`.
- Mac Compose stacks normally live at `/Users/jack/Server/compose/<service>`.
- Independent Mac applications live under `/Users/jack/Server/apps` and may be
  separate Git repositories.
- `/Users/jack/Server/services` is a browsing index of symlinks to the Mac's
  existing Compose stacks and application repositories; the authoritative
  files remain in `compose/` and `apps/`. The `hosts/mac` SSHFS view mounts this
  service index directly.
- Mac launchd definitions live at `/Users/jack/Server/config/launchd`; service
  notes live at `/Users/jack/Server/docs`.
- Runtime data under `/Users/jack/Server/data`, `media`, `photos`, and
  `minecraft` must not be treated as disposable source files.
- Linux service definitions currently live under `/home/jack/services`.
- On `spectre` and `surface`, `/home/jack/Server/services` is a symlink to
  `/home/jack/services`; the SSHFS browsing view mounts `/home/jack/Server`.
- Non-interactive SSH on `mac` may omit `/usr/local/bin` from `PATH`. Resolve the
  Docker CLI explicitly when needed; do not mistake this for Docker being absent.
- Use the read-only fleet scripts in `scripts/` for initial status checks.
