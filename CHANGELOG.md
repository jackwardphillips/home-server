# Changelog

## 2026-08-25

### Monitoring

- `mac`: Tuned the native Netdata service by disabling the unsupported
  privileged `powermetrics` collector and excluding the `Seagate Backup Plus
  Drive` and `Lab-backup2` mount paths from macOS disk/inode collection. This
  removed two false critical inode alarms and recurring collector warnings;
  the localhost-only listener and tailnet-only Tailscale Serve route remain
  unchanged. A pre-change configuration backup is retained beside
  `netdata.conf` as `netdata.conf.pre-fleet-20260825`. Removed the existing
  Netdata Cloud claim credentials and restarted the agent; it now reports
  unclaimed and offline while its local API, history, and alarms remain healthy.
- `spectre` and `surface`: Installed Netdata 2.11.0 as enabled native systemd
  services for on-demand diagnostics through the control-room SSH connection.
  Each API is bound only to `127.0.0.1:19999`; cross-host streaming and new
  Tailscale Serve routes are disabled. Each host retains up to seven days of
  local history in a single dbengine tier capped at 256 MiB. Existing Docker,
  Tailscale, watchdog, game, database, and application services were not
  restarted or reconfigured.

## 2026-08-23

### Fleet management

- Split Waxwing SSH access into passphrase-protected interactive aliases
  (`mac`, `spectre`, and `surface`) with PTY support and restricted unattended
  aliases (`mac-codex`, `spectre-codex`, and `surface-codex`). Updated the
  read-only fleet scripts and control-room guidance to use the unattended
  aliases without weakening their server-side restrictions.
- Added enabled weekly reboot timers on Spectre and Surface, staggered after
  the Mac's Saturday 3:00 AM maintenance window. Spectre reboots at 3:15 AM and
  Surface at 3:30 AM in `America/New_York`; missed timers do not trigger a late
  reboot on the next startup.

### Services

- `surface`: Added an on-demand, private-network Photon 1.2.1 reverse-geocoding
  service for Dawarich using the checksum-verified USA OpenStreetMap database.
  Photon is capped at 3 GiB, normally stopped, and started daily at 1:10 AM by
  a guarded catch-up runner that verifies readiness and progress before stopping
  it. Recreated only the Dawarich app and Sidekiq containers to load the local
  endpoint, then reverse-geocoded all 21,601 existing points without external
  coordinate requests. Tuned city detection to a 40-minute minimum with a
  30-minute maximum gap; nine cities currently satisfy that threshold.
- `surface`: Evaluated and then retired a tailnet-only local personal-assistant
  prototype using Ollama and Qwen 3 4B. The model handled simple prompts and
  narrow tools but was too slow and inconsistent for the desired cross-metric
  reasoning on the available CPU/6 GiB GPU-class hardware. Removed the assistant
  containers, downloaded model volume, images, and service tree after preserving
  the findings in `docs/personal-assistant-feasibility.md`; Observatory, Beszel,
  and all unrelated services and data were left unchanged.

## 2026-08-22

### Storage and media

- `mac`: Temporarily stopped Immich and cleanly unmounted the external `Media`
  SSD for a media-import workflow. Copied 47 distinct movies (95 GB) from the
  read-only `Lab-backup2` source through internal Mac staging into
  `/Volumes/Media/archive/Lab-backup2-20260822/movies`, then added clean
  per-title symlink entries under `/Volumes/Media/libraries/movies`. The source
  SSD and internal staging copy were retained. Reconnected `Media`, restarted
  Docker Desktop, and verified Immich and the other Mac containers healthy.

## 2026-08-21

### Storage and media

- Reformatted the Mac-attached 2 TB Seagate external SSD as GPT/APFS with the
  volume name `Media`, after the user confirmed the personal media had been
  backed up. Restored the selected entertainment archive from checksum-verified
  Surface staging and started a final source-to-destination checksum pass; the
  Surface copy remains in place pending explicit approval to remove it.
- Added a non-destructive Jellyfin movie view at
  `/Volumes/Media/libraries/movies`. It presents 17 consistently named movies
  through symlinks to the preserved files under `/Volumes/Media/archive`,
  omitting duplicate and bonus-title files without modifying the archive.

### Fleet management

- Changed Surface and Spectre from `Etc/UTC` to the
  `America/New_York` system timezone so host clocks, logs, and calendar timers
  consistently follow Eastern time, including daylight-saving transitions.
  Removed Surface's now-redundant Minecraft scheduler timezone override; its
  weekday 5:00 PM, weekend 7:30 AM, midnight shutdown, and 12:10 AM BlueMap
  boundaries are now interpreted directly in the host timezone.
- Added a dedicated unattended SSH identity on Waxwing for control-room access
  to `mac`, `spectre`, and `surface`. Each host authorization is restricted to
  its observed Waxwing source address and disables SSH agent, port, and X11
  forwarding plus PTY allocation. Existing user SSH keys were preserved.

### Services

- `mac`: Migrated the Immich photo library from the internal
  `/Users/jack/Server/photos` path to the external APFS `Media` volume at
  `/Volumes/Media/libraries/immich`. PostgreSQL remains on the internal SSD.
  Added a Compose bind-mount guard so Immich will not create a fallback library
  directory when the external volume is unavailable. Retained the original
  internal photo directory as a rollback copy.
- Added NASA DONKI event ingestion and a bandwidth-bounded NOAA GOES-19 GLM
  lightning sampler to Personal Observatory on Surface. DONKI polls hourly;
  GLM samples one 20-second frame every 15 minutes, retains flashes within
  100 km, rejects source objects above 2 MB, and does not retain NetCDF files.
  Applied additive database migration 0010 and rebuilt the API and scheduler;
  PostgreSQL and its persistent volume remained online and untouched.

## 2026-08-20

### Fleet management

- Established `/home/jack/server` on the Waxwing WSL control machine as the
  three-host control room for `mac`, `spectre`, and `surface`. Added a hardware,
  storage, network, role, and workload inventory plus fleet-wide read-only
  status, Docker, and disk inspection scripts.
- Added on-demand SSHFS browsing under `hosts/`, with resilient mount and
  unmount helpers that skip mounted or unavailable hosts, clean up stale FUSE
  endpoints, and avoid automatic startup mounting. The mounts are documented as
  live remote views intended primarily for inspection; SSH and VS Code
  Remote-SSH remain the preferred administration paths.
- Standardized Linux service browsing through `/home/jack/Server/services`,
  which links to the existing `/home/jack/services` directory on Spectre and
  Surface without relocating service definitions.
- Added `/Users/jack/Server/services` on the Mac as a non-disruptive symlink
  index of its existing Compose stacks and application repositories. The Mac
  SSHFS view now opens this index directly, while following remote symlinks so
  VS Code presents service entries as directories.
- Moved the fleet-wide changelog from the Mac operations repository to
  `/home/jack/server/CHANGELOG.md` on Waxwing and updated operational references.
  Service-specific changelogs remain with their respective services.
- Initialized host-specific Git repositories at `/home/jack/services` on
  Spectre and Surface. They track Compose definitions, service code, scripts,
  documentation, examples, and Surface's installed custom systemd units while
  excluding credentials, databases, generated content, and mutable state.
- Removed live Beszel agent tokens from both hosts' Compose definitions. Each
  definition now requires `BESZEL_TOKEN` from a mode-600, Git-ignored `.env`
  file, with a credential-free example and reconstruction guidance committed.

### Monitoring and alerting

- Deployed Uptime Kuma 2.3.2 on Surface with a local persistent Docker volume,
  automatic container recovery, and a listener restricted to Surface's
  Tailscale address. Completed the SQLite administrator setup and added
  authenticated checks for the Mac and Spectre hosts plus Homepage, Beszel,
  Portainer, Netdata, Immich, Dawarich, Personal Observatory, and BlueMap.
- Added and tested a default Discord notification provider in Uptime Kuma and
  assigned it to all fleet monitors. Checks run every 60 seconds and retry twice
  at 20-second intervals before reporting an outage.
- Added an independent watchdog on Spectre that checks Surface's Uptime Kuma
  endpoint every minute. It sends one Discord alert after three consecutive
  failures and one recovery notification when connectivity returns, allowing a
  failure of the primary monitoring host to be reported from another machine.
- Added containerized AC-power monitors on Spectre and Surface, matching the
  Mac's existing behavior: silent startup on AC, one alert when switching to
  battery, and one recovery when AC returns. The monitors read Linux power state
  every 20 seconds and do not control charging.
- Preserved the Mac's existing Beszel hub, host-local restart policies, startup
  health checks, scheduled-job alerts, power monitoring, and update automation.
  Off-site dead-man heartbeats remain pending so a complete home power or
  internet outage can eventually be detected outside the home network.

### Services

- Migrated Briancraft II Minecraft, BlueMap, and the read-only player dashboard
  from Mac to Surface. Copied the complete Minecraft and BlueMap data trees,
  successfully loaded and saved the world in an isolated rehearsal, then
  started the production server with a 6 GiB maximum heap. Surface now serves
  Minecraft on port 25565, BlueMap on its tailnet address at port 8100, and the
  player dashboard at port 3003.
- Replaced the Mac launchd automation with Surface systemd scheduling, a Discord
  join watcher, and nightly BlueMap rendering. Homepage, Uptime Kuma, and the
  Mac scheduled-health integration now query the Surface endpoints. The stopped
  Mac containers, source data, Compose definitions, and disabled launchd jobs
  remain intact as the rollback path; no source data was deleted.
- Made the migrated Minecraft scheduler evaluate its player-facing hours in
  `America/New_York` explicitly, independent of Surface's UTC system timezone.

- Published Dawarich through tailnet-only Tailscale Serve HTTPS at the
  configured Surface tailnet hostname. Serve proxies to the existing
  Tailscale-bound listener on port 3002; Funnel remains disabled, and no
  Dawarich containers were restarted.
- Migrated Dawarich's web application, Sidekiq worker, PostgreSQL database,
  Redis state, and persistent application volumes from Mac to Surface. The web
  listener is restricted to Surface's Tailscale address; Homepage and Uptime
  Kuma now use the Surface endpoint.
- Preserved the stopped Mac Dawarich containers and named volumes plus a final
  PostgreSQL dump and archives of all non-database volumes for rollback. The
  Surface environment file is restricted to its owner, and no source data was
  deleted.
- Migrated Personal Observatory from Mac to Surface. Its FastAPI service,
  scheduler, and restored PostGIS database now run together as a Docker Compose
  project restricted to Surface's Tailscale address. Updated the Mac-hosted
  Homepage link and scheduled-job health integration to query Surface.
- Preserved a final compressed database dump, the original Mac application
  tree, Docker volume, stopped PostgreSQL container, and disabled launchd jobs
  as a complete rollback path. No source database or application data was
  deleted.

### Dashboard

- Added Surface's Tailscale-only Uptime Kuma dashboard to the Mac-hosted
  Homepage `Infrastructure` group alongside Beszel and Netdata.

## 2026-08-17

### Reliability

- Added a root-owned Saturday 3:00 AM maintenance job that requires AC power,
  installs recommended macOS updates with Apple's updater when pending, and
  otherwise performs the weekly reboot. Update and reboot output is retained in
  the system-update log; startup health remains responsible for post-reboot
  service verification. The job verifies that an update is no longer pending
  before issuing any fallback reboot.


### Services

- Fixed scheduled BlueMap mask generation to skip zero-byte Minecraft region
  placeholders that the server tolerates, allowing nightly renders to proceed.

## 2026-08-16

### Dashboard

- Added the tailnet-only Immich photo library to Homepage under Personal with
  container stats and its Tailscale Serve URL on port 2283.


### Services

- Added Immich 3.1.0 as a tailnet-only Docker Compose stack for Apple Silicon.
  The web service listens on `localhost:2283` and is published through
  Tailscale Serve on port 2283; the database stays on the internal disk under
  ignored `data/immich/postgres/`, while the empty `photos/` directory is
  reserved as the future SSD library mount point. Added operational notes
  covering first login, secrets, upgrades, and safe photo-SSD attachment.


- Added Dawarich to the Homepage dashboard under Personal, including its Tailscale URL and container stats.
- Updated the Dawarich card to use the official Dawarich dashboard icon.
- Pointed the Dawarich card at the running app’s own `/icon.svg` asset so it matches the application branding.

Notable changes to the home server are recorded here in reverse
chronological order.


### Services

- Installed the official Jellyfin 10.11.11 Apple Silicon application through Homebrew and added a
  headless user LaunchAgent with crash recovery, standard internal-disk application state, LAN
  access on port 8096, and documented local-media and external-SSD migration procedures. Native
  execution preserves supported macOS scanning and VideoToolbox acceleration; no public route,
  router port forward, automatic port mapping, or Docker container was added.


### Services

- Added Dawarich 1.10.3 as a persistent Docker Compose stack for Apple Silicon.
- Bound Dawarich to `localhost:3002` and exposed it tailnet-only through Tailscale Serve.
- Added ignored local secrets configuration and operational notes in `docs/dawarich/README.md`.
## 2026-08-15

### Services

- Corrected Personal Observatory JPL close-approach identity so revised orbit solutions update one
  encounter instead of appearing as duplicate asteroids. Older raw solutions remain preserved as
  superseded revision facts; no service, listener, paid dependency, or browser egress was added.

## 2026-08-14

### Services

- Added a tailnet-only Briancraft II player-stat dashboard linked from Homepage. It reads the
  persisted world statistics and player-name cache through read-only mounts, compares individual
  playtime, deaths, mining, crafting, travel, combat, loot, and jumps, and breaks down each
  player's recorded killers without polling or waking the Minecraft server. Restyled the page
  with locally vendored Minecraft 1.21.1 dirt and stone textures, inventory-slot treatments,
  cached official player-skin heads, and a wide responsive three-column player layout with
  four-column stat grids. Player heads preserve both modern 64x64 and legacy 64x32 skin-sheet
  proportions. Retained server death messages replace Minecraft's generic player-killer cause
  with the responsible player's name when available. Removed the aggregate record cards and
  expanded each player to four stat rows covering damage, fishing, breeding, waystones, and sleep;
  a compact header dropdown replaces the button row and refresh timestamp for sorting players.
  Log-derived falling and lava/fire categories replace the remaining generic environmental deaths
  while respecting Minecraft's cumulative entity-attribution totals.
- Expanded the Personal Observatory Overview with a fuller local NWS weather synthesis and a
  compact six-period forecast. Current humidity, dew point, wind, pressure, visibility,
  precipitation reporting, and cloud layer context now remain visible alongside space weather
  and ingestion health; the existing API/Tailscale route serves the updated local-only build with
  no new listener, process, browser egress, or paid dependency.

## 2026-08-13

### Services

- Added data-driven Moon, Sun, Earth, and major-planet graphics to the local Personal Observatory.
  The Moon now uses a locally calculated Skyfield phase angle to show its actual waxing or waning
  illuminated limb; all artwork is bundled code-native SVG with no third-party browser requests,
  new service, listener, or data egress.
- Started a side-by-side Beszel 0.18.7 monitoring trial with its ARM64 Hub in
  Docker, persistent named-volume storage, a localhost-only listener at port
  8090, and tailnet-only Tailscale Serve access. Paired its native ARM64 macOS
  Homebrew agent over an outbound-only local WebSocket with the SSH listener
  disabled and its token file restricted to the owner; verified host samples
  and all nine running Docker containers. Netdata remains installed for deeper
  diagnostics during the trial.
- Added the new NOAA/WHOI marine-mammal streams to the existing local Observatory dashboard:
  Overview summarizes current advisories, latest reviewed detections, and platform coverage, while
  Earth exposes detection history, negative review effort, certainty, active zones, and retained
  provenance without adding a process, listener, or browser-to-third-party request.
- Applied the additive Personal Observatory ephemeral-marine migration and restarted the existing
  localhost-only API/scheduler LaunchAgents with 20 healthy collectors. NOAA right-whale Slow
  Zones now poll hourly, WHOI Robots4Whales New York Bight/Coastal New Jersey missions poll every
  30 minutes, and archival OBIS polling was reduced to weekly; no new listener, process, secret,
  paid service, or browser egress was introduced.
- Added first-class biodiversity visualization to the local Personal Observatory: Overview now
  surfaces recent nearby life, while Earth provides bounded activity trends, taxonomic and source
  coverage, recent occurrences, iNaturalist quality context, and eBird-notable reports with direct
  links to retained provenance. A read-only PostgreSQL summary endpoint avoids bulk browser data
  transfer; the existing localhost API/Tailscale route is reused with no new listener or egress.
- Installed Netdata 2.11.0 as a native Homebrew service for host CPU, process,
  memory-pressure, swap, disk, and network history. Bound its dashboard to
  `localhost:19999`, disabled anonymous telemetry and Netdata Cloud, and
  published it through tailnet-only Tailscale Serve HTTPS at
  `https://mac.example:19999/`; Funnel remains disabled.
- Expanded the independently versioned Personal Observatory with scheduled iNaturalist and OBIS
  biodiversity ingestion, including a 9,093-record local iNaturalist baseline and a deduplicated
  354-record regional marine seed. Activated two-hour eBird live collection with bounded HTTP 429
  backoff, adding a 1,648-record regional bird baseline while sharing the personal API allowance
  conservatively with Twitcher; the optional local Basic Dataset import remains disabled until an
  extract path is supplied. The existing localhost-only API and scheduler LaunchAgents now
  supervise 18 active collectors with no new listener, paid service, or browser data egress.
- Published the localhost-bound Personal Observatory frontend/API through a persistent,
  tailnet-only Tailscale Serve HTTPS route at
  `https://mac.example:8000/`; FastAPI remains bound to `localhost`, with no
  public Funnel or LAN listener added.
- Added a reusable, fully local MapLibre/PMTiles foundation for the Personal Observatory's
  Operations map, with its approximately 190 MB Northeast basemap and rendering assets stored
  under the ignored standalone-app data directory; the existing localhost-only API serves range
  requests, so no new listener, supervised process, or browser data egress was introduced. The
  map uses a quiet basemap and selection-focused animated data paths to keep dense station coverage
  legible while retaining the visual sense of observations flowing home.
- Refined the local Observatory Operations view with a 15-ingestion health bar, clearer station
  paths, concrete geographic feed labels, human-readable station provenance, and explicit
  measurements with accessible scientific tooltips. The more compact view distinguishes fixed
  stations from area/grid and remote feeds without exposing storage-oriented record types; this
  remains a read-only frontend/API change with no new listener or external browser request.
- Applied the additive Personal Observatory water-source migration and restarted the existing
  localhost-only API/scheduler LaunchAgents to collect USGS Water, HRECOS, NOAA CO-OPS/PORTS,
  NDBC, NYC DEP reservoir, and Riverkeeper Croton Point data at independent cadences.

### Reliability

- Removed the short-lived custom Server Health dashboard, its LaunchAgent, and
  its Tailscale Serve route after deciding that learning the native Netdata UI
  is simpler and avoids maintaining a redundant monitoring service.
- Corrected the Observatory's HRECOS `HPMNT` provenance label from Bear Mountain to its actual
  Piermont location without modifying retained upstream payloads.
- Corrected NOAA PORTS' literal `"null"` normalized name for station `n07010` to Newark Bay
  Entrance LB 18 without altering retained source payloads.
- Removed a spurious horizontal scrollbar caused by hidden measurement tooltips in the
  Observatory's selected-station panel.
- Moved Observatory measurement tooltips outside the scrollable station panel and constrained
  them to the browser viewport so definitions remain fully readable at panel edges.
- Simplified Observatory measurement chips by hiding unit suffixes at presentation time while
  preserving canonical units and labels in stored observations.
- Added presentation-only aliases for cryptic USGS, HRECOS, and NDBC measurement names while
  retaining source terminology and scientifically meaningful distinctions in stored data.
- Tightened the Observatory Operations header, equalized empty and selected station-panel sizing,
  identified NWS alert/forecast coverage as Southern Westchester under NWS New York, and removed
  selected-path arrow glyphs without changing collector or network exposure.
- Verified all 15 Observatory collectors as healthy after deployment, preserved the production
  PostGIS volume, and made overlapping observation upserts skip unchanged facts to reduce WAL,
  disk churn, and misleading update counts.

## 2026-08-12

### Services

- Extended the local Personal Observatory interface with read-only source-provenance drawers and
  bounded weather and earthquake history views; no new service, listener, or data egress was added.
- Added time-bucketed weather/space-weather exploration, earthquake filters, chart-to-provenance
  links, and bookmarkable frontend state without adding export, write, or event-generation paths.

## 2026-08-11

### Services

- Added the local read-only Personal Observatory frontend to the existing supervised FastAPI
  process; no new listener, LaunchAgent, external service, or data egress was introduced.

- Established `apps/the-observatory/` as an independently versioned application
  repository, excluded from the parent home-server repository.
- Started a localhost-only PostgreSQL 16/PostGIS 3.5 service with persistent
  storage, applied the initial Observatory schema, and completed the first
  successful USGS earthquake ingestion.
- Configured the official AMD64-only PostGIS image to run under Docker
  emulation on the Apple Silicon host.
- Added user LaunchAgents that keep the localhost-only Observatory API running
  and collect the USGS earthquake feed every five minutes with overlap and
  database-level idempotency protection.
- Extended transition-only Discord monitoring to alert after two consecutive
  Observatory API/database health failures, three consecutive USGS failures,
  or 20 minutes without a successful collection, and to announce recovery.
- Added a Homepage monitoring card for Observatory API/collector status, last
  successful collection age, fetched records, and insert/update counts using a
  non-sensitive, read-only host health snapshot.
- Replaced the five-minute USGS-only LaunchAgent with a supervised Observatory
  scheduler that runs USGS, NWS, NOAA SWPC, CelesTrak, and NASA/JPL collectors
  at application-configured cadences while keeping the API process separate.
- Extended Observatory Discord monitoring from USGS-only history to all named
  collectors using cadence-relative freshness, retaining the three-failure
  notification threshold and transition-only recovery messages.
- Removed the stopped, ephemeral Observatory PostGIS integration-test
  container after the initial test run; production storage was preserved and
  the production database and API health checks remained healthy.
- Pruned six dangling Docker images left by earlier builds, reclaiming 6.36 GB;
  retained the build cache and the active Twitcher poller's reusable Compose
  network.

### Security

- Restricted the server-local Observatory environment file to its owner and
  kept PostgreSQL and the supervised API listener bound to localhost.

### Development

- Installed `uv` with a managed Python 3.12 runtime for the independently
  versioned Observatory application and verified its full unit and PostGIS
  integration suite.
- Documented installation, inspection, logging, and database-startup behavior
  for the supervised API and scheduled USGS collector.
- Increased scheduled-job health checks from every 15 minutes to every five
  minutes so sustained Observatory API failures are detected in about 10
  minutes without increasing routine notification volume.

## 2026-08-07

### Reliability

- Fixed scheduled BlueMap mask generation under launchd by using the explicit
  Homebrew Node path, and made Docker, Minecraft, and mask preflight failures
  immediately record a failed render status.
- Fixed incremental renders under macOS's system Bash by avoiding expansion of
  an empty command-argument array.
- Prevented intentional BlueMap skips and in-progress renders from clearing a
  prior failed or overdue alert; recovery now requires a successful render.

## 2026-08-04

### Reliability

- Restored all Compose services after the macOS update reboot and verified all
  six containers, five configured healthchecks, and the local and tailnet web
  endpoints.
- Updated startup recovery to explicitly reconcile the always-on Homepage,
  Portainer, and BlueMap services after Docker becomes available while leaving
  Minecraft under its existing schedule.
- Corrected startup-health accounting to inspect stopped containers, treat
  their configured healthchecks as unhealthy, require two stable healthy
  samples, and include per-service states in failure notifications.
- Made startup-health accounting aware of the Minecraft availability schedule,
  excluding an intentionally stopped server from the expected container and
  healthcheck totals while still detecting failures during availability hours.

## 2026-08-03

### Host

- Updated the standalone macOS Tailscale app from 1.98.9 to 1.102.1.

### Monitoring

- Added a post-login Discord recovery check that distinguishes macOS updates
  from ordinary restarts and reports dynamic Compose, healthcheck, Tailscale,
  and critical LaunchAgent status after Docker finishes starting.
- Added transition-only Discord alerts for low and critical battery thresholds
  plus failed, overdue, and recovered scheduled jobs.
- Removed residual Netdata configuration stubs, runtime directories, and empty
  repository directories after uninstalling the trial deployment.
- Added an owner-supervised macOS power watcher that sends Discord notifications
  when the server switches to battery power and when AC power returns.
- Added persistent transition state, failed-delivery retries, owner-only
  credential storage, and `launchd` restart supervision.

### Services

- Limited scheduled BlueMap renders to days when at least one player joined,
  while preserving unconditional manual and forced renders and reporting
  no-activity skips as healthy scheduled checks.

### Development

- Added trusted project-local Codex command rules for routine Git, Docker
  Compose, testing, validation, GitHub, and host-diagnostic workflows while
  keeping dependency installation and unmatched commands subject to approval.

## 2026-07-31

### Services

- Added an automatically generated BlueMap mask that buffers inhabited chunks
  by one chunk and fills everything enclosed by its outer boundary, removing
  interior holes without revealing terrain connected to the unexplored
  exterior.
- Added cached, read-only region scanning before nightly renders; mask output
  is written atomically and a failed or empty scan prevents rendering.
- Enabled rendering of selected pregenerated chunks that lack saved lighting
  data, accepting fully lit terrain and possible cave geometry to eliminate
  remaining map holes.
- Added an explicit force-render mode for safely applying BlueMap render
  setting changes without deleting existing map data.

### Reliability

- Replaced ad hoc BlueMap completion supervisors with a PID-owned render lock
  that temporarily holds Minecraft, cleans up on renderer exit, and triggers
  immediate schedule reconciliation without touching administrative holds.
- Added stale-lock cleanup to the Minecraft scheduler for interrupted BlueMap
  wrapper processes.
- Added a Homepage BlueMap render-status widget showing active, successful,
  failed, and stale states plus completion age, duration, and render mode.
- Increased BlueMap's renderer to a 1 GiB Java heap inside a 2 GiB container
  after a dense tile exhausted the default heap during the initial render.
- Configured BlueMap's renderer to exit immediately on a future heap
  exhaustion instead of remaining stuck indefinitely.

## 2026-07-30

### Monitoring

- Renamed the Briancraft II Homepage card descriptions and BlueMap card title
  for clearer, friendlier labels.
- Added a Homepage card for macOS update availability.
- Added a read-only host update check that runs at login and every six hours;
  it reports update status without automatically installing anything.
- Removed the Twitcher card's response-time badge so its ingestion-health
  widget remains the single, meaningful status indicator.
- Replaced the last-ten-runs count with a rolling three-day success percentage
  that treats missed scheduled runs as unsuccessful.

### Services

- Added a tailnet-only BlueMap viewer for Briancraft II with a standalone,
  resource-limited renderer and an always-on lightweight web container.
- Limited BlueMap to inhabited Overworld chunks, excluding untouched
  pre-generated terrain, caves, and other dimensions.
- Added a nightly incremental render after Minecraft's scheduled shutdown;
  rendering is skipped if Minecraft is running or another render holds the
  lock.
- Added an owner-supervised host watcher that sends a Discord notification
  only after a player successfully joins the Briancraft II server.
- Added persistent log cursor tracking, Minecraft log replacement handling,
  delivery retries, and `launchd` restart supervision.
- Tied the watcher lifecycle to the existing Minecraft schedule, including
  administrative hold and force-running overrides, so it is unloaded whenever
  the server is unavailable.

### Security

- Restricted the server-local Twitcher backend and poller environment files
  to the owning macOS account.
- Restricted Homepage's published port to localhost and added a tailnet-only
  Tailscale Serve endpoint.
- Kept the Discord webhook URL in a Git-ignored, owner-only runtime file and
  prevented it from appearing in watcher logs.
- Disabled Discord mention parsing in notification payloads.

### Operations

- Grouped operational scripts into service-specific directories under
  `scripts/`.
- Updated the active Twitcher cron entry and Minecraft LaunchAgents to use
  the organized script paths.

### Documentation

- Grouped Minecraft, Twitcher, and Portainer operations documentation into
  service-specific directories under `docs/`.
- Added a documentation index and retained the root changelog for host-wide
  changes.
- Documented webhook provisioning, notification testing, LaunchAgent
  installation, operational checks, and credential rotation.

## 2026-07-26

### Services

- Added a Docker Compose service for the Briancraft II Fabric 1.21.1 server,
  installed from a local CurseForge export.
- Pinned the Java 21 Minecraft image to an immutable digest.
- Configured the Java heap to start at 2 GiB and grow to at most 4 GiB.
- Stored generated server and world data under the Git-ignored `minecraft/`
  directory.
- Published TCP port 25565 for LAN access; no public router forwarding was
  configured.
- Configured a two-minute graceful shutdown window, automatic container
  restart, and a startup-aware health check.
- Disabled Minecraft's tick watchdog so long server ticks do not terminate
  the container.
- Added a Briancraft II Homepage card with container health, resource usage,
  and scheduled running/stopped state.
- Enabled automatic Java-process pausing after ten minutes without players.
  Removed active Minecraft-port polling from Homepage so dashboard refreshes
  do not wake the paused server.
- Added a self-correcting `launchd` schedule: 5:00 PM to midnight on weekdays
  and 7:30 AM to midnight on weekends, in `America/New_York`.
- Added an administrative schedule hold that keeps Minecraft stopped until
  explicitly released, without disabling the schedule LaunchAgent.
- Added a temporary force-running schedule override for supervised
  maintenance.

### Security

- Replaced Homepage's direct Docker socket access with a pinned, internal-only
  Docker socket proxy.
- Limited Homepage to read-only container API endpoints and disabled all
  Docker API POST operations through the proxy.
- Restricted the Minecraft runtime tree and source modpack ZIP to the owning
  macOS account.

### Host

- Increased Docker Desktop's memory allocation from 4 GB to 6 GB.

## 2026-07-24

### Development tooling

- Installed Node.js 26.5.0 and npm 12.0.1.
- Installed Codex CLI 0.145.0.
- Installed GitHub CLI 2.96.0 and authenticated it.

### Host

- Changed the macOS system timezone from `America/Los_Angeles` to
  `America/New_York`.

### Documentation

- Added `AGENTS.md` with repository-specific operating and safety guidelines.
- Added `docs/CHANGELOG.md` to record notable server changes.
- Replaced the initial setup README with a concise current service inventory,
  access guide, repository layout, and service-specific operations commands.

### Security

- Restricted Portainer's published port to `localhost:9443`.
- Added a tailnet-only Tailscale Serve proxy for Portainer at
  `https://mac.example`.
- Kept Tailscale Funnel disabled so Portainer is not exposed publicly.
- Recreated the Portainer container while preserving its existing
  `portainer_data` volume.
- Changed the Compose template to bind new services to localhost by default
  and require an explicit choice for LAN access.
- Added `backend/.env.poller` to Twitcher's ignore rules to protect its
  ingestion secrets from accidental commits.

### Reliability

- Pinned Portainer to version 2.39.5 and Homepage to version 1.13.2 using
  immutable image digests, preventing unplanned changes from floating
  `latest` tags.
- Added tested server-owned Twitcher poller runner and rollback scripts with
  overlap prevention, cached-image fallback, image digest/revision logging,
  daily compressed log rotation, and 90-day structured run history.
- Added `docs/TWITCHER_POLLER.md` with the intended schedule and rollback
  procedure.
- Activated the GHCR-based Twitcher poller using the merged Linux ARM64 image
  from source revision `8cbed20eb72d2c66dd8ed8fee64752c8d5210df0`.
- Replaced the old Git-pull/local-build cron job with the server-owned runner
  at midnight, 2:00 AM, and every two hours from 6:00 AM through 10:00 PM,
  intentionally skipping 4:00 AM.
- Completed a supervised production run successfully: 59 targets and 59
  correlated eBird API calls with zero failures, plus 7 updated incident
  summaries and 9 freshness skips. Verified the structured history record,
  Neon poll-run correlation, and one-shot container cleanup.
- Added a Twitcher Ingestion card to Homepage backed by a local-only health
  service. The card summarizes the latest run and recent success rate, reports
  eBird request counts, and marks failed, degraded, missing, or stale scheduled
  runs unhealthy while accounting for the intentional overnight schedule gap.
  The service reads poller history through a read-only mount and publishes no
  host port.
- Updated the Homepage links so Portainer opens through its tailnet-only
  Tailscale Serve URL and the Twitcher health card opens the deployed Vercel
  frontend.
- Removed eight unattached anonymous PostgreSQL test volumes and the obsolete
  locally built `backend-twitcher-poller:latest` image. Preserved the active
  Portainer data volume, all running service images, and the cached immutable
  GHCR production poller image.

### Backups

- Added `scripts/backup-portainer.sh` to create verified local snapshots of
  the `portainer_portainer_data` volume with a 14-backup retention limit and
  owner-only filesystem permissions.
- Added `docs/PORTAINER_BACKUPS.md` with backup behavior, storage limitations,
  and a guarded restore procedure.

## 2026-07-23

### Host

- Set up a MacBook Pro A2918 as a headless home server.
- Kept macOS as the host operating system.
- Installed Homebrew, Tailscale, Docker Desktop, and Docker Compose.
- Disabled system sleep with `sudo pmset -a disablesleep 1`.
- Enabled optimized battery charging and configured a 95% maximum charge.
- Configured Docker Desktop and Tailscale to start automatically at login.
- Configured remote SSH access through Tailscale.
- Initialized the home server Git repository.

### Services

- Installed and initialized Homepage.
- Installed and initialized Portainer.

### Twitcher automation

- Moved scheduled Twitcher ingestion to a cron job on the server.
- Configured the job to run every two hours from 1:00 AM through 9:00 PM.
- Each run:
  1. Updates the Twitcher repository with `git pull --ff-only`.
  2. Builds the poller image.
  3. Starts the Render backend through its health endpoint.
  4. Runs the poller in a temporary container, which performs ingestion
     against the production services.
  5. Removes the temporary container after completion.
- Added local file logging at `apps/twitcher/backend/poller.log`.

### Troubleshooting

- Resolved a macOS Keychain credential issue by removing the Docker Desktop
  credential-store setting from `~/.docker/config.json`.
