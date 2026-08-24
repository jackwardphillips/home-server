# Home Server Control Room

This directory is the lightweight control room for a three-node home server. It
runs on `waxwing` in WSL. Application workloads run on the remote hosts and are
managed over SSH. The short aliases `mac`, `spectre`, and `surface` are for
interactive administration; unattended control-room commands use the matching
`mac-codex`, `spectre-codex`, and `surface-codex` aliases.

This document records observed reality as of 2026-08-20. See
[`inventory.yaml`](inventory.yaml) for hardware, network, storage, and software
details.

## Repository layout

This public control-room repository links the host-specific service-definition
repositories as Git submodules:

- `repos/hs-mac`
- `repos/hs-surface`
- `repos/hs-spectre`

Clone the complete definition set with:

```bash
git clone --recurse-submodules https://github.com/jackwardphillips/home-server.git
```

Published addresses are documentation-only examples. Real endpoint values,
credentials, runtime state, databases, generated artifacts, and the local
inventory overlay remain ignored on their owning machines.

## Current architecture

### `mac` (`jack-server`)

The Apple M3 MacBook remains a major application and data host. Its authoritative
operations repository appears to be `/Users/jack/Server`; it contains Compose
definitions, application repositories, service documentation, scripts, runtime
data, media, photos, Minecraft data, and launchd definitions.

Docker Desktop currently runs Immich, Homepage, Beszel, Portainer, and
a short-lived Twitcher ingestion container. The public Twitcher
application is hosted on Vercel; only its ingestion workload runs on the Mac.
Native services include Jellyfin, Netdata, and the Beszel agent. The former
Minecraft and BlueMap workloads are stopped and preserved for rollback. The
Observatory's former launchd jobs and
PostgreSQL container are stopped and preserved for rollback after its migration
to Surface.

Important locations:

- `/Users/jack/Server/compose` — Compose stacks
- `/Users/jack/Server/apps` — independent application repositories
- `/Users/jack/Server/services` — browsing index linking to stacks and apps
- `/Users/jack/Server/config/launchd` — host service/job definitions
- `/Users/jack/Server/docs` — detailed operations documentation
- `/Users/jack/Server/data`, `media`, `photos`, and `minecraft` — runtime data

Docker is version 29.6.2 and Compose is v5.3.1. A non-interactive SSH shell did
not include `/usr/local/bin` in `PATH`, although the Docker CLI symlink exists
there. The helper script accounts for this.

### `spectre`

This Ubuntu 26.04 LTS host runs Docker, containerd, Tailscale, the Beszel agent,
an independent Surface watchdog, and a power monitor. The watchdog checks
Surface's Uptime Kuma endpoint once per minute and sends a Discord alert after
three consecutive failures, plus one recovery notification. The power monitor
reports AC-to-battery and battery-to-AC transitions. Their definitions are at
`/home/jack/services/surface-watchdog/compose.yaml` and
`/home/jack/services/power-monitor/compose.yaml`.

### `surface`

This Ubuntu 26.04 LTS host runs Docker, containerd, Tailscale, the Beszel agent,
Uptime Kuma 2.3.2, Personal Observatory, Dawarich, Minecraft, BlueMap, the
Minecraft player dashboard, and a power monitor.
Observatory's API, scheduler, and PostGIS database run as one Compose project under
`/home/jack/services/the-observatory`; its API is restricted to Surface's
Tailscale address at `http://surface-tailnet.example:8000`. Dawarich runs under
`/home/jack/services/dawarich` and is restricted to the same Tailscale address
at `http://surface-tailnet.example:3002`. Kuma is the fleet's active
availability monitor and is available only over Tailscale at
`http://surface-tailnet.example:3001`. Its definition is at
`/home/jack/services/uptime-kuma/compose.yaml`, and its database is in a local
Docker volume rather than a network filesystem. The power monitor reports
AC-to-battery and battery-to-AC transitions from
`/home/jack/services/power-monitor/compose.yaml`. Minecraft is schedule-managed
by systemd, with its live data under `/home/jack/minecraft`; BlueMap and the
read-only player dashboard use that same Surface-hosted world. Their tailnet
URLs are `http://surface-tailnet.example:8100` and
`http://surface-tailnet.example:3003/minecraft/` respectively.

## Monitoring and alerting

The alerting design deliberately spans hosts:

- The Mac's launchd jobs continue to provide local restart, scheduled-job,
  update, power, and Discord reporting.
- Beszel on the Mac provides fleet metrics and resource alerts.
- Uptime Kuma on `surface` actively checks hosts and services.
- `surface-watchdog` on `spectre` independently checks Surface and Kuma, so a
  failure of the primary monitor can still generate a Discord alert.
- Native/containerized power monitors on all three laptops report when a host
  changes from AC to battery and when AC power returns. The Linux monitors read
  `/sys/class/power_supply` every 20 seconds and do not control charging.
- Off-site dead-man heartbeats remain pending. They require private, account-
  specific ping URLs and are needed to detect a whole-site power or internet
  outage independently of the home network.

Manage Kuma at `http://surface-tailnet.example:3001` while connected to Tailscale. It
checks the Mac and Spectre hosts plus the fleet's primary web services, with
Discord notifications enabled. Never store its credentials in this repository.

Fleet-wide operational history is maintained in [`CHANGELOG.md`](CHANGELOG.md)
on `waxwing`, rather than inside one particular remote host's repository.

## Access and multi-host work

Connect from `waxwing` with:

```bash
ssh mac
ssh spectre
ssh surface
```

Those short aliases use the owner's passphrase-protected key and permit an
interactive terminal. Read-only scripts and other unattended commands use the
restricted, no-PTY aliases instead:

```bash
ssh mac-codex command
ssh spectre-codex command
ssh surface-codex command
```

Codex should begin multi-host work here, inspect the relevant host before making
changes, and name every host it intends to touch. Do not infer placement from a
service name or historical role. Prefer the Compose file, repository, or native
service definition on the owning host over ad-hoc container state.

The scripts in [`scripts/`](scripts/) provide fleet-wide read-only snapshots:

```bash
scripts/status-all
scripts/docker-all
scripts/disk-all
```

Each continues if an individual host is unavailable.

## Browsing remote filesystems

The [`hosts/`](hosts/) directory provides convenient SSHFS views of each remote
home directory in VS Code Explorer:

```text
hosts/
├── mac/      # mac:/Users/jack/Server/services
├── spectre/  # spectre:/home/jack/Server
└── surface/  # surface:/home/jack/Server
```

The Mac mount opens its service index directly, so its Compose stacks and
applications appear immediately below `hosts/mac/`. The Linux mounts retain
their `services/` level because `/home/jack/Server` is their common browsing
root.

On the Linux hosts, `Server/services` is a symlink to the existing
`/home/jack/services` directory. The service definitions remain in their
original location; they have not been moved or duplicated. The mount helper
uses SSHFS's `follow_symlinks` option for these two hosts so VS Code presents
`services` as a browsable directory instead of a symlink-like file.

These are live remote files, not local copies or one shared filesystem. Editing,
moving, or deleting anything below `hosts/` immediately affects the corresponding
server. Use them primarily for browsing and inspection; SSH and VS Code
Remote-SSH remain the preferred ways to perform administration and development
work on each machine.

Mount all available hosts from `/home/jack/server` with:

```bash
scripts/mount-hosts
```

The script skips hosts that are already mounted and continues if a host is
unavailable. Safely unmount all mounted host views with:

```bash
scripts/unmount-hosts
```

These mounts are intentionally not configured to start automatically with WSL.
Run the mount script when browsing access is wanted and the unmount script when
it is no longer needed.

## Confirmed context and remaining inconsistencies

- Historical application services remain concentrated on `mac`; the Linux
  nodes now also carry the monitoring workloads described above.
- Minecraft, BlueMap, and the player dashboard run on `surface`. Minecraft is
  intentionally schedule-controlled; the stopped Mac copies and disabled Mac
  launchd jobs are retained solely for rollback.
- Personal Observatory runs on `surface`. Its stopped Mac launchd jobs,
  PostgreSQL container, volume, and final migration dump are retained solely as
  a rollback copy.
- Dawarich runs on `surface`. Its stopped Mac containers, named volumes, final
  database dump, and non-database volume archives are retained as a rollback
  copy.
- Twitcher is intentionally split: the application is hosted on Vercel and its
  ingestion workload runs on `mac`.
- `spectre` has a 238.5 GiB disk but only a 100 GiB root logical volume;
  `surface` has a 953.9 GiB disk but an 800 GiB root logical volume. Both systems
  were intentionally built with LVM.
- `spectre` was initially stood up about three days before inspection and brought
  back online that day; `surface` was first brought online on the inspection
  date. This explains their short host uptimes. Beszel on `spectre` nevertheless
  reported three weeks of container uptime; that status discrepancy remains
  unexplained.
