# Changelog

All notable changes to **NETFORY** (SmartHoldem decentralized P2P client) are documented [here](https://smartholdem.io/devhub/).

Component versions covered in this file:

- NETFORY client bumped from **1.0**
- Oxid Mail workspace (`oxid-mail/*`) - bumped from **0.1.0** to **0.2.0**
  alongside NETFORY 1.120.0, then to **0.3.0** (encrypted mail protocol v1),
  to **0.5.0** (Iroh P2P delivery + contacts + QR share), to **0.6.0**
  (attachments Dropzone + inline NTFRY + attachment preview overlay
  + honest P2P delivery progress) and finally to **0.7.0** (paid unlock for
    unverified dApps, reserved-names guard, recipient chips + contact
    autocomplete, ReaderView empty-screen fix, `finish_progress` visibility
    fix), then to **0.8.0** (P2P reply keys in the NTFRY envelope +
    auto-contact on receive).
    Every crate in the workspace (`mail-core`, `mail-crypto`, `mail-store`,
    `oxid-bridge`, `src-tauri`) and the Vue UI share the same version number.

---

## [1.135.37] (Network Map: cloud provider badge inline before the pubkey)

### Fixed

- **YA / GD / MAIL badge wrapped to a new line and broke the Cloud Seeders
  table layout.** The badge moved from the TYPE cell (after the CLOUD
  chip) into the NODE cell — it now sits on the SAME line, right before
  the short author pubkey (`[YA] 039d6f18…6ef35b`), with `white-space:
  nowrap` so it never wraps. Provider tooltip and test ids unchanged.
- Versions bumped to **1.135.37** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.36] (Torrent modals localized — en/ru/id)

### Changed

- **`TorrentMagnetModal` (magnet download dialog) fully moved to i18n**
  (new `torrents.*` section in `en.ts` / `ru.ts` / `id.ts`): title
  fallback “Loading metadata…”, DHT fetch status, “Torrent files (N)”,
  “Save folder”, “Sequential download (for video streaming)”,
  “Selected: N / M”, START / STARTING… button, error strings (no source /
  metadata failed / start failed).
- **`TorrentStreamModal` (stream-while-downloading player)** localized
  too: stream errors (open failed / no media files / metadata not ready),
  the playback hint line and size units (KB/MB/GB per locale).
- Versions bumped to **1.135.36** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.35] (Sled DB migrated out of %TEMP% into the app data/ directory)

### Changed

- **`smartnet_db` (sled) moved from `%TEMP%/smartnet_db` to
  `<app_data>/data/smartnet_db`** — next to `dapps/`, `manifests/`, `keys/`
  (Windows: `%APPDATA%\io.smartholdem.smartnet\data\smartnet_db`; Linux:
  `~/.local/share/io.smartholdem.smartnet/data/smartnet_db`; macOS:
  `~/Library/Application Support/io.smartholdem.smartnet/data/smartnet_db`).
  TEMP cleanup (Windows Storage Sense, CCleaner, reboot policies) could
  wipe the live DB — killing the client and erasing settings/config keys.
- **One-time automatic migration**: on first start the existing TEMP DB is
  moved via `rename` (instant on the same volume) with a recursive-copy
  fallback; if the copy fails the client stays on the TEMP DB and records
  the reason in `%TEMP%/smartnet-crash.log`. No AppHandle needed — the
  OS app-data dir is resolved manually (`os_app_data_dir()`), matching
  Tauri’s `app_data_dir()` layout, because the Lazy DB initializes before
  Tauri starts. If a user overrides `cfg:data_dir` (movable data/), the DB
  intentionally stays at the DEFAULT location — the override itself is
  stored inside the DB (chicken-and-egg).
- Versions bumped to **1.135.35** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.34] (Silent client exit forensics + headless seeder retry backoff)

### Added

- **Crash forensics for the “client just closed with no warning” reports.**
  Root of the silence found in `lib.rs`: ANY `Destroyed` event of the main
  window calls `exit(0)` — so when the OS kills the WebView renderer
  (WebView2 crash, OOM, GPU driver reset), the whole app vanishes without a
  trace (Windows GUI build has no visible stderr, syslog is in-memory
  only). Now:
  - a global `panic::set_hook` writes every thread panic (thread name +
    location + message) to a persistent **`%TEMP%/smartnet-crash.log`**;
  - `CloseRequested` on the main window sets a flag; a `Destroyed` WITHOUT
    a prior close request is logged to the same file as an abnormal
    termination (probable renderer crash) before the process exits.
    Non-blocking appends only — the main process never hangs (async rule).
    After the next silent exit, check `%TEMP%\smartnet-crash.log` for the
    recorded cause.
- **Headless seeder: exponential retry backoff (5 → 15 → 60 min, max 60).**
  Targets that fail all providers get a per-app pause (1st fail 5 min, 2nd
  15 min, 3rd+ 60 min) instead of a QUIC/relay dial storm every 30-second
  cycle — this is what pushed the server CPU past 100% while six stale
  targets were permanently unfetchable. A successful fetch clears the
  backoff; a changed target hash (republish) resets it so new blobs are
  tried immediately. Skip reason + retry countdown go to the debug log,
  the warn line now shows `backoff N min (fail #K)`.

### Changed

- Versions bumped to **1.135.34** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.33] (P2P updater UI fully localized — en/ru/id)

### Changed

- **All updater-facing strings moved to i18n** (new `updater.*` section in
  `en.ts` / `ru.ts` / `id.ts`). Previously hardcoded Russian text is now
  translated for every supported locale:
  - status bar phases (`updater.ts`): “Starting update download…”,
    rotating search hints (“Connecting to torrent network…”, “Searching
    for a source in DHT…”, “Waiting for live release seeds…”, “Requesting
    torrent metadata…”), live download line (“Downloading · seeds N /
    peers M · ↓ speed”), “Downloaded — preparing to install…”, localized
    speed units (B/s, KiB/s, MiB/s);
  - `UpdateBar.vue`: fallback status, “Update vX is ready to install”,
    Update button, “Launching the installer…”;
  - `UpdateModal.vue`: UPDATE tag, “New version downloaded!”, missing-
    installer warning, Later / Update now buttons;
  - `installNow()` error “Installer not found for the current OS.”
- Versions bumped to **1.135.33** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.32] (Headless seeder: fix version-inflation bug that pinned stale blobs)

### Fixed

- **Headless seeder kept fetching STALE blob hashes forever (“provider
  returned no data / blob absent on provider”).** In the target-selection
  loop (`seeder/src/main.rs`) the per-provider version was inflated via
  `p.version.max(app.version)` (app.version = highest across ALL
  providers). A provider still announcing an OLD hash therefore looked
  like it carried the newest version; if it iterated first, its stale hash
  became the fetch target and the node actually serving the NEW blob was
  discarded (`ver == entry.ver` with a different hash → skipped). Verified
  against the live seeder at x.x.x.x:9466 `/metrics`: DEX was
  targeted at provider `b310b058b1…` (stale) while the publisher’s node
  `a06678f058…` was live-announcing the new blob `fce28547…`. Now `ver =
  p.version` — the provider genuinely serving the highest version wins the
  target, so republished dApps (free→paid re-uploads) propagate to
  headless seeders again. Rebuild + restart of the seeder binary required.

### Changed

- Versions bumped to **1.135.32** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.31] (Gray DIRECT LINK until ready + cloud provider badges on Network Map)

### Added

- **My DAPPS: ⚡ DIRECT LINK and ▦ QR buttons are now disabled (gray) until
  the direct link is ready.** Links are prefetched sequentially when the
  list loads (and re-fetched after each publish/update via `publishTick`);
  the backend lazily re-seeds on a miss (v1.135.30), so buttons turn active
  automatically. Clicking always copies an already-built
  `sth://<id>?n=<nodeId>&h=<hash>` from the cache — no more “NOT SEEDING
  YET” flashes on the button label (the state moved to the tooltip).
- **Network Map: provider badge (YA / GD / MAIL) on each Cloud Seeder
  row.** The same author pubkey published on Yandex Disk and Mail.ru Cloud
  used to look like duplicate rows; the badge (parsed from the
  `sn://seed/<y|g|m>@…` URI prefix, full provider name in the tooltip) now
  tells them apart. Row `:key` switched from pubkey to the full URI —
  fixes Vue duplicate-key warnings for multi-provider authors.

### Changed

- Versions bumped to **1.135.31** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.30] (Direct link works right after publishing — lazy re-seed in `app_share_info`)

### Fixed

- **DIRECT LINK / QR showed “NOT SEEDING YET” for a freshly published
  dApp.** The `sth://<id>?n=<nodeId>&h=<hash>` link is built from
  `SEEDED_APPS`, which is only filled by the heavy `seed_all()` pass
  (deterministic tar rebuild + iroh import) fired asynchronously after
  `finalize` — clicking the button during that window (or if the background
  re-announce failed) returned `not-seeded`. `app_share_info` is now async:
  on a miss it awaits `p2p_impl::seed_all()` itself and retries the lookup,
  so the button waits a few seconds and returns a valid link instead of
  erroring. Frontend call is unchanged (`invoke('app_share_info')`).

### Changed

- Versions bumped to **1.135.30** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.29] (Network defaults: drop retired DHT seeder x.x.x.x)

### Changed

- **`DEFAULT_SEEDERS` (Mainline DHT bootstrap) no longer includes
  `x.x.x.x:6885`.** The built-in defaults are now exactly
  `x.x.x.x:6881` and `y.y.y.y:6881` (both were already
  present; the retired anchor is excluded per operator request). The
  Settings placeholder for WAN trackers also stopped referencing the
  retired IP (now shows `z.z.z.z:7374`, the first live default
  tracker).
- SMARTNET RELAYS defaults were verified as already built-in
  (`DEFAULT_SMARTNET_RELAYS = relay-fsn7.sth.cx + relay-ru1.sth.cx`,
  always added to the endpoint relay map) — no change needed.
- Versions bumped to **1.135.29** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

---

## [1.135.28] - 2026-08 (Official Repository: full pagination of on-chain registry scans)

### Fixed

- **Registry scans only read the first 100 transactions.** Both the client
  (`scan_registry()` in `official.rs`) and the tracker
  (`official_scan_registry()` in `tracker/src/main.rs`) fetched a single
  `page=1&limit=100` - once the admin issues more than 100 approvals, the
  oldest developer/dApp entries would silently drop out of the Official
  Repository. Both scanners now walk all pages up to `meta.pageCount`
  (safety cap: 50 pages = 5000 tx), stopping early on an empty `data`
  array. Verified against the live registry
  (`SdAppjWaQf4v2JjuubvtwsKe3yuUCuq8i8`): the response `meta.pageCount`
  field matches the implementation, and the first on-chain approval
  (`vendorField=SWzrpsh9jYH9fx3FnY4Sh8KguVk1gW3tEa`) parses correctly.
- Partial results are never persisted: the client propagates any page error
  (`node_get` already fails over across the node pool, and `refresh_now`
  keeps the previous snapshot on error); the tracker restarts the scan on
  the next pool host - the registry snapshot is always replaced atomically
  with a complete list.

### Changed

- Versions bumped to **1.135.28** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.27] - 2026-08 (Build fix: `tauri:build:bonsai:mail:win/linux/mac` shipped a STALE frontend dist)

### Fixed

- **The `yarn tauri:build:bonsai:mail:win` (and `:linux` / `:mac`) shortcuts
  never rebuilt the web assets.** They pass `--config
  src-tauri/tauri.mail.conf.json`, which sets `beforeBuildCommand: ""` -
  that override exists for `build-mail-bundle.cjs` (which runs `yarn build`
  itself), but the platform shortcuts bypass that script entirely. Result:
  every MSI/deb/dmg baked in whatever old `frontend/dist` was lying around
  (users kept seeing pre-1.135.22 UI strings such as the old Russian
  “Yandex Disk” Cloud Seeders hint on the Network Map, despite the source
  being long fixed). All three shortcuts now run `yarn build` before
  `tauri build`. The `[STALE UI!]` window-title marker
  (`syncTitleVersions()` in `AppLayout.vue`) flags such mismatches at
  runtime.

### Changed

- Versions bumped to **1.135.27** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.26] - 2026-08 (Encrypted discovery_cache media on Cloud Seeders - no plaintext images on cloud disks)

### Added

- **Catalog media (dApp logos/previews) replicated to Cloud Seeders is now
  stored encrypted.** Files in `/ntfryseed/discovery_cache/` are uploaded as
  `<safeId>-<prefix>-<hash>.<ext>.enc` (e.g.
  `SXLN…Guc-preview-7186…95d1.png.enc`) instead of plaintext PNG/JPG, so
  Yandex Disk / Mail.ru Cloud / Google Drive never see the images in the
  open and cannot content-scan or index them.
- **No separate key index needed.** The AES-256-GCM key is derived
  deterministically from the filename components via
  `blake3::derive_key("ntfry discovery media enc v1", safeId|prefix|hash)`
  (`media_key()` in `cloudseeder.rs`). Any client reconstructs the key from
  the announced app id + media hash, decrypts, and restores the original
  file name locally (`<prefix>-<hash>.<ext>` in `data/discovery_cache/…`).
  This is obfuscation from cloud-disk scanners, not secrecy - authenticity
  is still enforced by the mandatory `blake3(plaintext) == announced hash`
  check, so a tampered/forged blob is rejected and the next seeder is tried.

### Changed

- `replicate_discovery_media()` now uploads **only** `.enc` files (both own
  dApp media from `data/dapps/<id>/` and the foreign-dApp cache); upload
  size cap extended by 28 bytes for the GCM nonce+tag overhead.
- `fetch_discovery_media()` tries the encrypted `.enc` name first, then
  falls back to the legacy plaintext name - old files already sitting on
  seeders keep working; nothing is deleted from the cloud.
- Versions bumped to **1.135.26** across `package.json`, `tauri.conf.json`,
  `Cargo.toml` and `Cargo.lock`.

## [1.135.25] - 2026-08 (Build fix: remove trailing garbage at end of p2p.rs that broke `cargo build --features p2p`)

### Fixed

- **`cargo build --features p2p` failed with `error: unexpected closing
  delimiter: )` at `src/p2p.rs:5857`.** After the previous edit that
  translated all `doctor_run` result strings to English (v1.135.23), a
  three-line garbage tail remained at the very bottom of the file:
  ```
  ures p2p)".to_string(),
      }]
  }
  ```
  It was a leftover from a partially-deleted `#[cfg(not(feature = "p2p"))]`
  stub in an earlier session. When the crate was built **without**
  `--features p2p`, the parser reached a different code path and the
  garbage was ignored; with the `p2p` feature enabled the parser reached
  end-of-file and hit the stray `)`. Removed the tail.
- While cleaning that up, both `#[cfg(not(feature = "p2p"))]` stubs for
  `net_doctor_run` / `net_doctor_check_relays` now also emit English
  (`"P2P is disabled in this build (rebuild with --features p2p)"`) instead
  of Russian, to match v1.135.23.

### Notes

- No behavioural change when built with `--features p2p` - the fix is
  purely a syntax cleanup.
- Versions bumped to **1.135.25** across `package.json`, `tauri.conf.json`,
  `Cargo.toml`, and `Cargo.lock` (`smartnet-client`).

---

## [1.135.24] - 2026-08 (Storages & Media: rename "Web3 player" to "Web4 player" on the P2P player & albums card)

### Changed

- **Storages & Media page** → `[ P2P player & albums ]` card description
  updated from `"Local music library and Web3 player - buy albums
  straight from artists, no middlemen."` to `"Local music library and
  Web4 player - buy albums straight from artists, no middlemen."`
  (`frontend/src/views/Storages.vue`).
- Em-dash `-` replaced with a plain hyphen `-` per user request.

### Notes

- Copy-only change; no runtime behavior modified.
- Versions bumped to **1.135.24** across `package.json`, `tauri.conf.json`,
  `Cargo.toml`, and `Cargo.lock` (`smartnet-client`).

---

## [1.135.23] - 2026-08 (Network Doctor: diagnostics results in English only)

### Changed

- **All Network Doctor result strings** (Settings → Network → Network Doctor)
  are now emitted in English by the Rust backend (`p2p.rs`, `doctor_run`).
  Previously the row descriptions and the final verdict were hardcoded in
  Russian, which looked out of place next to the already-English row labels
  ("IPV6 OUTBOUND", "NAT TYPE", "VERDICT", …) in the UI.
- Every check now uses concise, protocol-neutral English:
  - `ipv6`: "IPv6 outbound works - two v6 nodes connect directly, bypassing
    CGNAT" / "no outbound IPv6 - on mobile networks a direct connect will
    hit CGNAT"
  - `node`: "P2P node failed to start: {err}" · verdict "diagnostics
    aborted - node did not start"
  - `sockets`: "iroh bound: {addrs} (IPv4 only - no IPv6 stack available on
    this host)"
  - `udp`: "QUIC/UDP outbound: v4=yes/no · v6=yes/no (QAD probes to
    relays)" / "net_report did not finish within 20s - DPI likely drops all
    unknown UDP"
  - `nat`: "Symmetric NAT (typical mobile CGNAT): hole-punch does not
    work…" / "Cone NAT: hole-punch possible · external address {ip}" /
    "NAT type unknown (too few UDP responses - UDP is likely filtered)"
  - `relay`: "{N} relays reachable via QUIC, best latency {ms} ms · home:
    {url}" / "no relay responded via QUIC (UDP path to relays is blocked)"
  - `relay_https`: "{ok}/{total} relays reachable over TCP/TLS 443, best
    {ms} ms · IPv4 TCP blocked by ISP - relays reachable over IPv6 TCP
    (bypassing CGNAT)"
  - `dht`: "nodes in table: {N} · BEP5-ping bootstrap: response
    received / no response (UDP to DHT is likely blocked)"
  - `dns`: "system DNS works" / "system DNS not responding, but DoH
    (Cloudflare) works - ISP is filtering DNS" / "neither system DNS nor
    DoH respond"
  - `verdict`: "direct P2P connections available: UDP open, IPv6 present -
    CGNAT is no obstacle" / "UDP open, but Symmetric NAT (CGNAT): direct
    only with v6 peers, everything else goes via relays. Network WORKS" /
    "UDP open, NAT traversable - network works in direct mode" / "UDP
    blocked by ISP. Network runs via relays over TCP/TLS 443 - SmartNet
    relays (relay-*.sth.cx) are built in, traffic will automatically route
    through them" / "neither UDP nor relays over TCP/TLS 443 are reachable
    - the network cannot operate in this environment"
- Fixed a truncated verdict string that read "The network is up and running." (a
  merge-mangled Russian phrase) - the corresponding English branch now
  correctly reads "UDP open, NAT traversable - network works in direct
  mode".

### Notes

- Row labels ("IPV6 OUTBOUND", "DUAL-STACK BIND", "NAT TYPE", "RELAYS
  (QUIC)", "RELAYS (TLS 443)", "MAINLINE DHT", "DNS / DOH", "VERDICT") and
  the top hint were already localised via `i18n/*.ts` and are unchanged.
- Rust source comments inside `doctor_run` remain in Russian (developer
  notes, not user-facing).
- Versions bumped to **1.135.23** across `package.json`, `tauri.conf.json`,
  `Cargo.toml`, and `Cargo.lock` (`smartnet-client`).

---

## [1.135.22] - 2026-08 (Network Map copy: rename "Yandex Disk fallback" to generic "Cloud Disks fallback")

### Changed

- **Network Map / Seeders help copy** now describes the Public Cloud Seeders
  fallback as generic **Cloud Disks** instead of naming a single provider
  ("Yandex Disk"). The Cloud Seeder layer already supports multiple providers
  (Google Drive, Mail.ru Cloud, Yandex Disk), so the UI wording is aligned
  with the actual architecture:
  - `en.ts` → `Public Cloud Seeders - encrypted archives on Cloud Disks
    fallback (anti-censorship layer). Identified by author pubkey, no OAuth
    required.`
  - `ru.ts` → `Public Cloud Seeders - fallback via encrypted archives on cloud storage …`
  - `id.ts` → `Cloud Seeders publik - arsip terenkripsi di Cloud Disks
    sebagai fallback …`
- Em-dash `-` after "Public Cloud Seeders" replaced with a plain hyphen `-`
  per user request.

### Notes

- Copy-only change; no runtime or protocol behavior modified.
- Versions bumped to **1.135.22** across `package.json`, `tauri.conf.json`,
  `Cargo.toml`, and `Cargo.lock` (`smartnet-client`).

---

## [1.135.21] - 2026-08 (Public Network Seeders: the selected publish account and secondary drive survive page navigation)

### Fixed

- **Primary publish account reset to the first one (Google) and the
  Secondary Drive went blank after leaving the Seeders page.** `activeAccId`
  was a plain ref re-initialized to `accounts[0]` on every mount; the
  secondary selection WAS persisted, but because the restored primary
  defaulted back to the first account (the very one chosen as secondary),
  the secondary got filtered out of the options and displayed empty.
- `activeAccId` is now persisted in `localStorage` (`sth_seeder_pub_acc`),
  restored on mount and validated against the current Crypto Vault account
  list (a detached account falls back to the first one). Selecting a primary
  that equals the current secondary clears the secondary - mirroring to the
  same disk is meaningless. (`Seeders.vue`)

## [1.135.20] - 2026-08 (Discover: periodic background Cloud Seeder scan while P2P is alive - new cloud-only dApps appear without manual rescans)

### Added

- **Background cloud scan** (`cloudseeder.rs::spawn_cloud_fallback`): the P2P
  scan works as before, and additionally - while P2P is ALIVE - the client now
  polls Cloud Seeders for new dApps in the background: first pass ~2 minutes
  after startup (to quickly fill the catalog), then once every 30 minutes,
  capped at **3 seeders per pass** (seeders with a healthy status are picked
  first). Previously clouds were polled only when P2P was fully unreachable,
  so dApps that existed only on cloud seeders (publisher offline) never
  appeared in the catalog of a P2P-connected client.
- `run_fallback_once` refactored into `run_fallback_scan(limit)` (`limit=0` =
  poll ALL seeders - emergency fallback and the manual Discover rescan button
  keep their old behavior). SYSLOG marks passes as `(background, P2P alive)`
  vs `(P2P unreachable)`.

### Verified as already implemented (no code change needed)

- A connected Cloud Seeder already publishes to `/ntfryseed/dapps/` BOTH the
  apps the user created AND the apps they installed (downloaded), free and
  paid alike: P2P install registers the manifest in the same `app:*` registry
  (`p2p.rs::install_app`), `build_network_state` publishes every non-draft
  entry from it, and the hourly auto-replication (`lib/seederRep.ts`) keeps
  the cloud state fresh.

## [1.135.19] - 2026-08 (Window title now shows the UI bundle version and flags a stale dist - core/frontend mismatches are impossible to miss)

### Added

- **Stale-bundle detector.** The UI bundle version (from `package.json`) is
  baked into the JS at build time (`vite define: __UI_VERSION__`) and compared
  on startup with the Rust core version (`getVersion()`). If they match, the
  title stays `NETFORY // SmartHoldem WEB4 P2P network v<core>` as before;
  if they diverge (an outdated `dist` slipped into the build - the exact
  confusion from the v1.135.18 report), the title becomes
  `…v<core> · UI v<ui> [STALE UI!]` and a warning line is written to SYSLOG.
  (`AppLayout.vue::syncTitleVersions`, `vite.config.js`, `vite-env.d.ts`.)

## [1.135.18] - 2026-08 (Catalog cards: rich no-preview fallback + preview cache is only replaced by a successfully downloaded NEW image)

### Fixed

- **Previews "reset" after the publisher edited app tags (Linux report).**
  The on-disk `discovery_cache` was never wiped - the re-announce after a
  metadata edit transiently (or persistently) carried EMPTY logo/preview
  hashes, and clients stopped even *looking* at their cache: an empty hash
  failed validation in `discovery_media`, and the UI marker logic never
  retried once set. Fixes on both sides:
  - Rust `discovery_media`: an EMPTY hash now returns the last cached file
    for that app (new `cached_media_any` helper, no network I/O) instead of
    rejecting; and when a valid hash can't be fetched anywhere (P2P + cloud
    both fail) the stale cached file is served as a last resort. The cache
    is physically replaced ONLY when a new image is successfully downloaded
    and validated (existing hash-change cleanup) - exactly "drop old only
    when the new one is available".
  - `Discovered.vue` `loadLogos`: per-app retry keys (`hash|providers?`)
    replace the one-shot '' markers - a fetch is re-attempted whenever the
    announced hash or provider availability CHANGES, and an already
    displayed image is never overwritten by a failure.

### Added

- **No-preview card fallback** (Discovered.vue): instead of an empty
  gradient, the preview area now shows the app type badge (FREE - green /
  PAID - amber / PRIVATE - violet, i18n ru/en/id), up to 5 tag chips, and
  the short description as a PERMANENT bottom strip in the same style as
  the hover strip on cards with a preview (`.disc-desc-strip.static`).

## [1.135.17] - 2026-08 (Linux embedded dApps, round 2: flat GtkFixed instead of GtkOverlay - input events reach the webviews again)

### Fixed

- **After 1.135.16 the dApp rendered in the right place but NOTHING was
  clickable - neither the dApp nor the main UI.** Cause: GtkOverlay creates a
  wrapper GdkWindow for each overlay child sized to the child's allocation;
  our GtkFixed overlay child filled the whole window (default GTK_ALIGN_FILL),
  so that wrapper window sat above the main webview and swallowed all input
  that didn't hit a dApp - and reparent side effects left the dApp's own
  input window dead too.
- New flat scheme in `linux_embed_bounds`: BOTH webviews live in a single
  `GtkFixed` (exactly the container wry natively supports for multiwebview):
  `vbox → GtkFixed { main-webview (0,0, full size), dApp webviews }`.
  The main webview tracks the window size via the fixed's `size-allocate`
  signal; dApps get hard bounds (`move_` + `set_size_request` + immediate
  `size_allocate`, mirroring wry's own fixed-parent path) and are kept above
  the main webview by raising their GdkWindow. Input now routes directly by
  each webview's GdkWindow geometry - no wrapper windows in between.

## [1.135.16] - 2026-08 (Linux embedded dApp webviews: reparented from the window GtkBox into GtkOverlay+GtkFixed - bounds finally apply)

### Fixed

- **Embedded dApp opened "under" the main UI (bottom half of the window) or
  swallowed the whole window on Linux.** Root cause confirmed in
  tauri-runtime-wry 2.11.4 + wry 0.55.1 sources: on Linux
  `Window::add_child` packs the child webview into the window's *vertical
  GtkBox* with `pack_start(expand=true, fill=true)` (`WebviewKind::WindowChild`
  → `build_gtk(default_vbox)`), so the main webview and the dApp split the
  window 50/50 - and wry's `set_bounds` only works for a GtkFixed (or x11)
  parent, making every `set_position`/`set_size` a silent no-op. The user's
  SYSLOG proved the frontend sent correct bounds (`sync-embed …
  host=[200,93 1013x628]`) that GTK ignored. The 1.135.10 hide/show
  workaround could never fix this and is now removed.

### Added

- `webview.rs::linux_embed_bounds` (Linux only): on the first bounds call the
  dApp webview is reparented out of the GtkBox into
  `vbox → GtkOverlay { main-webview, GtkFixed { dApp webviews } }`.
  GtkFixed is a no-window container - empty areas stay transparent to input
  (clicks reach the main webview), children get hard positions via
  `gtk_fixed_move` + `set_size_request` that survive any window re-layout,
  resize or maximize. Subsequent calls just move/resize within the fixed.
  Visibility state is preserved across reparenting; the reparent is logged to
  SYSLOG (`embed reparent GtkBox → GtkFixed [x,y WxH]`).
- New Linux-only dependency `gtk = "0.18"` (same crate version wry/webkit2gtk
  already use, so the dependency graph is unchanged).

## [1.135.15] - 2026-08 (Paid file replacement upgrades a FREE link-only dApp to PAID status with full network distribution)

### Changed

- **"Replace files (paid)" now transitions a free (link-only) dApp to the
  paid tier.** Replacing files is always a paid operation (dynamic update
  fee to the network escrow), but `update_app_at` used to preserve the old
  `paid=false`/`free=true` flags, so the app kept the "FREE - LINK ONLY" badge and was never network-announced. After a successful update
  payment the re-ingest now sets `paid=true`, `free=false`,
  `status=published` - `ingest_app` writes the `announce:<id>` key, so the
  app is distributed under all paid-app rules: mDNS `a{i}` records, tracker
  announces (with logo/preview hashes), headless seeders, catalog listing.
  SYSLOG records the transition (`update <id>: free → paid`).
- **Update-payment txid is now persisted**: the wallet passes the broadcast
  tx id of the update fee through `updateFromPath` → `update_app_at(txid)`
  and it lands in the manifest's `escrow_txid` (previous value kept as a
  fallback when absent).
- Update-modal note (`apps.updatePaidNote`, ru/en/id) now explains that a
  free app becomes paid and announced after a paid file replacement.

## [1.135.14] - 2026-08 (Free/link-only dApp logos & previews now propagate through Cloud Seeders - phase 2 of catalog media distribution)

### Fixed

- **Other clients showed an identicon + gradient instead of the logo/preview
  for a freshly published FREE (link-only) dApp.** Free dApps are not
  tracker-announced (announcing is a paid feature), so remote catalogs learn
  about them from a Cloud Seeder's `dapps.enc` - and those entries carried NO
  media hashes ("phase 2" placeholder), so clients never even attempted to
  fetch the images. On top of that, the publisher only replicated OTHER
  apps' cached media to the cloud (`discovery_cache/*`), never the media of
  its OWN dApps, so there was nothing to download even via the 1.135.11
  cloud fallback.

### Added

- **`dapps.enc` now carries `logo`/`preview` blake3 hashes** for every
  non-private dApp with a valid media file (`build_network_state`,
  cloudseeder.rs). The hash equals the iroh blob hash announced over
  mDNS/trackers, so all channels stay content-addressed and interchangeable.
- **Client side** (`merge_seeder_dapps`, p2p.rs): seeder-discovered apps now
  populate `PeerApp.logo`/`preview` from `dapps.enc`, so the catalog card
  lazily fetches media - first over P2P, then via the 1.135.11 Cloud Seeder
  download fallback when no live providers exist.
- **Publisher upload of OWN media** (`replicate_discovery_media`,
  cloudseeder.rs): every `cloud_seeder_publish` now also uploads the
  logo/preview files of the publisher's own non-draft, non-private dApps
  (free link-only included) from `data/dapps/<id>/` to
  `/ntfryseed/discovery_cache/<id>-<logo|preview>-<hash>.<ext>` - same
  content-addressed naming, same per-provider dedup map, ≤ 256 KB and
  PNG/JPG magic-byte validation. Result: catalog images stay available even
  when the publishing peer is fully offline.

## [1.135.13] - 2026-08 (Windows build fix, round 2: relative `frontendDist` resolved against `src-tauri/`, not the cwd)

### Fixed

- **`Unable to find your web assets … "dist" (which is …\src-tauri\dist)` on
  Windows after 1.135.12.** Empirically the base directory Tauri CLI uses to
  resolve a *relative* `frontendDist` is not stable: the Ubuntu log behind
  1.135.6/7 pointed at the cwd, while the Windows log from 1.135.12 clearly
  resolved against `src-tauri/`. An absolute path is not an option either -
  on Windows a drive path (`D:/…`) parses as a valid URL with scheme `d:`
  and broke the runtime (1.135.11 regression, fixed in 1.135.12).
- New bulletproof scheme in `scripts/build-mail-bundle.cjs`: the config keeps
  `frontendDist: "dist"` (a bare relative segment can never be mistaken for a
  URL), and the wrapper now mirrors the freshly built `frontend/dist` into
  `src-tauri/dist` (`fs.rmSync` + `fs.cpSync`, done on every run, covered by
  the existing `dist` .gitignore rule). With identical copies present in BOTH
  candidate base directories (`frontend/` = the pinned cwd, and
  `frontend/src-tauri/`), the path resolves correctly under either CLI
  semantics, on every OS.

## [1.135.12] - 2026-08 (Windows build fix: the app window showed a file index of `frontend/dist` instead of the UI)

### Fixed

- **Windows regression introduced in 1.135.7.** The build wrapper
  (`scripts/build-mail-bundle.cjs`) wrote an ABSOLUTE `build.frontendDist`
  (e.g. `D:/projects/.../frontend/dist`) into the generated merge config.
  Tauri parses `frontendDist` as a URL first, and a Windows drive path is a
  *syntactically valid URL* with scheme `d:` - so instead of embedding the
  dist directory, the production window navigated to it as an external URL
  and WebView2 rendered a raw file-index listing of the `dist` folder.
  Linux/macOS were unaffected because `/home/...` does not parse as a URL
  and correctly fell through to the directory variant.
- The generated config now writes `frontendDist` RELATIVE to the build cwd
  (`frontend/`), i.e. simply `dist`. The wrapper pins the cwd itself for
  every spawned command, so the relative path resolves identically on
  Windows, Linux and macOS, and a bare `dist` can never be mistaken for a
  URL. This keeps the original 1.135.7 fix intact (no dependency on the
  caller's cwd) while restoring the Windows build.

## [1.135.11] - 2026-08 (Catalog logos/previews now fall back to Cloud Seeders when P2P providers are offline)

### Fixed

- **dApp catalog media (440×180 previews and logos) failing to load when the
  publishing P2P peer is offline.** `discovery_logo` / `discovery_preview`
  fetched exclusively over Iroh P2P from the announced providers; if none of
  them were reachable (or the announce carried no providers at all), the
  catalog card stayed on the gradient fallback forever - users on fresh
  installs saw only 2 of many previews load. Cloud Seeders already
  *replicated* these media files to `/ntfryseed/discovery_cache/` (flat
  content-addressed names `<appId>-<logo|preview>-<hash>.<png|jpg>`), but no
  client-side *download* path existed.

### Added

- **Cloud Seeder download fallback for catalog media**
  (`cloudseeder::fetch_discovery_media`): when the P2P fetch fails,
  `discovery_media` (p2p.rs) now anonymously pulls the file from up to 5 known
  Cloud Seeders (seeders with a healthy status are tried first) without OAuth,
  via the same `fetch_public` used for dApp archive rescue. Integrity is
  enforced: the file name is content-addressed, so `blake3(bytes)` must equal
  the iroh hash from the announce - a tampered cloud file is rejected and the
  next seeder is tried. Size (≤ 256 KB) and PNG/JPEG magic-byte validation,
  local `discovery_cache` caching and LRU pruning are shared with the P2P
  path, so a successful cloud rescue is cached exactly like a P2P download.
  SYSLOG (`dapp` scope) distinguishes the outcomes: "fetched from cloud seeder",
  "cloud fallback also empty", and `seeder` scope logs blake3 mismatches.

## [1.135.10] - 2026-08 (Linux/WebKitGTK/RDP fix: native dApp child webview no longer covers the sidebar/tab-strip after open or maximize)

### Fixed - dApp webview overlaps NETFORY chrome on Linux (Ubuntu via xrdp)

- Symptom (reported on Ubuntu 26 via RDP, display scale 100%): opening a
  dApp made the sidebar and top tab strip disappear - the native child
  webview visually filled the entire main window. Pressing the maximize
  button or F11 made the problem worse (the dApp ate 100 % of the window),
  while on a fresh Dashboard the sidebar/topbar were rendered correctly.
- Root cause: on WebKitGTK (Tauri v2 uses wry → `add_child`), the child
  webview is a GTK widget positioned inside a `GtkFixed` container. When
  `set_position` / `set_size` are called after a window resize/maximize,
  WebKitGTK does not always re-run the fixed-layout pass - the widget
  keeps stale bounds, which on our first-open path happen to be the full
  main-window size. On Windows/macOS the platform layer requeues layout
  automatically, so the bug never surfaced there. RDP-driven X11
  compositors (xrdp) additionally serve the reflow later, which is why
  the issue is much more visible over remote desktop.
- Fix 1 (host-side, `AppLayout.vue::syncEmbedded`): wait for **two**
  animation frames after `nextTick()` before reading `hostRef`'s
  `getBoundingClientRect()`. This makes sure Vue has committed AND the
  browser has flushed the CSS/reflow, so the returned bounds already
  exclude the sidebar/topbar even mid-transition.
- Fix 2 (Rust, `webview.rs::set_embedded_bounds` +
  `open_embedded_webview`): on Linux only, wrap the
  `set_position`/`set_size` calls in `wv.hide()` … `wv.show()`. This is
  the canonical GTK trick to force `queue_resize`/`queue_draw` on a
  `GtkFixed` child, guaranteeing WebKitGTK re-applies the new geometry
  after every reflow (open / maximize / drawer toggle / terminal open).
- Diagnostic: `syncEmbedded` now emits a `ui_log` line with the exact
  host rect, window size, DPR and shrink offsets right before it moves
  the webview - so if the geometry ever misbehaves again, System Console
  shows the true numbers immediately (no DevTools needed).

### Chore

- Version bump `1.135.9` → `1.135.10` (`package.json`, `tauri.conf.json`,
  `Cargo.toml`, `Cargo.lock`).

## [1.135.9] - 2026-08 (Oxid Mail binary is now bundled as a Tauri sidecar on Linux/macOS - fixes "Oxid Mail not found" in deb/AppImage)

### Fixed - `oxid-mail` binary missing from Linux deb/AppImage installs

- Root cause: on Linux, Tauri places `bundle.resources` under
  `usr/lib/<app>/` while the main executable lives in `usr/bin/`. The
  launcher (`mail_launcher.rs`) only searched next to the exe, in
  `./oxid-mail/`, `OXID_MAIL_BIN` and `PATH` - so the bundled binary was
  never found. On Windows MSI resources land next to the exe, which is why
  it worked there. Resources also do not guarantee the executable bit.
- Fix (bundling): `tauri.mail.linux.conf.json` and
  `tauri.mail.macos.conf.json` now use `bundle.externalBin` (Tauri sidecar)
  instead of `bundle.resources`. Sidecars are installed next to the main
  binary (`usr/bin/oxid-mail` in deb/AppImage, `Contents/MacOS/` in .app)
  with the executable bit preserved - the existing "next to exe" lookup
  finds them without any env vars. Windows keeps `resources` (already
  working).
- Fix (build script): `scripts/build-oxid-mail.cjs` additionally copies the
  built binary to `src-tauri/oxid-mail-<host-triple>` (triple detected via
  `rustc -vV`) as required by `externalBin`, and `chmod 755`s both copies.
  The copies are git-ignored.
- Fix (launcher, defense in depth): `launch_oxid_mail` now also checks the
  Tauri resource dir (covers older builds that shipped the binary via
  `resources`) and restores a missing executable bit on Unix before
  spawning.
- No separate download is needed: the mail binary ships inside the deb and
  the AppImage.

### Chore

- Version bump `1.135.8` → `1.135.9` (`package.json`, `tauri.conf.json`,
  `Cargo.toml`, `Cargo.lock`).

## [1.135.8] - 2026-08 (First-launch UI language now follows the system locale instead of defaulting to Russian)

### Fixed - Russian UI on first launch on English systems

- Root cause: `src/i18n/index.ts` correctly detected the browser/system locale
  (`navigator.languages` → `ru`/`en`/`id`, fallback `en`), but the Pinia UI
  store (`src/store/ui.ts`) immediately overwrote it: both the initial state
  and `init()` fell back to a hard-coded `'ru'` whenever `localStorage` had no
  `sth_locale` key (i.e. on every first launch), then force-applied it via
  `applyLocale()`.
- Fix: `detectBrowserLocale()` is now exported from `src/i18n/index.ts` and
  used by the UI store as the fallback in both the initial state and
  `init()`. First launch on an English (or any non-ru/id) system now starts
  in English; Russian/Indonesian systems get their language; an explicitly
  chosen language in Settings is still persisted and always wins.

### Chore

- Version bump `1.135.7` → `1.135.8` (`package.json`, `tauri.conf.json`,
  `Cargo.toml`, `Cargo.lock`).

## [1.135.7] - 2026-08 (Absolute `frontendDist` injected by the build wrapper - final fix for `Unable to find your web assets` on Ubuntu)

### Fixed - Tauri CLI resolving `frontendDist: "../dist"` against the CWD instead of `src-tauri/`

- Root cause (confirmed by the user's Ubuntu log): even with `dist/` correctly
  built at `frontend/dist`, Tauri CLI resolved the relative `../dist` from the
  **current working directory** (`frontend/`), producing
  `/home/…/smartnet/dist` - one level too high - and aborting with
  `Unable to find your web assets`.
- Fix: `scripts/build-mail-bundle.cjs` now generates a temporary merge config
  `src-tauri/tauri.mail.gen.conf.json` at build time. It is the
  platform-specific mail config (`tauri.mail.conf.json` /
  `tauri.mail.linux.conf.json` / `tauri.mail.macos.conf.json`) plus an
  **absolute** `build.frontendDist` pointing at the freshly built
  `frontend/dist` (forward-slash normalized so it is valid JSON on Windows
  too). `tauri build` is invoked with `--config
  src-tauri/tauri.mail.gen.conf.json`, so path resolution no longer depends on
  the CWD, the OS, or Tauri's relative-path semantics.
- The generated config is a build artifact and is ignored by git.

### Chore

- Version bump `1.135.6` → `1.135.7` (`package.json`, `tauri.conf.json`,
  `Cargo.toml`, `Cargo.lock`).

## [1.135.6] - 2026-08 (Deterministic vite output + wrapper builds frontend explicitly - fixes `Unable to find your web assets` on Ubuntu)

### Fixed - `Unable to find your web assets, did you forget to build your web app?` on Linux/macOS

- Root cause: `vite.config.js` used `process.cwd()` for both the `@` alias
  and an implicit relative `build.outDir: 'dist'`. When Tauri ran
  `beforeBuildCommand: yarn build` from a working directory that wasn't the
  frontend folder (typical when `src-tauri/` sits next to `frontend/`
  instead of inside it), vite wrote `dist/` next to the wrong cwd and Tauri
  looked for it at a different absolute path (e.g.
  `/home/techno/projects/smartnet/dist`) → error.

### Changed - `frontend/vite.config.js`: absolute paths pinned to `__dirname`

- `root` explicitly set to `__dirname` (ESM `import.meta.url`).
- `resolve.alias['@']` now uses `path.resolve(__dirname, 'src')` - no more
  reliance on `process.cwd()`.
- `build.outDir` is now `path.resolve(__dirname, 'dist')` (absolute), plus
  `emptyOutDir: true` for clean rebuilds.  → vite always writes to
  `<frontend-dir>/dist`, no matter where you invoke `yarn build` from.

### Changed - `frontend/scripts/build-mail-bundle.cjs`

- The wrapper now runs `yarn build` itself (explicitly, from
  `path.resolve(__dirname, '..')` = the frontend directory) **before**
  invoking `tauri build`, and validates that `<frontend>/dist/index.html`
  actually exists - if not, aborts with a clear message pointing at
  `vite.config.js`.
- Merge configs
  (`tauri.mail.conf.json`, `tauri.mail.linux.conf.json`,
  `tauri.mail.macos.conf.json`) now include
  `"build": { "beforeBuildCommand": "" }` so Tauri does **not** re-run
  vite from an unpredictable cwd. This makes the frontend build
  deterministic across Windows / Ubuntu / macOS.
- New env vars honored by the wrapper:
  - `TAURI_BUNDLES` - override bundle formats (e.g. `deb`, `msi,nsis`)
  - `TAURI_EXTRA_ARGS` - extra CLI args passed to `tauri build`
  - `SKIP_FRONTEND=1` - skip the vite step (useful when `dist/` is already
    up to date, e.g. during CI incremental jobs)

### How to use (unchanged for the user)

```bash
yarn tauri:build:bonsai:mail
```

Produces `.msi` on Windows, `.deb + .AppImage` on Linux, `.dmg + .app` on
macOS - from the same command, on the same tree.

---

## [1.135.5] - 2026-08 (Cross-platform Oxid Mail bundle: Ubuntu/macOS/Windows share one build command)

### Fixed - `resource path 'oxid-mail.exe' doesn't exist` on Ubuntu / macOS

- **`frontend/src-tauri/tauri.mail.conf.json`** hard-coded a Windows-only
  resource path (`"resources": ["oxid-mail.exe"]`), so `yarn
  tauri:build:bonsai:mail` failed on Linux/macOS with
  `resource path 'oxid-mail.exe' doesn't exist` - the copied binary there is
  named `oxid-mail` (no `.exe` suffix).
- Added **platform-specific merge configs** next to the existing Windows one:
  - `frontend/src-tauri/tauri.mail.linux.conf.json` → `"resources": ["oxid-mail"]`
  - `frontend/src-tauri/tauri.mail.macos.conf.json` → `"resources": ["oxid-mail"]`
  - `frontend/src-tauri/tauri.mail.conf.json` stays Windows (msi) with
    `oxid-mail.exe`.

### Added - universal build wrapper `scripts/build-mail-bundle.cjs`

- New Node wrapper that:
  1. Runs the existing `scripts/build-oxid-mail.cjs` (compiles Oxid Mail,
     copies the binary - already cross-platform: `.exe` on Win, no suffix
     elsewhere).
  2. Detects `process.platform` and picks the matching merge-config
     (`tauri.mail.conf.json` / `tauri.mail.linux.conf.json` /
     `tauri.mail.macos.conf.json`) and the correct bundle formats
     (`msi` / `deb,appimage` / `dmg,app`).
  3. Invokes `tauri build --features p2p,bonsai --bundles <list> --config
     <cfg>` via `npx tauri` so it also works in npm-only environments.
  4. Honors optional `TAURI_BUNDLES` / `TAURI_EXTRA_ARGS` env vars for
     overrides.

### Changed - package.json scripts

- **`frontend/package.json`** - `tauri:build:bonsai:mail` now points to the
  universal wrapper, so the same command builds on Windows, Ubuntu and
  macOS with no extra flags:
  - `tauri:build:bonsai:mail` → `node scripts/build-mail-bundle.cjs`
- Explicit per-platform aliases were added for CI clarity:
  - `tauri:build:bonsai:mail:win`   (msi)
  - `tauri:build:bonsai:mail:linux` (deb + AppImage)
  - `tauri:build:bonsai:mail:mac`   (dmg + app)

### Usage

```bash
# One command, three OSes - no changes to CI scripts required:
yarn tauri:build:bonsai:mail
```

---

## [1.135.4] - 2026-08 (DISCOVER APPS_ card header height trimmed 64 → 56 px)

### Changed - `.disc-cover` (catalog card header)

- **`frontend/src/views/Discovered.vue`** - reduced the card header height
  from `64px` to `56px` (`.disc-cover`). The plate icon and title text
  continue to center-align inside, so the visual balance is preserved while
  the header no longer eats vertical space away from the 440×180 preview
  below.  Net effect: ~8 px shaved off every card in the DISCOVER APPS_
  grid (~4 px on top and ~4 px on the bottom of the header block).

---

## [1.135.3] - 2026-08 (MY_APPLICATIONS_ preview: strict 440×180 aspect, no vertical stretch)

### Fixed - owner-card preview thumbnails were stretching to match card height

- **`frontend/src/views/MyApplications.vue`** - previous release (1.135.2)
  aligned `.card-preview` with `align-self: stretch` and `min-height: 100%`,
  which forced the preview column to match the tallest sibling (`.card-body`).
  When cards had tags or extra metadata (BattleShip, SmartHoldem) the preview
  container grew taller than 440/180 and the `object-fit: cover` image was
  effectively enlarged, so previews looked inconsistent in the list.
- Switched `.app-card:has(> .card-preview)` to `align-items: start` and gave
  `.card-preview` a strict `align-self: start` + `width: 100%; aspect-ratio:
  440 / 180`, dropping `min-height: 100%`. Every preview is now the same
  visual size across the whole list - a strict 440×180 ratio that never
  spills outside the card - regardless of what the left column contains.

---

## [1.135.2] (MY_APPLICATIONS_ owner cards show uploaded preview on the right, 70/30 split)

### Added - 440×180 owner-preview thumbnail on every card in MY_APPLICATIONS_

- **`frontend/src/views/MyApplications.vue`** - imported `appPreview` from
  `@/lib/bridge` and added a reactive `previews` record parallel to the
  existing `logos` record. `refreshLogo()` and `loadLogos()` now also fetch
  the preview blob so the thumbnail refreshes instantly after Edit/Publish
  without a catalog re-scan.
- Both the **drafts** and **published** cards were split into a left
  `.card-body` (main content: head, tags, sth:// row, meta, actions) and a
  right `.card-preview` (`<img>`) that renders **only when the owner uploaded
  a preview**. If the preview is missing the card falls back to a single
  column - no empty rectangle.
- CSS uses `grid-template-columns: minmax(0, 7fr) minmax(0, 3fr)` via
  `.app-card:has(> .card-preview)` so the split is a strict ~70/30 without
  hardcoded pixel widths. Narrow viewports (`max-width: 900px`) collapse to a
  single column with the preview stacked below the content.

### Notes

- `:has()` is safe here because Tauri v2 bundles a recent Chromium/WebView2
  build; every supported target ships `:has()` support.
- The preview `<img>` is `object-fit: cover` so images uploaded through
  `PreviewCropModal.vue` (already enforced to 440×180) render pixel-perfect
  regardless of the card's exact width.

---

## [1.135.1]  (Discover header: hide-installed toggle, rounded SCAN pill, compact install button)

### Added - "Hide installed" toggle (localStorage-persisted)

- **`frontend/src/views/Discovered.vue`** - new `hideInstalled` reactive flag
  reads/writes `localStorage['ntfry.discover.hideInstalled']` (chose
  localStorage over sled: no extra Tauri round-trip, survives across reloads,
  and the flag is UI-only so there is no need to sync it across seeders).
  The catalog's `filtered` computed now drops apps with `installed === true`,
  **but keeps private dApps** even when they are installed - owners still need
  to see them in the catalog to launch/reinstall.
- Toggling the switch resets pagination to page 1 so the user immediately sees
  the top of the filtered catalog instead of an empty tail.
- **i18n** - added `discover.hideInstalled` and `discover.hideInstalledHint`
  in `en.ts`, `ru.ts`, `id.ts`.

### Changed - SCAN button now a rounded pill matching PRIVATE FOUND

- **`Discovered.vue`** - replaced the square `btn btn-primary` variant with a
  new `.scan-pill` class: pill-shaped (`border-radius: 999px`), inner
  highlight + drop-glow to visually pair with the golden PRIVATE FOUND badge
  next to it. Adds a subtle `scan-pill-pulse` animation while
  `discovery.scanning === true` and a spinning ⟳ glyph so users have live
  feedback that the mesh scan is actually running.

### Fixed - install button icons no longer wrap to a second line

- **`Discovered.vue`** - the action row (`.disc-actions-clean`) grid layout
  used three columns (`1fr auto 36px`) while the card actually renders four
  children: install, size badge, ⓘ details, copy-link square. The 4th child
  wrapped onto a new line and clipped underneath the card. Changed the grid
  to `minmax(0, auto) minmax(0, 1fr) 36px 36px`: install shrinks to the label
  width (with `padding: 8px 14px` and `white-space: nowrap`), size badge fills
  the middle and right-aligns, then two 36px squares for ⓘ and copy.  All four
  elements now sit on a single row regardless of card width.

### Notes

- The toggle is UI-only - it does not re-run `discovery.scan()`, so switching
  it on/off is instant and free of P2P traffic.
- Private dApps are intentionally excluded from the hide filter: their owner
  or an address on the access list must still see them in the catalog even
  after install.

---

## [1.135.0]  (Cloud seeders replicate discovery_cache media + auto-replication 30 → 60 min)

### Added - `/ntfryseed/discovery_cache` replication (cloud fallback for catalog artwork)

- **`ntfry/src/seeder.rs`** - `SEED_MEDIA_DIR = "discovery_cache"` + three
  provider uploaders `yandex/mailru/gdrive_upload_media_file(token, name,
  data)`: WebDAV providers MKCOL the folder idempotently and PUT the file;
  Google Drive resolves/creates the folder by id and upserts.
- **`cloudseeder.rs` / `replicate_discovery_media`** (runs inside
  `cloud_seeder_publish`): takes the **300 most recent** app folders from the
  local `data/discovery_cache` (sorted by mtime), uploads `logo-*` /
  `preview-*` files flat as `<appId>-logo-<hash>.<ext>`. Upload happens
  **only on change**: names are content-addressed, and an uploaded-set per
  provider×role is persisted in sled (`cfg:seeder_media_uploaded:*`) - an
  unchanged file is never re-sent. 256 KB per-file cap enforced. The publish
  syslog line now includes `+N media`.
- Purpose: when a client has NO live seeds/peers and only the cloud, catalog
  artwork can be restored from `/ntfryseed/discovery_cache/` (client-side
  consumption of this fallback is a separate next step).

### Changed

- Auto-replication interval **30 → 60 min** (`lib/seederRep.ts`
  `REP_INTERVAL_MS`), label updated in EN/RU/ID (`seeders.autorepLabel`).

---

## [1.134.1]  (Dashboard "Popular on the network": full-height preview, no vertical crop)

### Changed

- `views/Dashboard.vue` - `.top-app-cover` fixed `height: 96px` replaced with
  `aspect-ratio: 440 / 180`: the cover now matches the DAPP Preview's native
  proportions, so the whole image is visible without vertical cropping and
  keeps scaling with the card width (`object-fit: cover` becomes an exact
  fit at the matching aspect). Gradient/identicon fallback unaffected.

---

## [1.134.0]  (DISCOVER APPS card redesign: hover description strip, no OPEN button, icon-only details)

### Changed - `views/Discovered.vue` (per user's spec + POKER mockup)

- **Description** is hidden from the card body; it slides up from the bottom
  of the preview area on hover (`.disc-desc-strip`: dark header-like glass
  background, white text, `translateY` transition). Rendered only when a
  description exists.
- **OPEN button removed entirely** - the app opens by clicking the title bar
  or the preview (both already wired). Non-installed apps keep INSTALL.
- **Tags removed from the card in ALL modes** (still available in the
  Details modal).
- **DEV/GEEK modes**: the text "DETAILS" link is replaced by the same ⓘ
  square icon as USER mode (single unconditional button).
- **dApp size** (`⛁ N MB`) moved onto the action row, one line before the ⓘ
  details icon (`.disc-actions-clean` switched from grid to flex; details
  icon is pushed right via `margin-left: auto`).
- **"via seeder" badge** moved onto the preview area - bottom-right corner,
  the same glass pill styling as the seeds/status pill top-right
  (`.disc-seeder-overlay`).

---

## [1.133.5]  (Fix: installed apps on OTHER clients showed no preview - P2P fallback added)

### Root cause

Client 2 had XBTS SITE / SmartHoldem WEB 4.0 **installed** - and for
installed apps the card only read the LOCAL `<id>-preview.<ext>` file. Their
local copies were installed BEFORE the publisher attached previews (a FREE
metadata EDIT does not bump the version, so no update is triggered) → no
local file → gradient. The announce carried `previewHash`, but the installed
branch never used it.

### Fixed - cascade: local file → P2P blob from the announce → gradient

- `views/Discovered.vue` + `views/Dashboard.vue`: for INSTALLED apps, when
  `app_preview` / `app_logo` returns null and the announce has
  `previewHash` / `logoHash`, the card now lazily fetches the blob over P2P
  (`discovery_preview` / `discovery_logo`, cached in `discovery_cache`) -
  identical to the non-installed path. Old installs get artwork without
  re-installing; a future app update still brings the file locally (it is
  part of the payload tar since the publisher re-seeded).

---

## [1.133.4]  (Full session audit after silent write failures - one lost edit found and restored)

### Audit scope (every touch-point of v1.130.0 → v1.133.3 re-verified byte-by-byte)

- **p2p.rs** (18 patterns): constants, `jpeg_is_arithmetic`, both SEEDED maps,
  `l{i}`/`p{i}` announce + parse, `fetch_logo` (impl + stub),
  `discovery_logo/preview/media`, `media_data_url`, `prune_discovery_cache`,
  GC keep-set ×2, struct fields ×3, tracker/seeder mappings - ALL PRESENT.
- **apps.rs**: `preview_ext` (+arithmetic), `read_preview_bytes`, `log_once`,
  `app_preview`, both preview writes, `preserved_preview`, and ALL 6
  `ingest_app` signature/call sites verified programmatically at 16 args each.
- **lib.rs**: 4 command registrations. **syslog.rs**: `ui_log`.
- **bridge.ts / apps.ts / discovery.ts**: all wrappers, params, fields.
- **Discovered.vue / Dashboard.vue**: parallel loaders, preview blocks,
  overlays, fallback chip, diagnostics.
- **MyApplications.vue**: pickers, crop wiring, `missingMeta`, EDIT flow.
- **i18n**: 14/14/14 keys in EN/RU/ID. **PreviewCropModal.vue**: z-index 300.
- **tracker**: 7 touch-points (re-verified in 1.133.2).

### Fixed (the one lost edit)

- `views/MyApplications.vue` / `resetForm()` - did not clear
  `previewBytes`/`previewName` (the 1.131.0 edit had silently failed): after
  creating a draft, the previous app's preview stayed selected in the publish
  form. Restored.

---

## [1.133.3]  (Fix: crop modal never rendered - missing template tag + z-index below the EDIT modal)

### Fixed

- `views/MyApplications.vue` - the `<PreviewCropModal>` template tag was
  missing (the 1.133.0 edit did not persist; only the import existed), so the
  crop modal never rendered for either the publish form or the EDIT modal.
  Tag restored after `<AboutModal>`.
- `components/PreviewCropModal.vue` - overlay `z-index` raised 200 → 300 so
  the crop opens ON TOP of the EDIT modal (`.modal-back` is 200).

---

## [1.133.2]  (Fix: tracker v1.4.0 compile error - AppRecord initializer missing logo/preview)

### Fixed

- `/app/tracker/src/main.rs` - the `or_insert_with(|| AppRecord { ... })`
  initializer in the announce handler was missing the new `logo`/`preview`
  fields (E0063; the 1.133.0 edit did not persist). Added
  `logo: a.logo.clone()`, `preview: a.preview.clone()`. All 7 tracker
  touch-points re-verified byte-by-byte in the file: 3 structs, the record
  initializer, the refresh block, `app_to_out`, and both test fixtures.
- Client binary unchanged (version bumped for iteration tracking only).

---

## [1.133.1]  (Dashboard "Popular on the network": logo chip only as a fallback)

### Changed

- `views/Dashboard.vue` - the glass logo/identicon chip on the top-3 covers
  is now rendered ONLY when the card has no DAPP Preview image
  (`v-if="!topPreviews[a.id]"`). With a preview present, the artwork is shown
  clean, without the chip overlapping it.

---

## [1.133.0]  (Tracker passthrough of logo/preview hashes + in-app 440×180 crop tool)

### Added - Phase 2: tracker passthrough (`/app/tracker` v1.4.0)

- `AppAnnounce` / `AppRecord` / `AppOut` gained pass-through `logo` and
  `preview` iroh-hash fields (`#[serde(default)]` in, `skip_serializing_if
  empty` out - NOT part of the ECDSA signature, so legacy announces/records
  keep verifying). The registry refreshes the hashes from the latest
  non-empty announce (content-addressed → a changed image = a new hash).
- The client already sends (`TrackerAnnounceApp`, since 1.130/1.131) and
  parses (`TrackerApp`) these fields - **no client changes needed**: catalog
  images now reach peers that discover an app via the tracker only (the blob
  itself is still fetched over iroh from the announced providers).
- Tracker test fixtures updated. Deploy: rebuild the tracker binary.

### Added - in-app preview crop (client v1.133.0)

- New `components/PreviewCropModal.vue`: pick ANY image (`accept="image/*"`)
  in the publish form or the EDIT modal → a modal with a live 440×180 canvas
  frame: drag to position, zoom via slider or mouse wheel (center-anchored,
  cover-clamped). APPLY exports the exact 440×180 frame via canvas -
  PNG first, falling back to JPEG q0.9 / q0.72 if over the 256 KB network
  limit. Canvas output is always browser-generated (Huffman) - the
  arithmetic-JPEG class of bugs can't re-enter through this path.
- The old strict 440×180 file validation in `MyApplications.vue` is replaced
  by the crop flow (`onPreviewPick` / `onEditPreviewPick` → modal →
  `onCropApply`); backend byte-level guards from 1.132.5 remain as the last
  line of defense.
- i18n: `apps.crop*` strings (EN/RU/ID).

---

## [1.132.5]  (Root cause CLOSED: arithmetic-coded JPEG previews are now rejected at every layer)

### Root cause (confirmed by the user)

The XBTS preview JPEG was compressed with **arithmetic coding** (SOF9-SOF11
markers). Chromium/WebView2 only decodes Huffman JPEGs - the image passed all
magic-byte checks (`FF D8 FF`), the backend served a valid data URL, but the
`<img>` silently failed to decode → "empty" card. Re-saving as PNG fixed it.

### Fixed - arithmetic JPEG detection (`jpeg_is_arithmetic`, JPEG segment walk)

- **`p2p.rs`** - shared `jpeg_is_arithmetic()` (walks JPEG header segments up
  to SOS, flags SOF9/SOF10/SOF11). `discovery_media` now rejects such blobs
  from peers (logged as "blob is not PNG/JPG (or arithmetic JPEG)").
- **`apps.rs` / `preview_ext`** - arithmetic JPEGs are never written to
  `<id>-preview.jpg` at publish/edit time, so they can't enter the network.
- **`MyApplications.vue`** - the picker scans the bytes and shows a clear
  message: "This JPEG uses arithmetic coding - browsers cannot display it.
  Re-save as PNG or standard JPEG" (i18n EN/RU/ID,
  `apps.previewArith`) instead of the misleading 440×180 error.

### Kept

- The 1.132.3/1.132.4 SYSLOG diagnostics (deduped one-shots) stay in place -
  they proved the exact failure layer and remain useful for support.

---

## [1.132.4]  (Diagnostics round 2: frontend-side preview logging - `ui_log` command)

### Context

The 1.132.3 SYSLOG showed `app_preview STsks… - OK (79423 B)` - the backend
returns the data URL, yet the card still shows no image. A standalone
reproduction of the exact card DOM/CSS with a real JPEG data URL rendered
correctly in Chromium, so the remaining unknown is the frontend data flow /
image decode inside the user's WebView2.

### Added

- **`syslog.rs`** - new `ui_log(scope, msg)` command: the frontend can write
  into the system journal (no DevTools needed in production builds).
- **`views/Discovered.vue`** - one-shot (deduped) diagnostic entries:
  - `preview <id>… - invoke returned N chars. / null` - what the Vue layer
    actually received from `app_preview`;
  - `preview <id>… - <img> decoded and painted` - the `@load` event
    fired (image decoded and painted);
  - `preview <id>… - DECODING ERROR <img>` - the `@error` event fired
    (bad data URL / unsupported JPEG variant / CSP block).
- `lib/bridge.ts` - `uiLog()` fire-and-forget wrapper.

### Expected diagnosis matrix

| Journal lines | Meaning |
|---|---|
| no `invoke returned` line | Vue branch never ran (installed flag / marker) |
| `invoke returned null` | IPC returned nothing despite backend OK (serialization) |
| `returned N chars.` + `DECODING ERROR` | WebView2 refused the JPEG (decode/CSP) |
| `returned N chars.` + `decoded and painted` | image painted - CSS/layout issue |

---

## [1.132.3]  (Diagnostics: preview/logo pipeline logs in the system journal - no DevTools needed)

### Added

- **`apps.rs` / `app_preview`** - every outcome is now written to the SYSLOG
  (`dapp` scope) with per-outcome dedup (no spam on the 4s catalog poll):
  - `app_preview <id>… - OK: <file> (<N> B)` - file found and returned;
  - `app_preview <id>… - file not found: <full path>` - read miss (shows the
    exact directory the backend checked);
  - `app_preview <id>… - private dApp, skip`.
    If NO `app_preview` lines appear at all while the catalog is open, the
    frontend never reached the command (invoke/registration issue) - that is
    itself the diagnosis.
- **`p2p.rs` / `discovery_media`** (logo + preview of non-installed apps) -
  logs invalid announce hash, fetch failure (with provider count and the
  transport error), unreadable blob, bad-format / over-limit rejection, and
  the success path (`downloaded and cached (<N> B, <ext>)`).

### How to use

Rebuild, open // DISCOVER APPS_, wait a few seconds, then open the system
journal (SYSLOG) and filter by `app_preview` / `discovery_`. Send the lines -
they pinpoint whether the failure is IPC, path, file read, or P2P fetch.

---

## [1.132.2]  (Fix: preview not showing on the developer's own card - blocking sequential loads)

### Root cause

`loadLogos()` in `Discovered.vue` awaited every card **sequentially**: the
preview loop only started after ALL logo fetches finished, and each
non-installed app with dead P2P providers blocks up to ~10-15 s inside
`download_complete_any` retries. With several offline apps in the catalog the
previews could take minutes to appear - perceived as "preview never shows".
On top of that, installed apps got a permanent `''` marker after the first
failed attempt, so a preview added later via EDIT never retried within the
session.

### Fixed

- **`views/Discovered.vue`** - `loadLogos()` now fires ALL logo/preview loads
  **in parallel** (`Promise.all` of independent jobs): local disk reads for
  installed apps resolve instantly and never wait for slow P2P fetches of
  offline peers.
- Installed apps no longer get a permanent in-flight marker: cheap local reads
  retry on every discovery poll, so a preview/logo added via EDIT appears live
  without restarting or re-opening the page.
- Defensive `(a.providers || [])` guard for apps hydrated from an older
  localStorage catalog cache.
- **`views/Dashboard.vue`** - the same non-blocking treatment for
  `topLogos` / `topPreviews` on the "Popular on the network" cards.

### Verified backend paths (diagnosis with the user)

`<id>-preview.jpg` was confirmed present on disk in `data/dapps/<id>/` and
`app_preview` reads exactly that path (identical pattern to the working
`app_logo`) - the backend write/read chain was correct; the failure was the
frontend's blocking load order + stale marker.

---

## [1.132.1]  (Catalog card layout: preview block under the header, status icons on the preview)

### Changed - `views/Discovered.vue` (per user's mockup)

- Card structure is now: **header bar** (logo chip + title + [vN], gradient)
  → **DAPP Preview block** (full-bleed, `aspect-ratio: 440/180`; the 440×180
  image when available, the deterministic gradient as fallback; click =
  open/details) → description → the existing action strip (full-width
  OPEN → / INSTALL + ⓘ details + copy-link squares).
- The status badges (official ✔, private 🔒, installed dot, online dot /
  seeds counter) moved from their own row **onto the preview block** - a
  glass pill (`.disc-top-overlay`, blur + translucent dark) pinned to the
  top-right corner, matching the mockup.
- The preview image no longer lives inside the 64px header cover; the header
  keeps only the icon plate + title. `.disc-cover` bottom margin adjusted so
  the preview block sits flush under the header.

---

## [1.132.0]  (Cards render DAPP Preview instead of the gradient - gradient stays as fallback)

### Added

- **`src-tauri/src/apps.rs`** - new command `app_preview(id)`: returns the
  local `<id>-preview.png|jpg` of an installed dApp as a base64 data URL
  (correct `image/png` / `image/jpeg` mime); `None` for private dApps and
  apps without a preview (the card keeps the gradient).
- **`views/Discovered.vue`** - the catalog card cover (`.disc-cover`) now
  renders the 440×180 DAPP Preview as an absolutely-positioned
  `object-fit: cover` image on top of the deterministic gradient
  (`coverStyle` stays as the fallback; the dark scrim `::after` and the
  title/logo plate remain above the image for readability). Installed apps
  read the preview from disk (`app_preview`); non-installed apps lazily
  fetch it over P2P via `discovery_preview` (announced `previewHash` +
  `discovery_cache`), mirroring the logo pipeline. `overflow: hidden` added
  to `.disc-cover` so the image clips to the rounded top corners.
- **`views/Dashboard.vue`** - the three "Popular on the network" covers
  (`.top-app-cover`) use the same preview-image-over-gradient treatment
  (`topPreviews` map next to `topLogos`); the glass icon chip (z-index 1)
  stays on top of the preview.
- `lib/bridge.ts` - new `appPreview()` wrapper.

### Notes

- Fallback chain per card: DAPP Preview image → deterministic tinted
  gradient. Logo chip / identicon behavior unchanged.

---

## [1.131.0]  (Mandatory DAPP Preview 440×180 on publish + P2P distribution like logo)

### Added - DAPP Preview (publish form + Edit modal)

- **`views/MyApplications.vue`** - new mandatory "PICK PREVIEW" control to the
  right of the logo picker in the publish form: JPG/PNG only, **max 256 KB**,
  resolution validated strictly at **440×180 px** (via `Image.naturalWidth/
  Height`); draft creation is blocked (`missingMeta`) until a preview is
  chosen. Hidden for **private dApps** (their visual identity is never stored
  or announced). The EDIT modal got the same optional picker so already
  published apps can attach a preview through the FREE metadata edit.
- **`apps.rs`** - `create_draft_app` / `edit_app_metadata` (+ `ingest_app`)
  accept `preview: Option<Vec<u8>>`; the file is written next to the payload
  as **`<id>-preview.<png|jpg>`** (presentation metadata like the logo - not
  part of the signed merkle at ingest) and is preserved across re-ingest
  (finalize/preview/update) via `read_preview_bytes`. Backend stays lenient
  (Option) so updates of legacy apps without a preview keep working; magic
  bytes (PNG/JPEG) + 256 KB limit enforced before writing.

### Added - network distribution (same pipeline as the 1.130.0 logo blobs)

- `seed_all` seeds `<id>-preview.png|jpg` as a standalone iroh blob →
  `SEEDED_PREVIEWS` (GC-protected); private dApps skipped.
- mDNS announce: new per-app TXT record **`p{i}`** with the preview blob hash
  (mirrors `o{i}`/`l{i}`; old clients ignore it).
- `TrackerAnnounceApp.preview` / `TrackerApp.preview` plumbed through
  (skip-if-empty, NOT part of the ECDSA signature - Phase 2 groundwork).
- New Tauri command **`discovery_preview(id, previewHash, nodeIds)`**; the
  logo path was refactored into a shared `discovery_media` pipeline: cache
  `data/discovery_cache/<appId>/<logo|preview>-<hash>.<png|jpg>`, tmp-file
  download, magic-byte + size validation (JPEG allowed for previews only),
  per-app stale-file eviction, 300-folder LRU prune.
- Frontend groundwork for the next stage (cards showing previews instead of
  gradients): `DiscoveredApp.previewHash` aggregated in `store/discovery.ts`,
  `discoveryPreview()` wrapper in `lib/bridge.ts`.
- i18n: new `apps.preview*` strings in EN/RU/ID.

### Out of scope (next stage, by user's request)

- Dashboard "Popular on the network" + catalog cards rendering the 440×180
  preview image instead of the generated gradient (gradient stays as the
  fallback).

---

## [1.130.1]  (Fix: 1.130.0 compile errors - missing logo constants + export() return type)

### Fixed

- `src-tauri/src/p2p.rs` - the `LOGO_MAX_BYTES` / `PNG_MAGIC` module constants
  referenced by `seed_all` and `discovery_logo` were missing from the file
  (E0425 ×4), breaking the 1.130.0 build. Constants are now defined at module
  top: `LOGO_MAX_BYTES = 256 * 1024`, `PNG_MAGIC = [0x89, 'P', 'N', 'G',
  0x0D, 0x0A, 0x1A, 0x0A]`.
- `fetch_logo` - `store.blobs().export()` returns `Result<u64, _>` (bytes
  written), not `Result<(), _>` (E0308); mapped with `.map(|_| ())`.

---

## [1.130.0]  (Catalog logos for non-installed dApps - P2P logo blobs + discovery cache)

### Added - PNG logo distribution for non-installed apps

Until now a dApp's icon only appeared in the catalog after installation (the
logo file lives inside the app folder). Non-installed apps always fell back to
the identicon. The publisher's logo is now distributed over P2P as a separate
content-addressed iroh blob:

- **Publisher/seeder side** (`src-tauri/src/p2p.rs`, `seed_all`):
  - For every seeded dApp, `<id>-logo.png` / `logo.png` / `icon.png` (PNG
    magic-bytes validated, **≤ 256 KB**) is imported into the iroh store as a
    small standalone blob. Hashes live in the new `SEEDED_LOGOS` map and are
    protected from blob-store GC alongside app tars.
  - **Private dApps are skipped** - their visual identity is never announced.
  - mDNS announce: new per-app TXT record `l{i}` carries the logo blob hash
    (mirrors the `o{i}` owner-record pattern; `a{i}` format untouched, old
    clients ignore the unknown key, TXT size stays safe - a hash is ~52 chars).
  - `TrackerAnnounceApp` gained an optional `logo` field
    (`skip_serializing_if empty`, NOT part of the ECDSA signature) - current
    trackers silently ignore it; groundwork for Phase 2 tracker passthrough.
- **Client side**:
  - New Tauri command `discovery_logo(id, logoHash, nodeIds)`:
    1. validates the hash (alphanumeric-only - path-traversal safe) and the id
       (`sanitize`);
    2. checks the content-addressed cache
       **`data/discovery_cache/<appId>/logo-<hash>.png`** (per-app folder so
       future discovery caches can live next to it) - instant data-URL hit;
    3. on miss, downloads the blob from the given live P2P providers
       (`download_complete_any` race), enforces the 256 KB / PNG-magic limits
       (bad blob → deleted + rejected), evicts older `logo-*.png` files of the
       same app (a changed logo means a new hash - automatic version
       invalidation), prunes the cache root to 300 app folders (oldest-mtime
       first).
  - `PeerApp.logo` / `TrackerApp.logo` plumbed through `merge_tracker_apps`
    and the mDNS resolver.
- **Frontend**:
  - `store/discovery.ts` - `DiscoveredApp.logoHash` aggregated from announces
    (first non-empty; private apps excluded); old localStorage catalog cache
    normalized.
  - `views/Discovered.vue` - non-installed cards (and the details modal) now
    lazily fetch the announced logo via `discoveryLogo()`; identicon remains
    the fallback. Retries once providers appear if the first attempt ran
    provider-less.
  - `views/Dashboard.vue` - the "Popular on the network" top-3 cards use the
    same lazy P2P logo path for non-installed apps.
  - `lib/bridge.ts` - `PeerApp.logo?`, new `discoveryLogo()` wrapper.

### Out of scope (Phase 2, by design)

- Tracker passthrough of the `logo` field (server change).
- Cloud Seeder fallback thumbnails (clients with zero live P2P keep the
  identicon).
- SVG logos over the network (PNG only - untrusted-peer safety).

---

## [1.129.9] - 2026-07 (Dashboard: "Popular on the network" cards fill the full column width)

### Changed

- `views/Dashboard.vue` - removed `max-width: 780px` from `.top-apps-row`.
  The three grid columns (`repeat(3, minmax(0, 1fr))`) now stretch to fill
  the entire welcome block width beneath the search bar, matching the width
  of the search input and the section title. Card content stays perfectly
  proportional because the grid tracks are still `1fr` each.

---

## [1.129.8] - 2026-07 (Fix: "Invalid Date" in tab install-source tooltip)

### Fixed

- `layouts/AppLayout.vue` - tab tooltip like `Installed directly via P2P
  (iroh-blobs) · v1 · Invalid Date` was caused by the frontend calling
  `new Date(s.at).toLocaleString()` on a value that the Rust backend actually
  writes as a Unix-seconds string (`crypto::now_iso()` - despite its name).
  New `formatInstalledAt(raw)` helper accepts:
  - number,
  - numeric string in **seconds** (`> 10¹²` → ms, otherwise ×1000),
  - or a real ISO/RFC-2822 date string,
    and returns `''` on unparseable input (so the ` · <date>` part is
    simply omitted instead of showing `Invalid Date`).
- Fix applies to both `"kind": "p2p"` (from `p2p.rs`) and
  `"kind": "seeder"` (from `cloudseeder.rs`) - both branches write the
  same Unix-seconds string via `crypto::now_iso()`, so no Rust changes needed.


---

## [1.129.7] - 2026-07 (Remove Neumorphic Light theme + sidebar active-pill cleanup)

### Removed - Neumorphic Light theme

- Theme `neumo` is no longer available in Settings → Interface → APPEARANCE.
  All CSS rules keyed on `[data-theme='neumo']` were deleted from `styles/base.css`,
  `styles/themes.css`, `views/Settings.vue`, `views/ChatWidget.vue`, and
  `components/SeedingCard.vue`, along with the `neumo` swatch and i18n label
  (`themes.neumo`) in `en/ru/id`.
- `store/ui.ts` - `'neumo'` dropped from `ThemeId` union and `THEME_CYCLE`.
  Users whose saved theme was `neumo` are transparently migrated to
  `default-light` on next launch (LocalStorage rewritten in `init()`).

### Fixed - Sidebar active-item pill in default themes

- `styles/base.css` - `html[data-theme^='default-'] .nav-item.active`
  no longer paints `var(--accent-soft)` behind the label. Only the text
  colour switches to `var(--accent)`, giving a clean "text-highlight"
  active state in both `default-dark` and `default-light`. The 3px left
  indicator from `AppLayout.vue` (`::before`) still marks the current
  route.

---

## [1.129.6] - 2026-07 (Sidebar: cleaner active-item look)

### Changed

- `layouts/AppLayout.vue` - the active `.nav-item` no longer paints a rounded
  `var(--panel-2)` pill behind the label. Only the text highlight
  (`color: var(--text)`) and the accent left indicator (`::before`,
  `var(--secondary)`) remain. Applies to both default-dark and default-light
  themes. Hover state is untouched - it still shows a subtle pill on
  non-active items to give a clear pointer target.

---

## [1.129.5] - 2026-07 (DISCOVER APPS: no hover underline on card title)

### Changed

- `views/Discovered.vue` - removed the `text-decoration: underline` (with
  its thickness / underline-offset) that appeared on `.disc-cover-title`
  when hovering a discover card. The whole card is already a link - an
  extra underline on the title read as visual noise.

---

## [1.129.4] - 2026-07 (Dashboard: deep-tint gradient for "Popular on the network" cards)

### Changed - cover-art palette

- `src/lib/coverArt.ts` - the deterministic hue is still derived from the app id,
  but saturation/lightness are dropped (`28%/20%` → `20%/12%` → near-black `#0d1113`)
  and the second stop is a tiny hue shift (`+24°` instead of `+46°`). Cards remain
  visually distinct but now live inside NETFORY's dark palette instead of the old
  neon teal/orange combo that clashed with the UI.
- `views/Dashboard.vue` (`.top-app-cover`) - added `border: 1px solid var(--border)`
  and `border-radius: 10px 10px 0 0` so the cover stitches into the card frame.
- Softer highlight overlay (`rgba(255,255,255,0.22) → 0.08`) and slightly deeper
  bottom vignette (`0.28 → 0.35`).
- Hover shine sweep dampened from `0.32` peak alpha to `0.14` so the animation is
  visible without "glassing" on a dark tile.

---

## [1.129.3] - 2026-07 (Rebrand: SmartNet → NETFORY user-facing name + Russian dApps guide)

### Changed - user-facing product name is now **NETFORY**

- `frontend/index.html` - window `<title>` set to `NETFORY // SmartHoldem WEB4 P2P Network`.
- `frontend/package.json` - package `name` renamed to `netfory-client`.
- `frontend/src-tauri/tauri.conf.json` - `productName` is now `NETFORY`; window title updated.
- `frontend/src-tauri/Cargo.toml` - package description now reads `NETFORY - SmartHoldem WEB4 P2P network`.
- Chat widget title (`views/ChatWidget.vue`) and i18n messenger titles (`en/ru/id`) updated
  to `NETFORY // P2P MESSENGER` / `NETFORY // P2P MESSENGER` / `NETFORY // PESAN P2P`.

  **Backwards compatibility preserved on purpose:**
  - `smartnet_lib` / `smartnet-client` Rust package names - unchanged.
  - Bundle identifier `io.smartholdem.smartnet` - unchanged (existing installs keep their profiles/data).

### Added - icon set regenerated

- Full desktop icon set (`icon.icns`, `icon.ico`, `32/128/128@2x`, `icon.png`), Windows Store logos,
  iOS AppIcon-* set and Android mipmap-{m,h,x,xx,xxx}hdpi with round + adaptive foreground.

### Added - Locale bootstrap: browser-language auto-detection

- `src/i18n/index.ts` - on first launch, when no `sth_locale` is stored, the app resolves the
  initial locale from `navigator.languages` / `navigator.language`, matching against
  supported set `{ru, en, id}`; falls back to `en` when no match. Stored user choice always wins.

### Added - Translations for TORRENTS page

- New `torrents.*` i18n namespace (`en/ru/id`) covering:
  page subtitle (with new wording *"…seed in the NETFORY"*), `Create torrent`,
  `ADD` button, `Total download / upload / downloaded / uploaded` stats, and
  table headers `Name / Size / Progress / Status / Speed`.
- `views/Torrents.vue` - imports `useI18n` and reads all those strings through `t(...)`;
  `Seeds/Leechers` kept as a language-agnostic BitTorrent term.

### Added - Translations for SETTINGS → NETWORK (up to Network Doctor)

- New `updates.*` i18n namespace (`en/ru/id`) with `title`, `hintPrefix`, `hintSuffix`,
  `checkNow`, `checking`, `foundVersion({version})`, `checkError({error})`, `upToDate`.
- `views/Settings.vue` - "P2P UPDATES" section header, hint text, action button
  and `manualUpdateCheck()` toast strings now go through `t('updates.*')` instead of hardcoded RU.

### Fixed - Duplicate i18n keys `music.paid`

- Each of `en/ru/id` had two `music.paid` entries (short badge label vs. purchase toast),
  which caused Vite/esbuild `Duplicate key "paid"` compile errors. Removed the unused short
  label; the buy-flow toast retains `music.paid` and continues to work in `AlbumView.vue`.

### Docs

- New Russian guide: [`docs/16-dApps-Guide-RU.md`](./docs/16-dApps-Guide-RU.md) -
  end-to-end walkthrough of publishing dApps (Public / Private / Direct-Link),
  `sth://appId` routing in third-party browsers, repository views (Official / All / Private),
  paying and renewing domain names, editing/updating dApps (free & paid), Boost mechanics,
  where to view stats and how to donate after opening a dApp - with a closing "why this rocks"
  section on why it works without any servers.

---

## [1.129.2] - 2026-07 (Duplicate i18n key fix)

### Fixed

- Removed a duplicate `music.paid` key in `en.ts`, `ru.ts`, and `id.ts` that made
  Vite's esbuild refuse the module (`Duplicate key "paid" in object literal`).
  The purchase-toast semantics were kept; the short badge label was removed
  because no template referenced it.

---

## [1.129.1]  (Fix: Direct Link / QR for FREE "link-only" dApps)

### Fixed - Direct Link generation for free (unlisted) apps
- **`app_share_info` no longer depends on the tracker snapshot**
  (`frontend/src-tauri/src/p2p.rs`): the command previously looked up the
  seeded blob hash in `TRACKER_SNAPSHOT`, which only contains apps written to
  the `announce:` index - i.e. **paid published** dApps. Free "link-only"
  apps are seeded by the node just the same (`seed_all` archives every dApp
  in the `dapps/` folder), but were absent from the snapshot, so pressing
  **⚡ DIRECT LINK** or **▦ QR** on a free app failed with a bogus
  *"not seeding yet"* error and nothing was copied.
- **New `SEEDED_APPS` registry** (`p2p.rs`): a global `id → tar-blob-hash`
  map covering *everything* this node actually seeds - free link-only apps
  included. `seed_all` refreshes it on every re-seed pass (startup discovery,
  publish, delete/reclaim, blob-store optimize), so the map never goes stale.
- `app_share_info` now resolves the hash from `SEEDED_APPS` and the node id
  from `OWN_IDENTITY`, falling back to the old tracker-snapshot path only if
  the seeding pass has not completed yet in the current session. Direct
  `sth://<id>?n=<nodeId>&h=<hash>` links and their QR codes now work for
  both paid and free dApps.

### Fixed - gold $-coin badge missing on ALL installed-app cards
- **`hasEarnings` was too strict** (`frontend/src/views/Installed.vue`): the
  coin was only shown when the *user's own* accrual (`estSth`) or current-day
  forecast (`forecastSth`) was positive, so apps with a live payout pool but
  no personal accrual lost the badge entirely. The check now also accepts an
  active funding bank (`bankSth > 0`) - the coin is hidden only when every
  stat is genuinely zero, as originally intended.
- **Race with async wallet load**: `earnings.refresh()` silently no-ops when
  `auth.address` is still empty at page mount, leaving the earnings snapshot
  empty (and every coin hidden) until a manual reload. A `watch` on
  `auth.address` now re-triggers the refresh as soon as the wallet address
  becomes available.

---

## [1.129.0] - 2026-07 (Trackerless dApp sharing: `sth://<id>?n=<node>&h=<hash>` direct links + one-tap QR modal)

### Added - Direct Link sharing (bypass trackers & seeders)
- **Self-contained install URI** (`frontend/src/views/Resolve.vue`): the
  `sth://<id>` scheme now accepts two optional query parameters,
  `?n=<endpointId>&h=<blobHash>`, which together carry everything the
  receiving client needs to install a dApp without touching any tracker,
  cloud seeder or gossip peer. When both parameters are present,
  `Resolve.vue` skips the tracker lookup entirely, pre-populates the
  `direct` ref with `{ providers: [n], hash: h }` and calls
  `discovery.installDirect(id, providers, hash)` straight away - the
  P2P blob is pulled directly from the author's node over Iroh. If the
  tracker lookup happens to succeed in parallel it is still honoured as a
  richer source of metadata, but the direct link is always the
  authoritative fallback so the install never gets stuck on an empty
  swarm.
- **New "⚡ Direct link" copy button on every published dApp card**
  (`frontend/src/views/MyApplications.vue`): for each entry in the PUBLISH
  tab the owner can now generate a ready-to-share
  `sth://<id>?n=<nodeId>&h=<hash>` URL with a single click. The link is
  built from the local `endpointId` (author's Iroh NodeId) and the
  content-addressed `hash` of the current tar blob, so it is stable
  across restarts and does not require the recipient to trust any
  intermediate infrastructure. Success/error feedback is shown inline on
  the button (`copiedDirect` / `directLinkErr` state), with a 2.6 s
  auto-clear.
- **QR-code modal for the direct link** (`frontend/src/components/FileQrModal.vue`,
  `frontend/src/views/MyApplications.vue`): a new "▦ QR" button next to
  the direct-link copy button opens a themed modal that renders the same
  `sth://<id>?n=<node>&h=<hash>` URL as a scannable QR code (via the
  existing `qrcode` package used by Oxid Mail contact sharing). Users can
  now hand over a dApp in person - the phone camera reads the QR, the
  SmartNet mobile companion opens the URI scheme, and the P2P install
  begins over Iroh without a tracker round-trip. The modal is generic:
  it accepts arbitrary `link` / `name` / `title` / `hint` props, so the
  same component now serves both the private-file share flow and the
  new direct-link flow.
- **Translated new UI strings** in `frontend/src/i18n/{ru,en,id}.ts`:
  `apps.directLink`, `apps.directLinkHint`, `apps.directLinkErr`,
  `apps.qrBtnHint`, `apps.qrTitle`, `apps.qrHint`, `resolve.directBadge`,
  `resolve.directNote`. Russian / English / Indonesian copy provided,
  wired via `data-testid="qr-direct-<id>"` for automation.

### Rationale
- Public trackers and cloud seeders are the "yellow pages" of the
  network - great for discovery, but useless when the author just wants
  to hand a link to a friend or paste it into a private chat. The direct
  link is the SmartNet-equivalent of an IPFS `ipfs://<cid>#<providers>`
  URL: it is a self-contained pointer to a specific piece of content on
  a specific node, verifiable end-to-end by the client. No third-party
  service is involved, and the URL degrades gracefully - a QR on a
  business card still installs the dApp a year later, as long as the
  author's node is online.
- Everything ships as a pure frontend change (Vue + i18n); the Rust
  backend already exposed `discovery.installDirect(...)`, `endpointId`
  and `hash`, so no `frontend/src-tauri` code was touched in this
  release. Rust crate version is nevertheless bumped in lockstep
  (SmartNet client versions are workspace-global) for release-tracking
  consistency.

### Files touched
- `frontend/src/views/Resolve.vue` - direct-link query parsing +
  `installDirect` fast-path + themed badge.
- `frontend/src/views/MyApplications.vue` - direct-link copy button +
  QR button + FileQrModal wiring.
- `frontend/src/components/FileQrModal.vue` - generalised to accept a
  raw `link` prop (previously hard-coded to `sn://file/...`).
- `frontend/src/i18n/{en,ru,id}.ts` - 8 new keys.
- `frontend/package.json`, `frontend/src-tauri/tauri.conf.json`,
  `frontend/src-tauri/Cargo.toml`, `frontend/src-tauri/Cargo.lock`
  (1.128.0 → 1.129.0).

### Notes
- No Oxid Mail changes in this release; Oxid Mail stays on **0.8.0**.
- Per user policy in this session, no `cargo check` / `yarn build` /
  automated tests were executed by the agent - the release is
  user-verified locally.

---

## [1.128.0] - 2026-07 (Private dApps hardening: on-disk `content.json` fallback, cloud/tracker announce, gold catalog badge, "Private Repository" label, no-earnings coin hide)

### Added - Private dApp catalog experience
- **Gold "Private apps found (N)" badge** on `Discovered.vue`: appears next to
  the search bar whenever the network reveal payload decrypts at least one
  private dApp for the current wallet. Clicking the badge filters the catalog
  down to those private apps only (toggle). RU / EN / ID translations added
  in `frontend/src/i18n/{ru,en,id}.ts`.
- **"🔒 Private Repository" label** replaces the "✔ Official Repository"
  chip inside the private dApp modal (`Discovered.vue`) - clearer signal that
  the app is E2EE-gated and not part of the public official catalog.
- **In-app RAM decryption progress page** (`private_dapp.rs`,
  `protocol.rs::serve_private_dapp`): when a private dApp is opened while the
  in-memory decryption is still running, the tab now shows an auto-generated
  themed progress splash ("Decrypting encrypted container…") instead of the
  raw "App not found" error. The page auto-refreshes every 800 ms until
  `PRIVATE_RAM` has the entry and then swaps to the real `sth://` payload.

### Fixed - Private dApp lifecycle
- **`Manifest.private` compile error (E0609)** in
  `frontend/src-tauri/src/lib.rs`: added the missing
  `private: Option<bool>` field on `Manifest` so downstream code
  (`apps.rs::ingest_private`, `protocol.rs`, `cloudseeder.rs`) can compile
  and correctly branch between public and private ingest/serving paths.
- **"App not found" 404 on private dApps even for the owner** (fresh install
  after a wallet reload): `protocol.rs` now falls back to reading
  `content.json` from the app directory when the in-memory manifest cache
  is empty; if `"private": true` is set on disk, the request is routed to
  the private-decryption pipeline instead of the public 404 branch. Owners
  and access-listed clients no longer see stale errors while `PRIVATE_RAM`
  spins up.
- **Cloud Drive replication skipped private dApps**
  (`frontend/src-tauri/src/cloudseeder.rs`): the archive builder now
  includes `app.enc` + `access.json` for private manifests (skipping the
  no-longer-used `dist/`), and the announce payload carries the same
  reveal blob as the tracker path. Private dApps now correctly propagate
  via headless Google Drive seeders (both primary and secondary drive
  dual-replication introduced in 1.123.0).
- **Headless seeder + HTTP tracker rejected private announces**
  (`seeder/src/main.rs`, `tracker/src/main.rs`): the ECDSA verification
  payload for the announce was `id|name|description|merkle`, but private
  dApps ship placeholder name/description on the public manifest, so
  signatures never matched and every announce was 401'd. Both the seeder
  and tracker now branch on the `private` flag and verify the tuple
  `id|"__private__"|"__private__"|merkle` (matching what
  `apps.rs::publish_private` signs), unblocking peer discovery through
  headless seeders and public trackers.

### Changed - Installed apps UX
- **Golden "$" coin hidden when a dApp has no accruals**
  (`frontend/src/views/Installed.vue`): the earnings-summary popover trigger
  on each installed-app card is now gated by a new `hasEarnings(id)` helper
  that inspects the earnings snapshot (`useEarningsStore.byApp`). If neither
  the estimated (`estSth`) nor the forecast (`forecastSth`) reward is
  positive, the coin is not rendered - cutting visual noise for apps that
  are installed but not currently being seeded / rewarded. The store is
  refreshed on mount so the first paint already reflects real trackers'
  data.

### Notes
- No Oxid Mail changes in this release; Oxid Mail stays on **0.8.0**.
- No blockchain or on-chain protocol changes; only client-side manifest
  handling and tracker/seeder verification tuples were adjusted.

---

## [1.127.0]  (Private dApps: E2EE containers with on-chain access lists)

### Added - Private dApp (owner-controlled access list)
- **Publish flow** (`MyApplications.vue` → PUBLISH tab): new "🔒 Private dApp
  (E2EE)" checkbox with a textarea for SmartHoldem addresses (one per line) -
  who can discover, install and open the dApp. Native client only. Each listed
  address must have ≥1 outgoing on-chain tx (public key must be visible).
- **E2EE container** (`private_dapp.rs`, `apps.rs::ingest_private`): the dist
  folder is tar-packed and encrypted with AES-256-GCM under a random container
  key (CEK). CEK is ECIES-wrapped (secp256k1 ECDH - same crypto as private
  files) for every listed address + the owner. On disk everyone stores ONLY
  `app.enc` + `access.json`; both files are covered by the manifest merkle →
  protected by the v2 signature. The public manifest carries a placeholder
  name ("🔒 Private dApp"); real name/description/tags/logo live encrypted in
  `access.json` (privMeta).
- **RAM-only decryption** (`protocol.rs` + `PRIVATE_RAM`): sth:// requests for
  a private dApp are served from an in-memory decrypted map. Plaintext never
  touches the disk. The map is dropped on tab close (`tabs.ts`), wallet lock
  (`session_clear`) and app exit; a themed "locked" page is served when the
  session is locked or the address is not on the list.
- **Discovery/search visibility**: tracker announces carry a reveal payload
  (`§P1§` marker + encrypted meta + wrapped-key map) in the description
  field. Clients on the access list decrypt the real name/description/tags
  and see the dApp in search/catalog (bypassing the Official filter - an
  explicit owner invitation); all other clients hide it entirely
  (fail-closed). Results are cached in Sled per payload hash.
- **Access list editing** (`ACCESS` button on published private cards):
  owner edits the address list at any time. Every save ROTATES the container
  key (re-encrypts `app.enc` with a fresh CEK, re-wraps for the new list,
  version++, re-sign) - removed addresses lose access cryptographically, not
  cosmetically. No source folder needed: the tar is recovered by decrypting
  the current container with the owner key.
- **Seeding**: everyone may seed the encrypted container (owner's choice:
  survivability over traffic privacy). Payment scheme is unchanged - private
  dApps publish via the same escrow flow (or free, without tracker announce).

### Known trade-offs (documented by design)
- The member ADDRESS LIST is visible metadata (needed for client-side
  membership checks and key rotation); the content, name, description and
  logo remain encrypted.
- LAN-only (mDNS) announces cannot carry the reveal payload (TXT size), so
  member visibility requires a tracker round-trip or a direct sn:// link.

### Versions
- SmartNet client → **1.127.0** (`package.json`, `tauri.conf.json`,
  `Cargo.toml`, `Cargo.lock`).

---

## [Oxid Mail 0.8.0]  (P2P reply keys + auto-contact + honest key-lookup error)

### Problem
A pure-P2P user (only `SADDR@sth`, registered name + messenger keys in
SmartNet) could receive an encrypted P2P letter but could **not** reply:
mail key lookup only checks the local address book and the mail blockchain
registry, and the registry is populated exclusively by the email
Proof-of-Ownership binding - which P2P accounts never perform. Messenger
`xkey:` publications live in a different sink and carry no Iroh node ID,
so mail cannot use them. On top of that, the "keys not found" error advised
subscribing to the user in SmartNet, which never distributes mail keys -
a misleading, unimplemented hint.

### Added
- **Reply keys inside the NTFRY envelope** (`mail-crypto/envelope.rs`):
  optional `replyX25519Hex` / `replyIrohHex` fields in `EnvelopeContent`.
  They travel *inside* the ciphertext and under the sender's secp256k1
  signature, so they are private and authenticated. Old clients ignore the
  unknown JSON fields; old letters deserialize with `None` - the format is
  forward- and backward-compatible.
- **Sender side** (`mail-core/service.rs::send_p2p_message`): every outgoing
  P2P letter now embeds the sender's own x25519 public key and Iroh node ID
  derived from the seed.
- **Auto-contact on receive** (`mail-core/p2p.rs::store_incoming`): when an
  incoming letter carries reply keys (signature already verified, and the
  sender address is derived from the same signing pubkey - keys are
  authentic), a `p2p` contact `SADDR@sth` is created automatically. Existing
  contacts with non-empty keys are never overwritten (Local Trust priority),
  contacts with empty keys are filled in. Result: *anyone who writes to you
  first becomes instantly replyable* - no manual card exchange.

### Changed
- **Honest "keys not found" error** (`service.rs`): the false "subscribe in
  SmartNet profile" advice replaced with real options - ask the recipient
  for their `sn://add-email?…` contact card (Oxid Mail → Contacts → My
  card), or let them write first (reply keys auto-save), or bind an email
  in the registry.

### Versions
- All `oxid-mail` workspace crates (`mail-core`, `mail-crypto`, `mail-store`,
  `oxid-bridge`, `src-tauri`), `ui/package.json`, `tauri.conf.json` and
  `Cargo.lock` → **0.8.0**. SmartNet client untouched (1.126.0).

---

## [1.126.0] - 2026-07 (UI polish rollup: topbar mail shortcut + Default Light readability)

### Added
- **Topbar mail icon** (`AppLayout.vue`): envelope button
  (`data-testid="topbar-mail"`) placed before the settings gear, launches
  Oxid Mail via `launch_oxid_mail`. Shows an unread-count badge fed by the
  `mailBadge` store (IPC `oxid-mail-event` → `unread`) and disables itself
  while the mail window is starting. New i18n key `topbar.mail` (en/ru/id).

### Fixed
- **Sidebar NETFORY wordmark in light themes** (`AppLayout.vue`): the white
  "NET" half was invisible on the white sidebar. Both halves now switch via
  CSS `light-dark()` - dark ink + darkened teal in light themes, original
  white + neon cyan in dark themes.
- **SETTINGS section top tabs in Default Light** (`Settings.vue`): the
  metallic dark-slate gradients were hardcoded, so the seg-buttons stayed
  black on the light theme. Added a `[data-theme='default-light']` override:
  white tray, soft grey pills, teal `accent-soft` fill for the active tab.
- **Default Light readability overhaul** across `MyFiles.vue`, `Music.vue`,
  `Storages.vue`, `Seeders.vue`, `CryptoVault.vue`, `AlbumCard.vue`:
  hardcoded dark-theme slate colors (`#e2e8f0`, `#94a3b8`, `#64748b`,
  near-black `rgba(0,0,0,…)` panels) replaced with `var(--text)` /
  `var(--text-dim)`; accent hues (cyan/teal/green/purple/amber) switch to
  WCAG-friendly variants through `light-dark()`; dark chip/card/input
  backgrounds flip to light equivalents. Dark theme visuals unchanged;
  dark overlays (drag-and-drop, toasts, modal backdrops) keep their
  original colors in both schemes.

### Notes
- Consolidates the four UI-polish items originally landed under 1.125.0 into
  a single minor bump. No functional / API / storage / crypto changes.
  Rust backend (`frontend/src-tauri/Cargo.toml`) intentionally not bumped in
  this release - no Rust code was touched.

---

## [1.125.0]  (Topbar mail shortcut + Default Light readability overhaul)

### Added
- **Topbar mail icon** (`AppLayout.vue`): a new envelope button
  (`data-testid="topbar-mail"`) sits before the settings gear and launches
  Oxid Mail via `launch_oxid_mail`. It shows an unread-count badge fed by the
  `mailBadge` store (IPC `oxid-mail-event` → `unread`) and disables itself
  while the mail window is starting. New i18n key `topbar.mail` (en/ru/id).

### Fixed
- **Sidebar wordmark in light themes** (`AppLayout.vue`): the white "NET" part
  of the NETFORY wordmark was invisible on the white sidebar. Both halves now
  use CSS `light-dark()` - dark ink + darkened teal in light themes, original
  white + neon cyan in dark themes.
- **SETTINGS section tabs in Default Light** (`Settings.vue`): the metallic
  dark-slate gradients were hardcoded, so the top seg-buttons stayed black on
  the light theme. Added a `[data-theme='default-light']` override: white tray,
  soft grey pills, teal `accent-soft` fill for the active tab.
- **Default Light readability overhaul** (247 style replacements across
  `MyFiles.vue`, `Music.vue`, `Storages.vue`, `Seeders.vue`, `CryptoVault.vue`,
  `AlbumCard.vue`): pages were styled with hardcoded dark-theme slate colors
  (`#e2e8f0`, `#94a3b8`, `#64748b`, near-black `rgba(0,0,0,…)` panels), which
  produced pale/invisible text on light themes. All text colors now use
  `var(--text)` / `var(--text-dim)`, accent hues (cyan/teal/green/purple/amber)
  switch to darkened WCAG-friendly variants through `light-dark()`, and dark
  chip/card/input backgrounds flip to light equivalents. Dark-theme visuals are
  unchanged; dark overlays (drag-and-drop, toasts, modals backdrops) keep their
  original colors in both schemes.

## [1.124.4]  (UX: click-to-copy full address on user profile)

### Added
- **User profile → address chip under avatar** (`views/UserProfile.vue`):
  the short address (e.g. `SSU6Tv...niZu`) rendered below the avatar is now
  a clickable / keyboard-focusable button. Clicking it copies the **full**
  identity address to the clipboard and shows a toast
  (`profile.addressCopied`). Enter and Space keys work the same way for
  accessibility. Hover / active / focus-visible styles were added so the
  affordance is obvious. The `title` attribute now also includes a
  localized hint (`profile.clickToCopy`).
- New i18n keys `profile.addressCopied` and `profile.clickToCopy` in
  `en.ts`, `ru.ts`, `id.ts`.

### Notes
- Reuses the existing `copy(text, label)` helper, so behaviour is
  consistent with other clipboard actions on the profile page.

---

## [Oxid Mail 0.7.2]  (Share Contact polish + anti-spam send hint)

### Changed
- **Share My Contact modal** (`ShareContactModal.vue`):
  - The "classic email" block is hidden when no classic (SMTP/IMAP) mailbox
    is added - for the system P2P account it merely duplicated the
    "p2p address" row (`<addr>@sth`). Rows are renumbered dynamically.
  - When the user owns a registered `u://name`, a fourth row
    **"u://name alias"** (`name@sth`) is shown with copy/QR - powered by the
    existing `/naming/my` reverse lookup.
- **P2P send error** (`mail-core/src/service.rs`): when the recipient's keys
  are not found, the error now explains the anti-spam rule - subscribe to the
  user in their SmartNet profile to make their keys available (in addition to
  the existing "add contact / wait for registry binding" hints).

---

## [1.124.3]  (Fix: DEVELOPERS bypass for unverified dApps agreement)

### Fixed
- **Discover Apps → "Show unknown DAPPs"** - addresses listed in `.env
  DEVELOPERS=` lost their free bypass of the on-chain
  `NETFORY_UNVERIFIED_ACCEPT` agreement: in a packaged build neither the
  runtime `.env` (no file next to the exe) nor the compile-time
  `option_env!("DEVELOPERS")` (not exported during build) was populated, so
  `official::seed_developers()` fell back to the admin-only default and the
  payment button was shown instead of the toggle checkbox.
  - `official.rs::seed_developers()` now falls back to the `frontend/.env`
    embedded at build time via `include_str!` (same pattern as the
    `CHUNK_SIZE` fallback in `lib.rs`).
  - `wallet.rs::check_unverified_accept` now uses
    `dns_manager::is_developer()`, which also honours the compile-time
    `DEVELOPERS` constant - the same gate that already worked for reserved
    domain names.
- Removed a duplicated `DEVELOPERS=` line in `frontend/.env`.

---

## [1.124.2] - 2026-07 (Torrents & My Files modals i18n - RU / EN / ID)

### Changed
- **`TorrentLimitsModal.vue`** - "Torrent Client Settings" is now fully
  translatable. New `torrentLimits.*` namespace covers all three tabs
  (Connection / Speed / BitTorrent), the µTP protocol select, random-port row,
  UPnP checkbox, peer-limit input, download/upload sliders with `KiB/s` / `MiB/s`
  unit switch, DHT/LSD toggles, RuTracker keeper-key field (label + hint +
  placeholder), OK / SAVING… / Cancel footer, and the ⚠ restart-required notes.
  The `label(kib)` helper now reads unit strings from the current locale
  (KiB/s → EN `KiB/s`, RU `KiB/s`, ID `KiB/d`; MiB/s analogously).
- **`TorrentCreateModal.vue`** - "Create Torrent" localised. New
  `torrentCreate.*` namespace covers header tag + title, file/folder pickers,
  path placeholder, piece-length select (Auto + KiB/MiB variants localised
  via computed), private-torrent checkbox with hint, start-hint, trackers
  textarea section label, comment field, error messages (no path, generic
  failure) and Cancel / CREATING… / CREATE TORRENT buttons.
- **`AddFileModal.vue`** - "ADD FILE" (My Files) localised. New
  `addFile.*` namespace covers title, dropzone label "Pick a file from disk",
  privacy toggle (🔒 Private file (E2EE)), recipients label + placeholder +
  E2EE hint (AES-256-GCM + ECDH), paid file toggle, price label, error
  messages (no file, no recipients, invalid SmartHoldem address with `{addr}`
  interpolation, price ≤ 0), progress-time labels (ENCRYPTING / HASHING),
  submit labels (ENCRYPT & ADD / PUBLISH), and the CID/local-storage footer
  note.

### Files touched
- `frontend/src/components/TorrentLimitsModal.vue` - added `useI18n`,
  ~35 template + 2 script substitutions, `label()` now uses reactive unit
  strings via `computed()`.
- `frontend/src/components/TorrentCreateModal.vue` - added `useI18n`,
  ~20 template + 2 script substitutions, `pieceOptions` moved from static
  const to `computed()` so piece-size unit strings react to locale.
- `frontend/src/components/AddFileModal.vue` - added `useI18n`, ~13 template
  + 4 script (error message) substitutions with `{addr}` interpolation for
    the bad-address error.
- `frontend/src/i18n/{en,ru,id}.ts` - added `torrentLimits.*` (~30 keys),
  `torrentCreate.*` (~20 keys), `addFile.*` (~15 keys).
- `frontend/src-tauri/tauri.conf.json`, `Cargo.toml`, `frontend/package.json`
  (1.124.1 → 1.124.2).

### Rationale
- Same treatment as 1.124.1 for Seeders.vue - frontend UX strings only,
  no auth/storage/network semantics touched. Rust backend untouched.
  Placeholder attributes bound reactively via `:placeholder="t(…)"` so
  language switches take effect without a modal remount.

---

## [1.124.1] - 2026-07 (Seeders.vue full i18n - RU / EN / ID)

### Changed
- **`views/Seeders.vue` fully i18n-driven** - the `// PUBLIC NETWORK SEEDERS_`
  screen no longer contains any hard-coded Russian. New `pubSeeders.*` namespace
  in `frontend/src/i18n/{en,ru,id}.ts` (~130 keys) covers:
  - Header (title, subtitle with `<b>` interpolation via `v-html`)
  - Session summary (`/ntfryseed · SESSION`) - plural rules for archive count
    (RU: 1/2/many via i18n plural, EN: n/n archives, ID: n arsip)
  - Publish card - description, publish button (idle/busy), no-account gate,
    "→ Open Crypto Vault" link
  - Secondary drive selector (label, off-option, hint)
  - Mail.ru manual weblink block (open folder, apply button, hint)
  - Live progress lines (meta phase, archive phase, packing/uploading/done/error,
    retry badge with `progAttempt` count, finishing state)
  - Publish result (archives label, `updated {when}`, mirror URI label)
  - Auto-replication row (`Auto-replication every 30 min - ON/OFF`,
    last-publish text with `{when}` slot, pending hint)
  - Replicate Profiles switch (label with `<b>`, ON/hourly text, last-replication,
    "publish only when changed" note, PUBLISH NOW button)
  - Profile result stats (uploaded, unchanged, files, dedup, orphans)
  - vDapps section (tag, hint, announced-at, devs/dApps/seeders labels, none-state)
  - Fetch card (description, FETCH button idle/busy)
  - dApp list (archive badge + title, INSTALL button idle/busy, add-to-registry)
  - Profiles fetch (FETCH PROFILES button, `postsShort {n} posts` plural,
    empty state)
  - Cloud profile modal (`☁ from cloud` header, comments plural, empty-posts,
    LOAD / DOWNLOAD buttons)
  - Known Seeders registry (title with count `{n}`, holding badge+tooltip,
    CHECK ALL button, description, empty state, HOLDING marker, REMOVE button)
  - Origin badges (own/manual/gossip/seeders.enc/bootstrap) - label + title
  - Status text (`not checked`, `fetch OK · {when}`, `unavailable · last success: {when}`,
    ETA states `~N s / ~N min / ~H h M min left`, server-processing)
  - Units (MB / KB / MB/s / KB/s - localized per locale)
  - Toasts (open MR folder, bad link, check done, installed from cloud,
    QR failed, published, copied, copy failed, already in registry,
    added to registry)
  - Errors (generic, verify signature fallback, Secondary drive prefix,
    Mail.ru publish fallback long message)
- Time-formatting `toLocaleString()` uses default browser locale (no hard-coded
  `ru-RU`), matching the SystemConsole i18n pass from 1.124.0.

### Files touched
- `frontend/src/views/Seeders.vue` - `useI18n` wired in setup, ~90 template
  substitutions + ~15 script substitutions (toasts, error messages, badge
  labels, status strings, size/speed/ETA formatters)
- `frontend/src/i18n/en.ts` - added `pubSeeders.*` (English strings)
- `frontend/src/i18n/ru.ts` - added `pubSeeders.*` (Russian - verbatim from
  the original hard-coded copy)
- `frontend/src/i18n/id.ts` - added `pubSeeders.*` (Bahasa Indonesia)
- `frontend/src-tauri/tauri.conf.json`, `Cargo.toml`, `frontend/package.json`
  (1.124.0 → 1.124.1)

### Rationale
- Same treatment as the SystemConsole i18n pass - user-facing labels belong to
  the frontend translation layer; there is no functional change (no auth flow,
  API contract, encryption or storage code was touched). Vue-i18n plural rules
  used for archive/posts/comments counts so the RU locale renders proper
  "archive/archives" morphology.

---

## [1.124.0] - 2026-07 (Sidebar logo shrink · System Console i18n · English-only Rust syslog)

### Changed
- **Sidebar logo**: `NETFORY` brand mark reduced by 2× (48×48 → 24×24)
  and the gap to the wordmark tightened from 18 px to 10 px
  (`frontend/src/layouts/AppLayout.vue`). Hover glow and lift preserved,
  just proportionally softer. Fits comfortably above the collapse button
  and no longer visually dominates the DASHBOARD row.
- **System Console UI is now fully i18n-driven**
  (`frontend/src/components/SystemConsole.vue`). All hard-coded Russian
  strings (title "SYSTEM CONSOLE", filter placeholder, empty state
  "Waiting for network events…", action buttons "COPY ALL / EXPORT TXT /
  CLEAR / TO WINDOW / ✓ COPIED", scope chip tooltip, repeat-badge
  tooltip, dApp detail-modal section headers) moved to a new
  `sysconsole.*` block in `en.ts` / `ru.ts` / `id.ts`. Time formatting
  now uses the current locale (`toLocaleTimeString(undefined, …)`) instead
  of hard-coded `ru-RU`.
- **All Rust `syslog::log(...)` messages translated to English**
  (127 lines across `p2p.rs`, `updater.rs`, `torrent.rs`, `cloudseeder.rs`,
  `cloudvault.rs`, `devhub.rs`, `devhub_pr.rs`, `devhub_ipc.rs`,
  `translator.rs`, `swarm/{engine,bandit}.rs`, `netwatch.rs`, `stealth.rs`,
  `profile_seeder.rs`, `social.rs`, `protocol.rs`, `official.rs`).
  Emoji markers (🚀 📈 💀 🎉 🏆 🎲 🎯 ✅ ⛔ ⚠️) preserved as visual level
  indicators. Semantics unchanged - every message reads the same, just
  now in English. This aligns with the industry convention that low-level
  P2P / network / DHT / updater logs are English-only (technical logs
  should be searchable and copy-pastable into GitHub issues without a
  locale mismatch).

### Rationale
- Return-error / dialog-filter strings in Rust that surface as UI toasts
  or system file dialogs were intentionally left in Russian - they are
  end-user messages, not technical logs, and belong to a separate i18n
  pass. Only `syslog::log(...)` (which feeds the System Console) was
  switched to English in this release.

### Files touched
- `frontend/src/layouts/AppLayout.vue` (sidebar logo size + gap)
- `frontend/src/components/SystemConsole.vue` (full i18n wiring)
- `frontend/src/i18n/{en,ru,id}.ts` (new `sysconsole.*` section)
- `frontend/src-tauri/src/{stealth,netwatch,updater,devhub,devhub_pr,
  devhub_ipc,translator,cloudvault,social,cloudseeder,profile_seeder,
  p2p,official,torrent,protocol}.rs` (127 syslog strings)
- `frontend/src-tauri/src/swarm/{engine,bandit}.rs`
- `frontend/src-tauri/tauri.conf.json` (1.123.0 → 1.124.0)
- `frontend/src-tauri/Cargo.toml` (1.123.0 → 1.124.0)
- `frontend/package.json` (1.123.0 → 1.124.0)

---

## [Oxid Mail 0.7.1] - 2026-07 (English-only armor banner for outbound encrypted mail)

### Changed
- **`armor_body` banner is now English-only**
  (`oxid-mail/crates/mail-crypto/src/armor.rs`). When a NETFORY 0xID user
  sends an end-to-end-encrypted letter to a legacy mail provider
  (Gmail / Outlook / etc.), the pre-payload sniff banner that is visible
  to non-NETFORY recipients used to contain three lines - two Russian
  and one English. Two of them have been consolidated into a single
  English pair so that international recipients do not see Cyrillic text
  they cannot read:
    ```
    This message is encrypted with SmartNet Oxid Mail.
    Open it in the NETFORY 0xID mail: https://smartholdem.io
    ```
  The `-----NTFRY START-----` / `-----NTFRY END-----` sentinels and the
  base64 payload are unchanged, so decryption on the receiver side is
  fully backward-compatible - only the human-readable preamble was
  reworded. `docs/13-Oxid-Mail-E2E-Encryption-Plan.md` was refreshed to
  match the new wording.

---

## [1.123.0]  (Google Drive public seeder `sn://seed/g@…` + Secondary drive dual replication)

### Added
- **Google Drive as a third Cloud Seeder provider** (`ntfry/src/seeder.rs`).
  Author side uses Drive API v3 with the existing Crypto Vault Google OAuth
  account (`drive.file` scope): the `/ntfryseed` folder (+`dapps/` subfolder)
  is created at Drive root, encrypted state files and dApp archives are
  upserted by name (multipart create / media update), and the folder is
  published with an *anyone-with-link reader* permission. The folder ID
  becomes the `folder_token` of the new URI form
  `sn://seed/g@<folder_id>@<public_key>`.
- **Anonymous Google Drive reads without OAuth or API keys**: peers list the
  public folder via the HTML `embeddedfolderview` endpoint (per-level path
  resolution for `dapps/…` and `users/…` subfolders) and download blobs via
  `drive.usercontent.google.com/download`, including a fallback that replays
  the hidden-input form of the large-file virus-scan interstitial - the same
  anonymous-read model already used for Mail.ru weblinks.
- **Resilient Google uploads**: the shared `resilient_put` engine (progress
  stall guard, 3 attempts, live progress callbacks) is reused; strict
  pre-checks compare `sha256Checksum` from Drive metadata, so unchanged
  archives are skipped with zero traffic. New public API:
  `gdrive_upload_seed`, `gdrive_upload_seed_file`, `gdrive_upload_subpath`,
  `gdrive_delete_seed_dapp_file`, `gdrive_delete_seed_root_file`,
  `gdrive_delete_subpath`, `gdrive_fetch_public` (+parser unit tests).
- **Secondary drive (dual replication)** - a second cloud account from
  Crypto Vault (Yandex / Mail.ru / Google) can be selected as a mirror in
  `[ PUBLISH NETWORK STATE ]`. Manual publish, the 30-minute auto-replication
  and *Replicate Profiles* now push `/ntfryseed` to BOTH drives sequentially;
  the mirror URI is stored in its own Sled slot (`cfg:own_seeder_uri2`),
  joins the seeder registry as `own` and is broadcast in gossip announcements
  alongside the primary URI, so the network knows both mirrors.
- **Per-drive delta gates**: the `appId → merkle` uploaded-archive map, the
  profile-bundle snapshot and the `update.enc` version gate are now keyed per
  provider × role (with transparent migration of the legacy single-drive
  keys), so mirroring to a fresh drive re-uploads everything it actually
  misses instead of being skipped by the primary's markers.

### Changed
- `cloud_seeder_publish` / `cloud_seeder_publish_profiles` accept `gdrive`
  (or `google`) as `provider` plus a new optional `secondary` flag;
  `cloud_seeder_delete_dapp` also cleans mirrors. `cloud_seeder_fetch`,
  cloud-fallback scans and cloud installs (`cloud_seeder_install*`) now fully
  support `g@` URIs - the "Google Drive seeders - next phase" stubs are gone.
- Seeders page: provider badge shows *Google*, a `SECONDARY DRIVE` selector
  appears when more than one cloud account is connected, mirror errors are
  shown as a non-fatal amber warning and the mirror URI is rendered under the
  publish result with its own copy button.
- Background replication failures of the mirror never fail the primary
  publish (best-effort semantics confirmed by the user).

## [1.122.3] - 2026-07 (`DonateModal` i18n - English + Indonesian translations of "SUPPORT THE NETWORK_")

### Added
- **Full i18n coverage for the `SUPPORT THE NETWORK_` donation modal**
  (`frontend/src/components/DonateModal.vue`,
  `frontend/src/i18n/{ru,en,id}.ts`). Every hard-coded Russian string in
  the modal - title, subtitle, both intro paragraphs, the "pick an
  address below" CTA, the copy / copied button captions and their
  tooltips - is now driven by the new `donateModal.*` namespace. Vue-i18n
  `useI18n()` is wired into the component and switches all copy in place
  when the user flips the sidebar language selector, so a Russian user
  keeps the current wording, an English user gets:
  * *"Contribute to the freedom of the network"*
  * *"SmartNet is a fully independent project - no censorship, no
    venture investors, no corporate control. …"*
  * *"Back independent Web4 - pick the address that suits you below:"*
  * COPY / ✓ COPIED buttons + "Copy address" tooltip.
    and an Indonesian (`id`) user gets:
  * *"Berkontribusi untuk kebebasan jaringan"*
  * *"SmartNet adalah proyek yang sepenuhnya independen - tanpa sensor,
    tanpa investor ventura, tanpa kendali korporat. …"*
  * *"Dukung Web4 yang independen - pilih alamat yang sesuai di bawah:"*
  * SALIN / ✓ TERSALIN buttons + "Salin alamat" tooltip.
    Because "SmartNet" opens the first sentence in all three locales and
    needs to stay bold, the template keeps `<b>SmartNet</b>` and simply
    appends the localised continuation of the sentence via
    `t('donateModal.intro1')` - a pragmatic compromise that avoids
    `v-html` while still preserving the bold brand accent.

---

## [1.122.2] - 2026-07 (Sidebar brand refresh - raster NETFORY logo, retired Mail & Seeders entries, Dashboard `Micro Blog` card)

### Added
- **Raster NETFORY logo in the sidebar** (`frontend/public/netfory_logo.png`,
  `frontend/src/layouts/AppLayout.vue`). The previous procedural mark
  (two rotating CSS squares around a pulsing dot) has been replaced with a
  48×48 px PNG rendered via a plain `<img class="nf-logo">`. The wordmark
  spacing is now `gap: 18px` (was `9px`) so the icon sits with a clear
  15–20 px breathing room to the left of `NET` / `FORY`. A subtle cyan
  drop-shadow + `translateY(-1px) scale(1.04)` hover matches the neon
  aesthetic of the rest of the shell without the earlier rotate animation.
  Everything is skipped when the sidebar is in the collapsed state (rail
  mode), consistent with the previous behaviour.
- **Dashboard `Micro Blog` card** (`frontend/src/views/Dashboard.vue`). The
  legacy `// FORUM · P2P forum · uncensored discussions` tile is now
  `// MICRO BLOG · P2P Blog · uncensored discussions` and its click target
  routes to `{ name: 'profile' }`, i.e. the current user's own public
  profile page (`views/UserProfile.vue`). This lets users reach their
  micro-blog with a single tap from the dashboard and unblocks the future
  transition to a proper feed (top-5 popular / newest posts) without
  changing the tile itself.

### Changed
- **Retired `MAIL` (Oxid Mail) launcher from the sidebar** - the standalone
  bottom-cluster button that spawned the native mail app is no longer
  rendered. The Rust launcher command and the `openOxidMail` handler are
  intentionally kept in place because the `new_mail` toast handler still
  invokes them when a message arrives, and because the Dashboard `Mail`
  tile continues to be the officially supported entry point.
- **Retired `SEEDERS` nav item** (`frontend/src/config/nav.config.ts`).
  The Cloud Seeders (`ntfryseed`) UI is only relevant to power users who
  administer their own public replication node; keeping it in the primary
  sidebar was pulling visual attention away from the core P2P storage and
  discover flows for DEV/GEEK modes as well. Fetch/publish/registry
  management remains reachable from the internal registry APIs, Settings,
  and the Discover page fallback - the background replication timer keeps
  running unchanged.

---

## [1.122.1] - 2026-07 (`.sth` DEVELOPERS-only guard for 1–2 char labels + `dns_manager.rs` trailing-delimiter hotfix)

### Fixed
- **1–2 character `.sth` domain labels are now strictly DEVELOPERS-only**
  (`frontend/src-tauri/src/dns_manager.rs`, `frontend/src/lib/bridge.ts`,
  `frontend/src/components/DomainManager.vue`). Previously the tier-price
  ladder allowed *any* wallet to buy an ultra-short label as long as it
  paid the (very low) base fee - only the `CONFIG_ADMIN` wallet was gated
  in `register_domain`, and reserved-name lookup completely ignored the
  1–2 char class. The three-layer defence now enforces the same
  `DEVELOPERS` allowlist that already applies to reserved names:
  * `ingest_tx` (chain replay) - drops any historical short-label
    registration that was not signed by a `DEVELOPERS` address, so the
    client-side registry can never "resurrect" a squatted 1-char name.
  * `domain_lookup` - reports `reason: "reserved"` for short labels to
    non-developer callers so the UI grays out the register button and
    shows a clear explanation instead of a misleading price.
  * `register_domain` - hard-rejects the command with
    `"RESERVED - 1-2 character names are only available to network developers"`
    as a belt-and-braces backstop for wallets that would try to bypass
    the UI. The web preview mirrors the same rule in `webDomainLookup`
    so the browser build cannot mislead the user either.
- **`dns_manager.rs` "unexpected closing delimiter" compilation hotfix** -
  a previous `search_replace` operation left three stray lines
  (`memo_matches(&other_trunc, &full));\n    }\n}`) below the real end of
  the `#[cfg(test)]` module, which broke the entire Tauri build with
  `error: unexpected closing delimiter: )` at line 1190. The tail was
  trimmed back to the canonical `#[cfg(test)] mod tests { … }` block, so
  the crate builds cleanly again.

### Changed
- Doc-comments and error messages that still referred to the legacy
  "admin only" concept for 1–2 char labels are now uniformly phrased
  as "DEVELOPERS-only" to match the new gate - including the field
  documentation on `DomainStatus::admin_only`, the `tier_price` header
  and the human-readable rejection strings.

---

## [1.122.0 / Oxid Mail 0.7.0] - 2026-07 (Paid unlock for unverified dApps, reserved-names guard, recipient chips + contact autocomplete, ReaderView empty-screen fix)

### Added
- **Paid on-chain unlock for unverified dApps** - the SmartNet dApp catalog
  now surfaces a modal (`frontend/src/components/UnverifiedDappUnlockModal.vue`)
  the first time a user opens a dApp that is not on the official/verified
  list. The user acknowledges the risk (unaudited code, potential loss of
  funds) and confirms a single on-chain STH transfer of **FEE = 9999 STH** to
  the platform address; the transaction carries a signed marker
  `vendorField: NETFORY_UNVERIFIED_ACCEPT:<dappId>`. Once the transaction is
  confirmed, the acceptance is cached locally (per-address, per-dappId) and
  the user is admitted straight into the dApp on subsequent launches - no
  second payment, no repeated modal.
- **DEVELOPERS bypass for the 9999 STH fee** - a comma-separated
  `DEVELOPERS` list in `frontend/.env` (also mirrored via `vite.config.js`
  into `import.meta.env`) whitelists SmartHoldem addresses that skip the
  paid unlock entirely. `frontend/src-tauri/src/wallet.rs` reads the same
  variable at runtime, so both the UI gate and the Tauri command paths
  short-circuit the transaction for whitelisted developer wallets. This is
  a `.env`-only knob - no UI, no persistence, no bypass without editing the
  file.
- **New environment flag `FEE_ALLOW_ALL_DAPPS`** (`frontend/.env`,
  `vite.config.js`) - a global escape hatch that disables the paid-unlock
  gate entirely for development / testnet builds. Off by default.
- **Reserved-names guard for `u://name` and `sth://name` registration**
  (`frontend/public/reserved.txt`, `frontend/src-tauri/src/naming.rs`,
  `frontend/src-tauri/src/dns_manager.rs`) - the naming layer now refuses
  to register any human-readable name that appears in the shipped
  `reserved.txt`. The list is fetched at UI startup and mirrored inside the
  Rust `naming` module so the guard is enforced on **both** sides (UI form
  validation + backend command). The error surface has been reworded from
  the generic "invalid name" to a clear "this name is reserved". Addresses
  in the `DEVELOPERS` allowlist can still register reserved names (e.g. for
  bootstrapping official handles).
- **Compose recipient chips + contact autocomplete**
  (`oxid-mail/ui/src/components/ComposeModal.vue`) - the "To" field is now
  a chip-based input. As the user types, matches from the local address
  book (`mail-core::contacts`) are surfaced live under the field; commas,
  Enter and blur promote the input to a chip, Backspace on the empty input
  removes the last chip. Up to **20 recipients** per message are allowed;
  duplicates are collapsed silently. Each chip has a per-item ✕ delete
  affordance.
- **Multi-recipient SMTP send in `mail-core`** - `send_message` now accepts
  a `to: string[]` array end-to-end. `smtp_client::send_mail` serialises the
  full list into the `To:` header and passes every address to `RCPT TO`, so
  a single compose can fan out to many mailboxes with one SMTP session.
  Both the plain and NTFRY-encrypted code paths reuse the same array; the
  sender's own decryption slot is still appended automatically for
  encrypted sends.

### Changed
- **P2P (`<addr>@sth`) sends are capped at a single recipient** - mixing
  multiple `@sth` peers, or an `@sth` peer with a classic e-mail, in the
  same compose is rejected with an inline error before any transport
  fires. This keeps the NTFRY envelope semantics honest: one sealed
  envelope, one target NodeId, one Iroh stream. Multi-address blasts stay
  on the SMTP path.
- **Naming error copy** - the naming module now returns a dedicated
  `Reserved` error variant (formerly folded into the generic invalid-name
  bucket) so the UI can render a distinct red hint referencing
  `reserved.txt`.

### Fixed
- **1–2 char `.sth` names were purchasable by ordinary users** - the domain
  registration panel showed `available: ok` (with the correct `adminOnly`
  flag set, but never enforced) for 1–2 character labels, letting anyone
  reach the payment confirmation for e.g. `ab.sth`. Such names are reserved
  for `DEVELOPERS` only. Now enforced at all three layers in
  `frontend/src-tauri/src/dns_manager.rs`: `domain_lookup` returns
  `reason: reserved` for non-developers (UI shows RESERVED, no buy button),
  `register_domain` rejects the command, and the on-chain indexer ignores
  1–2 char `reg:` transactions from non-developer senders. The gate also
  widened from the single `CONFIG_ADMIN` address to the full `DEVELOPERS`
  list (compile-time constant + runtime `.env`). The web-preview mirror
  (`frontend/src/lib/bridge.ts` → `webDomainLookup`) applies the same rule.
- **`mail-core` compile error `E0603: function finish_progress is private`**
  - the progress-finalisation helper introduced in 0.6.0 was left with
  default (private) visibility while a sibling module needed to invoke it
  after the SMTP fallback path resolved. It is now `pub(crate)`, which
  restores a clean build across the workspace without widening the public
  API surface. No behaviour change - the same "100 % only after the ack /
  SMTP success" contract still holds.
- **Empty screen on open message in Oxid Mail (`ReaderView.vue`)** - Vue 3
  requires a single root node when the template is compiled with the
  runtime + compiler split we use; a stray sibling comment/element next to
  the root `<article>` slipped in during the 0.6.0 attachment-overlay work
  and caused the reader to render nothing on message click (silent mount
  failure). The template is back to a single root, so reading e-mail works
  again. No API or storage change.

### Security / Privacy
- The unverified-dApp acceptance record is stored **only locally** per
  wallet address; nothing about which dApps a user chose to unlock leaves
  the device. The on-chain footprint is exactly one transfer with a
  well-known `vendorField` marker - no metadata about the user's browsing
  history is ever broadcast.
- The `DEVELOPERS` allowlist and `FEE_ALLOW_ALL_DAPPS` flag are read from
  `frontend/.env` at build/launch time and are never exposed to the dApp
  sandbox. They cannot be flipped by a hostile dApp at runtime.
- The `reserved.txt` list ships with the client and is validated on both
  the UI and the Rust naming module, so a compromised or patched UI cannot
  bypass the guard on its own.

### Notes
- Component version bumps for this release:
  - SmartNet client → **1.122.0**
    (`frontend/package.json`, `frontend/src-tauri/Cargo.toml`,
    `frontend/src-tauri/tauri.conf.json`).
  - Oxid Mail workspace → **0.7.0**
    (`oxid-mail/src-tauri/tauri.conf.json`,
    `oxid-mail/src-tauri/Cargo.toml`,
    `oxid-mail/crates/mail-core/Cargo.toml`,
    `oxid-mail/crates/mail-crypto/Cargo.toml`,
    `oxid-mail/crates/mail-store/Cargo.toml`,
    `oxid-mail/crates/oxid-bridge/Cargo.toml`,
    `oxid-mail/ui/package.json`).

---

## [1.121.0 / Oxid Mail 0.6.0] - 2026-07 (Attachments Dropzone + inline NTFRY, attachment preview overlay, honest P2P delivery progress)

### Added
- **Compose Dropzone with a 25 MB total-size limit** (`ui/src/components/ComposeModal.vue`) -
  drag-and-drop overlay ("Drop files to attach"), footer button and file
  counter `N files · X MB / 25 MB`, per-file remove. Files are read as
  base64 via `FileReader.readAsDataURL` and passed to the existing
  `POST /accounts/:id/send { attachments: [{name, mime, dataBase64}] }` API.
- **Server-side 25 MB guard** in `mail-core::service::send_message`: the sum
  of decoded attachment bytes is checked before sealing/sending, returning a
  human-readable `CoreError::Msg` on violation. Mirrors the UI cap and
  protects P2P delivery from bloating a single envelope past SMTP quotas.
- **Inline attachments in the NTFRY envelope** - for P2P (`<addr>@sth`) and
  SMTP-encrypted flows, all files go **inside** the sealed envelope
  (`envelope::Attachment { filename, mime, data_b64 }`). No separate
  `sn://file/` payloads, no side-channel: the recipient unseals a single
  ciphertext and gets all files at once. Same code path works for the
  SMTP-fallback case.
- **P2P attachment activity toast** - after a successful send to `<addr>@sth`
  with attachments, a new Pinia store `stores/toast.js` + `ToastHost.vue`
  surface a persistent notification that the encrypted envelope is being
  seeded via Iroh while `oxid-mail` is open; if the peer is offline, the
  envelope will be delivered as soon as both sides come online.
- **Attachment preview overlay in `ReaderView.vue`** - clicking an
  attachment chip now opens a full-screen preview modal on top of the mail
  window. Images are rendered via `<img>`; PDFs via an `<iframe>` fed a
  `blob:` URL. Data is streamed straight into memory: inline NTFRY payloads
  reuse the base64 from `dec.attachments`, SMTP payloads are pulled through
  `GET /accounts/:id/messages/:mid/attachments/:idx` - **without ever
  touching the disk**. Buttons **Download** and **Close**, Esc key and
  backdrop click all dismiss the overlay; `URL.revokeObjectURL` is called on
  close, on message switch and on unmount. All other MIME types keep the
  legacy click-to-download behaviour.
- **Honest "file X of Y" P2P delivery progress** - end-to-end pipeline:
  - `mail-core::p2p::send_with_progress` slices the NTFRY payload into
    **256 KB QUIC chunks** and updates a shared
    `PROGRESS: Mutex<HashMap<progress_id, ProgressState>>` after each chunk.
    The `sent` counter is kept one byte below `total` on purpose - the final
    100 % is only set by `finish_progress` after the receiver's `OK` ack, so
    the UI never shows a fake "done".
  - `SendRequest` gained a `progressId: Option<String>` field
    (camelCase serde). The service layer registers the progress with the
    original file sizes, invokes `send_with_progress`, and finalises the
    state with `ok=true` on either P2P success or SMTP fallback, or
    `ok=false` on complete failure.
  - New transport surface: `GET /api/p2p/progress/:pid` on `oxid-bridge` and
    a matching Tauri IPC command `p2p_progress`. Wired into `lib/backend.js`.
  - The compose flow generates `crypto.randomUUID()` before sending, pushes
    a persistent (`ttl=0`) toast with `progress=0`, and polls the endpoint
    every **500 ms**. It recomputes "file X of Y · N%" by mapping
    `sent/total` of the payload back onto the cumulative sum of the
    original file sizes. When the send resolves, the same toast morphs into
    a success - **"✓ all N files delivered"** for pure P2P, or
    **"Delivered via SMTP"** when the Iroh fallback fired - with `ttl=6000`.
    On send error the toast is dismissed cleanly.
  - `stores/toast.js` now supports `progress: number|null` and an
    `update(id, patch)` method that safely reschedules the `ttl` timer.
    `ToastHost.vue` renders a copper/emerald progress bar only when the
    item has a `progress` field.

### Changed
- `p2p::send` is now a thin wrapper over `send_with_progress(None)` - one
  code path for both the progress-instrumented compose flow and any
  ad-hoc/legacy caller.
- `LRU-style trim in `PROGRESS`: the map self-cleans finished entries once
  it grows past 64 keys, so long sessions never leak progress state.
- Attachment chip in the reader shows a **👁 eye** glyph for previewable
  MIMEs (`image/*`, `application/pdf`) and the classic **⇩** for the rest,
  making the two actions visually distinct without extra buttons.

### Security / Privacy
- The preview overlay never persists attachment bytes to disk - data lives
  only in a `Blob` and its `blob:` URL, which is revoked on close/unmount
  and on any message switch. Same guarantee applies to inline NTFRY
  attachments (already in memory only) and to SMTP attachments fetched on
  demand.
- The progress channel exposes only `{ sent, total, fileSizes, done, ok }`
  - no message body, subject, recipient or key material is ever surfaced
  over the progress endpoint.

### Notes
- Chunk size (256 KB) is a deliberate compromise between smooth UI
  animation (~4 progress ticks per MB) and QUIC head-of-line efficiency;
  it does not change the on-wire format - the ciphertext is still one
  contiguous NTFRY envelope streamed over a single bi-directional Iroh
  stream, exactly as before.

---

## [1.120.0 / Oxid Mail 0.5.0]  (Iroh P2P delivery, external links, one-click sender contact, QR share)

### Added
- **Iroh P2P transport (`mail-core::p2p`)** - an iroh 1.0 endpoint (preset N0:
  n0 relays + DNS/pkarr discovery, QUIC hole-punching) starts with the session;
  the node key is seed-derived (`oxid-mail/iroh-node/v1`), ALPN
  `oxid-mail/p2p/1`. Sending to `<SADDR>@sth` seals a NTFRY envelope (keys from
  the local contact first, registry fallback), dials the recipient's NodeId
  (8s timeout) and waits for an `OK` ack; if the peer is offline, an automatic
  **SMTP fallback** delivers the same armored envelope to a local contact that
  shares the STH address and has a regular e-mail. Incoming P2P messages are
  verified (envelope signature), stored under the new **p2p folder** (sidebar
  entry added) and decrypt through the standard NTFRY reader flow with
  TRUSTED CONTACT verdicts.
- **External links open in the default browser**: plain-text bodies are
  linkified (clickable http/https), HTML bodies keep the sandboxed iframe but
  clicks are intercepted (`allow-same-origin`, scripts still blocked) - both
  routes go through the new `open_external` Tauri command (`open` crate) with
  a `window.open` fallback in web mode. Only http/https URLs are allowed.
- **One-click "add sender to contacts"** button next to the sender in the
  reader: if the sender is verified (NTFRY or X-Ntfry-Sig), the contact is
  pinned with their STH address (short form `ADDR:email`); otherwise a plain
  contact is stored. A green check replaces the button when already in the book.
- **QR code in "Share my contact"**: every share row (plain e-mail / encrypted
  URI / p2p address) gets a QR toggle for in-person exchange (client-side
  `qrcode` package, no network).

### Changed
- Oxid Mail workspace version: **0.4.0 → 0.5.0**; new deps: `iroh 1`, `tokio`
  (mail-core), `open 5` (Tauri), `qrcode` (UI).

### Verified
- 21/21 unit tests, `cargo check` clean (mail-core + bridge + Tauri), Vite
  build clean. Runtime P2P/link/QR flows are user-tested locally by request.

---

## [1.120.0 / Oxid Mail 0.4.0]  (Contacts module: Local Trust Layer over blockchain discovery)

### Added
- **Local address book (`mail-core::contacts`)** - Web-of-Trust style direct
  contact exchange. Three contact kinds: **plain** (classic e-mail),
  **encrypted** (e-mail + pinned STH address + X25519/Iroh public keys),
  **p2p** (internal `<sthAddress>@sth`). Stored in the encrypted sled tree
  `contacts`.
- **Local Explicit Trust beats the chain**: when sending encrypted mail, keys
  from a local contact are used directly and the blockchain registry is NOT
  queried; a short-form contact (`SADDR:email`, address pinned without keys)
  fetches keys from the chain but ABORTS the send if the on-chain address does
  not match the pinned one (anti-squatting guard). Cold contacts still resolve
  through the chain registry (discovery layer).
- **Sender verification priority**: incoming NTFRY/X-Ntfry-Sig verdicts check
  the local address book first (`source: local`), falling back to the chain
  (`source: chain`). New reader badges: ✔ TRUSTED CONTACT / ⚠ CONTACT MISMATCH.
- **Contact exchange formats** parsed by `contacts::parse_any`: full URI
  `sn://add-email?v=1&email=…&addr=…&x=…&node=…&name=…`, short form
  `SADDR:email`, p2p `SADDR@sth`, plain e-mail.
- **Share card** (`contacts::my_card`): three ready-to-send strings - classic
  e-mail, encrypted contact URI (keys derived from seed), p2p `@sth` address.
- **API**: `GET/POST /api/contacts`, `POST /api/contacts/parse`,
  `DELETE /api/contacts/:email`, `GET /api/accounts/:id/contact-card` +
  mirrored Tauri commands (`contacts_list`, `contact_add`, `contact_parse`,
  `contact_remove`, `contact_card`).
- **UI (Mailbox header, after the search bar)**: “Add contact” button →
  modal with smart input (auto-detects all four formats, live preview with
  kind chip, name/note fields) and the address book list with delete;
  “Share my contact” button → modal with the three shareable rows, each with
  a copy button.
- Guard: sending to `<addr>@sth` returns a clear message that direct Iroh
  delivery arrives in the next phase.

### Changed
- `DecryptedView` / `VerifyResult` gained `source` (`local | chain | none`).
- Oxid Mail workspace version: **0.3.0 → 0.4.0**.

### Verified
- 12/12 `mail-core` unit tests (incl. new contact parse/URI roundtrip tests);
  `cargo check` clean for bridge + Tauri; Vite build clean.
  Runtime flows are user-tested locally by request.

---

## [1.120.0 / Oxid Mail 0.3.0]  (Encrypted mail protocol v1: NTFRY envelope, email↔STH binding, Anti-Spoofing)

### Added
- **New crate `mail-crypto`** - frozen protocol v1 primitives:
  - Registry index hash: `Argon2id(m=128MiB, t=3, p=1)` over normalized e-mail
    with a global BLAKE3-derived salt, finished with keyed BLAKE3 → `H32`;
    on-chain index is `H16 = H32[..16]`, envelope slot hint is a
    domain-separated 8-byte BLAKE3 derivation (unlinkable to `H16`).
  - Seed-derived keys (HKDF-SHA256 from `sha256(mnemonic)`): X25519 static key
    (`oxid-mail/x25519/v1`) and Iroh ed25519 NodeId (`oxid-mail/iroh-node/v1`) -
    restoring the wallet restores all mail crypto, no separate key backup.
  - **NTFRY envelope**: ECIES (ephemeral X25519 → HKDF → XChaCha20-Poly1305) +
    zstd, multi-recipient CEK wrapping, secp256k1 ECDSA signature over the whole
    payload; real subject/attachments hidden inside the ciphertext. ASCII armor
    with `-----NTFRY START/END-----` markers and a human-readable notice for
    classic mail clients.
  - **X-Ntfry-Sig** header: detached signature (from|to|subject|ts|blake3(body))
    for anti-spoofing badges on PLAIN (unencrypted) mail.
  - Registry vendorField codec: `oxid1:<H16>:<X25519>:<IrohNodeId>` (116 chars,
    base64url) and `oxid1:revoke:<H16>`.
- **`mail-core::chain`** - autonomous SmartHoldem chain client: node pool with
  failover, balance/nonce, legacy bip-schnorr signing (byte-identical to
  SmartNet `sth_node.rs`), v2 transfer build/broadcast, tx confirmation polling,
  and the **registry indexer** with deterministic rules: earliest-wins per H16,
  key rotation only by the owner address, revoke frees the hash, foreign late
  bindings flagged `conflict`.
- **`mail-core::crypto_mail`** - service layer: cached Argon2 e-mail hashes,
  persistent registry index (sled), **Proof-of-Ownership binding flow**
  (balance check → signed proof e-mail to self via SMTP → IMAP verify with
  DKIM presence check → 0.5+0.5 STH transfer to `SMaiL…` registry address →
  chain confirmation), NTFRY decrypt with sender verification verdicts
  (`verified` / `mismatch` / `unknown` / `invalid`) and plain-mail
  X-Ntfry-Sig verification.
- **Send path** (`service::send_message`): `encrypt` flag - recipients resolved
  through the on-chain registry (clear error if a recipient is not bound), the
  sender always gets its own decryption slot; plain mail from bound accounts is
  automatically signed with `X-Ntfry-Sig`.
- **Bridge/Tauri surface**: `POST/GET /api/accounts/:id/bind`,
  `POST /api/registry/sync`, `POST /api/registry/lookup`,
  `POST …/messages/:mid/decrypt`, `GET …/messages/:mid/verify` and matching
  Tauri commands (`account_bind`, `account_bind_status`, `registry_sync`,
  `registry_lookup`, `message_decrypt`, `message_verify`).
- **UI**: Settings → "EMAIL ↔ STH BINDING" wizard with live status chips
  (NOT BOUND / PROVING / WAITING CHAIN / BOUND / CONFLICT), compose
  "Secure with SmartHoldem" toggle is now LIVE (was "Stage 2 · coming soon"),
  reader unseals NTFRY mail with sender STH address + authenticity badges
  (✔ VERIFIED SENDER / ⚠ SENDER MISMATCH / ⚠ BAD SIGNATURE / · UNREGISTERED),
  encrypted attachments download from inside the envelope.

### Changed
- `StoredMessage` gained `ntfrySig`; IMAP parser now flags NTFRY messages as
  `encrypted` and extracts the `X-Ntfry-Sig` header.
- `smtp_client::send_mail` accepts extra raw headers.
- Oxid Mail workspace version: **0.2.0 → 0.3.0** (all crates + Tauri + UI).

### Verified (container E2E, real privateemail.com mailbox)
- 12 unit tests in `mail-crypto` + `mail-core::chain` (envelope roundtrip,
  tamper detection, registry rules, schnorr sign/verify, live registry sync).
- Full NTFRY loop over a real mail server: seal → SMTP → IMAP sync (auto-detect
  `encrypted`) → decrypt endpoint → hidden subject/body recovered, sender STH
  address extracted. Binding flow verified up to the graceful
  "insufficient STH balance" guard (the on-chain transaction itself requires a
  funded wallet - user-side test).

---

## [1.120.0]  (Oxid Mail 0.2.0 · Gmail App-Password-only sign-in, custom seed passthrough, sled lock fix)

### Added
- **Oxid Mail: "Connect SmartNet" onboarding shortcut** - the mail app now
  detects an active SmartNet session over the local IPC channel and offers a
  one-click sign-in that skips the 12-word input entirely. If SmartNet is
  running, the user never sees the raw seed prompt again
  (`oxid-mail/ui/src/views/Onboarding.vue`).
- **Docs: encrypted Sled store rationale** - `oxid-mail/README.md` now
  documents the local store threat model (BIP-39 / custom secret → HKDF-SHA256
  → XChaCha20-Poly1305 + zstd, sled) and the reasoning behind the
  PIN-derived unlock flow.

### Changed
- **`mail-core::identity`: accept arbitrary secrets, not only BIP-39** - the
  SmartNet wallet may emit non-BIP-39 strings for advanced users, so the
  identity layer now uses `from_secret(...)` (raw KDF over the string bytes)
  as the primary path. `from_mnemonic(...)` is still available for wallets
  that generate real BIP-39 phrases. The onboarding "invalid BIP-39 mnemonic"
  error class is gone.
- **Gmail login is App-Password-only** - the OAuth "Sign in with Google"
  button has been removed from onboarding to eliminate a confusing UX where
  Google would silently reject headless OAuth. Onboarding now shows a
  step-by-step guide to generating a Google App Password and pasting it into
  the standard IMAP/SMTP account form.
- **SmartNet Mail launcher: `mail_launcher.rs`** - the child process lifecycle
  is now idempotent (repeated "Open Mail" clicks reuse the running instance
  and re-focus its window instead of spawning duplicates).

### Fixed
- **Sled lock error on re-open** - `oxid-bridge` did not always drop the
  previous `sled::Db` handle before `POST /api/session/open` fired again,
  causing `IO error: could not acquire lock`. The bridge now closes the
  previous session (and its DB handles) synchronously in
  `POST /api/session/close`, and `session_open` waits for the file lock to
  clear before re-opening (`oxid-mail/crates/oxid-bridge/src/main.rs`).
- **HTML body vanished during background IMAP IDLE refresh** - a diff-merge
  on the message cache dropped the parsed body when the server pushed an
  updated flag-set for an already-cached UID. The cache now preserves the
  full body when only metadata changes.

### Security / Privacy
- Removing Gmail OAuth eliminates the browser-based redirect surface for the
  Mail client. Only user-provided App Passwords (which the user can revoke
  independently on their Google account) touch the wire, and they never
  leave the encrypted local Sled store.

### Notes
- Component version bumps for this release:
  - SmartNet client → **1.120.0**
    (`frontend/package.json`, `frontend/src-tauri/Cargo.toml`,
    `frontend/src-tauri/tauri.conf.json`).
  - Oxid Mail workspace → **0.2.0**
    (`oxid-mail/src-tauri/tauri.conf.json`,
    `oxid-mail/src-tauri/Cargo.toml`, `oxid-mail/crates/mail-core/Cargo.toml`,
    `oxid-mail/crates/mail-store/Cargo.toml`,
    `oxid-mail/crates/oxid-bridge/Cargo.toml`,
    `oxid-mail/ui/package.json`).

---

## [1.119.0]  (Dashboard Mail badge + unread counter over IPC)

### Added
- **Mail card unread badge in the SmartNet Dashboard** - the "Mail" tile now
  shows a live badge with the number of unread messages, sourced from Oxid
  Mail over the local IPC channel. The badge listens to
  `mail://unread-updated` events pushed by `oxid-bridge` and stays consistent
  across window focus changes (`frontend/src/store/mailBadge.ts`,
  `frontend/src/views/Dashboard.vue`).
- **`mailBadge.ts` Pinia store** - small dedicated store so any view can
  subscribe to the unread counter without importing the whole IPC layer.

### Changed
- **Dashboard IPC listeners consolidated** - the Mail card, the unread badge
  and the "Open Mail" action now share a single IPC subscription lifecycle,
  so switching between routes no longer leaks Tauri event handlers.

### Notes
- Wire format between Oxid Mail and SmartNet for the unread counter:
  ```json
  { "type": "mail://unread-updated", "unread": 7, "account": "user@…" }
  ```
- No component versions were bumped for this milestone; it landed as part of
  the ongoing SmartNet 1.119.x line.

---

## [1.118.0]  (Mail integration: IPC seed handoff + IMAP IDLE system notifications)

### Added
- **Seamless IPC handoff of the SmartNet seed** - when the user launches Oxid
  Mail from the SmartNet dashboard, SmartNet passes the active seed over a
  private local IPC channel (stdin + short-lived HTTP diagnostic on the
  loopback bridge). Oxid Mail derives the same identity SmartNet uses,
  without the user ever seeing or typing a 12-word phrase
  (`frontend/src-tauri/src/mail_launcher.rs`,
  `oxid-mail/crates/oxid-bridge/src/main.rs`).
- **System notifications on IMAP IDLE push** - Oxid Mail now surfaces a
  native OS notification when the IMAP server pushes a new message via
  `IDLE`, including sender + subject snippet. Notifications are throttled
  per-account to avoid burst-storming after long offline periods.
- **Bridge diagnostic endpoints for the handoff** - new internal, loopback-
  only routes on `oxid-bridge`:
  - `/__diag?cmd=request_smartnet_seed` - the mail app asks SmartNet for a
    seed (SmartNet grants only if the request came from a child process it
    just spawned).
  - `/__diag?cmd=handoff_take` - one-shot consumption of the handoff token;
    the token is deleted from memory after the first successful read.

### Changed
- **Mail-core session bootstrap** - accepts an already-derived key from the
  IPC channel and skips the manual BIP-39 form when the caller is trusted
  (i.e., SmartNet's own process).

### Security / Privacy
- The handoff surface is bound to `127.0.0.1` and to a per-launch nonce; the
  token is only valid for the exact PID that SmartNet spawned, is single-use
  and expires within seconds. It never touches the network or disk.

## [1.117.0]  (vDapps status panel + Official Repository doc)

### Added
- **Seeders page → vDapps panel**: trusted developers now see when and what
  was announced in `vdapps.enc` - publish timestamp, developers count,
  official dApps count and known dApp seeders count (new `vdapps_status` IPC;
  the publish commit now also stores an announce summary in Sled).
- **docs/14-Official-Repository.md**: rationale for the two-tier catalog
  (official | all) - why it protects regular users from malicious content
  without violating decentralization or free content distribution (mark,
  don't censor; on-chain transparent lists; client-side filtering; direct
  links never filtered), including the vDapps cloud-seeder fallback and its
  trust chain.

## [1.116.0]  (Avatar flash fix, Update All + notifications, vDapps cloud fallback)

### Fixed
- **Stale avatar on profile switch**: opening another user's profile briefly
  showed the previous profile's avatar. `loadAvatar()` now resets the shown
  avatar immediately, paints the target user's cached avatar (users store /
  localStorage) and guards async responses against route races.

### Added
- **UPDATE ALL** button in the Installed screen's update banner - applies all
  pending app updates sequentially.
- **System notifications for updates**: when a background scan discovers new
  app versions, a native OS notification lists them
  (`@tauri-apps/plugin-notification`; deduped per `id@version` in
  localStorage, so each update notifies once).
- **vDapps cloud fallback (PUBLIC NETWORK SEEDERS)**: trusted developers
  (wallet ∈ DEVELOPERS) publishing their network state now also announce
  `vdapps.enc` - the official developers list, approved dApps and known
  seeder nodeIds of official apps. Publish is gated by a content hash over
  the stable lists (devs+dapps), committed only after a successful upload, so
  the 30-min auto-replication never re-uploads unchanged data. On fetch,
  peers WITHOUT blockchain access import these lists - but only when the
  publisher's address is already in their known DEVELOPERS set (trust chain
  anchored at the .env admin seed). `SEED_FILES` grew to 4 entries
  (backward compatible: missing file → null).

## [1.115.0]  (Update badge in sidebar + manual Official rescan)

### Added
- **Sidebar update badge**: the "INSTALLED / My Applications" nav item now
  shows an amber `⟳N` badge whenever the cloud/P2P network advertises a newer
  version of an installed app (the Installed screen itself already had the
  banner + per-card update tag; update detection works for both live P2P and
  Cloud Seeder fallback since versions travel in seeder listings).
- **Settings → Official Repository panel**: shows trusted-developer count,
  approved-dApps count and the last on-chain scan time, plus a
  "REFRESH OFFICIAL LIST NOW" button (`official_refresh` IPC) that rescans
  both registries immediately - no need to wait for the 24h cycle - and
  re-marks the DISCOVER catalog on the spot.

## [1.114.0]  (Official status in app details + build fixes)

### Added
- **App details modal**: new "VERIFICATION" row - green "✔ Official
  Repository" for verified apps, amber "⚠ Not verified - install at your own
  risk" otherwise; new "DEVELOPER" row shows the signed owner address as a
  link to the developer's profile (`/profile/<owner>`).

### Fixed
- Restored two edits lost during code sync (compile errors E0425/E0063):
  the mDNS `o{i}` owner-records collection in the browse parser and
  `owner: m.owner` in `build_tracker_snapshot` (`p2p.rs`).

## [1.113.0]  (Messenger fix + signed owner canon v2)

### Fixed
- **Messenger black screen**: `ChatWidget.vue` referenced `useUiStore()`
  without importing it (lost during the 1.108.0 messenger redesign) - the
  setup crashed with `ReferenceError` and the native chat window rendered
  black. Import restored; window renders again.
- **Build breakage**: stray duplicated code fragments at the end of
  `src-tauri/src/p2p.rs`, `tracker/src/main.rs` and `seeder/src/main.rs`
  (unexpected closing delimiter) removed; all three files parse cleanly.
- Version sync: `Cargo.toml` was stuck at 1.108.1 while package.json moved on.

### Security
- **Signed owner (canon v2)**: the publish/update signature now covers the
  developer address - `sha256(id|name|description|merkle|owner)`. The Official
  badge can no longer be forged by attaching a foreign `owner` to an announce:
  - client `verify_app` accepts v2 with a legacy (no-owner) fallback, so
    already-installed apps stay valid until their next republish;
  - tracker `verify_announce` returns whether the owner is signature-backed;
    legacy announces are still accepted but their unsigned `owner` claim is
    dropped (no Official-by-owner, only explicit id approval applies);
  - new unit tests: legacy+spoofed owner → untrusted; v2 owner → trusted;
    tampered v2 owner → rejected.
    Note: mDNS LAN announces remain unsigned by design (advisory only).

## [1.112.0]  (Official Repository: verified dApps by default)

### Added
- **Official Repository** (`src-tauri/src/official.rs`): the client scans two
  on-chain SmartHoldem registries on boot + every 24h (10-min retry while
  offline) through the wallet's failover node pool (`sth_node::node_get`,
  never a single hard-coded node):
  - `SdevsH3kMbcCFWnyYVtD2UYnBQQm66MxDn` → trusted DEVELOPERS (vendorField =
    developer address, sender must be the admin wallet);
  - `SdAppjWaQf4v2JjuubvtwsKe3yuUCuq8i8` → approved OFFICIAL_DAPPS
    (vendorField = appId).
    Results persist in Sled (`official:dev:*` / `official:dapp:*`). `.env`
    fallback `DEVELOPERS=` keeps a seed list when the blockchain is down;
    registry addresses are overridable via `OFFICIAL_DEVELOPERS` /
    `OFFICIAL_DAPPS`. New IPC: `official_lists`, `official_refresh`.
- **Owner propagation**: dApp announces now carry the developer address
  (manifest `owner`) end-to-end - mDNS TXT (separate backward-compatible
  `o{i}` records), tracker announce/directory (`owner` field on
  `TrackerAnnounceApp`/`TrackerApp`/`AppOut`), and Cloud Seeder listings
  (already included `owner`). An app is **official** when
  `owner ∈ DEVELOPERS` OR `id ∈ OFFICIAL_DAPPS`.
- **DISCOVER APPS filter**: the store search shows ONLY verified (official)
  dApps by default in every UI mode. DEV/GEEK modes get a "Show unverified"
  toggle (persisted, `sth_show_unverified`) with an explicit warning banner;
  USER mode is always official-only. Verified cards show a green ✔ badge.
  Direct `sn://` links keep working unfiltered (mark, don't censor).
- **Tracker** (`/app/tracker`): same 24h registry scanner (sled-persisted),
  `/apps` now exposes `owner` + `official` per app - marking only, nothing is
  removed from the index.
- **Headless Seeder** (`/app/seeder`): relays the manifest `owner` in tracker
  announces and runs the same 24h registry scanner (logged; seeding itself
  stays uncensored).

### Security note
- The `owner` field in announces is not yet covered by the announce
  signature; the strongest guarantee comes from `id ∈ OFFICIAL_DAPPS`
  (id ↔ pubkey binding) and the post-install signed manifest.

## [1.111.0]  (Comment cache: instant thread expand)

### Added
- **Comment cache** (`store/wallCache.ts`): comment threads are now cached per
  author/post inside the same `sn.wallCache.v1` localStorage payload (up to
  100 comments per post). Re-expanding a thread renders instantly from cache
  while a silent background `listComments` refresh brings in new replies.
  Adding/hiding a comment updates the cache immediately; comments of posts
  evicted from the feed cache (LRU) are pruned automatically.

## [1.110.0]  (Avatar cache everywhere: messenger, wall & comments)

### Added
- **Persistent avatar cache** (`store/users.ts`): avatar data-URLs are now
  persisted to localStorage (`sn.avatars.v1`, LRU 30 addresses, items > 96 KB
  skipped, quota-exceeded fallback shrinks to 10). Avatars paint instantly on
  app start - before a single IPC call - eliminating the Identicon flash in
  the messenger, contact list and reaction tooltips. A fresh blob is still
  fetched in the background (once per session per address).

### Changed
- **Wall post headers & comment avatars** migrated from per-CID media fetch
  (`fetch_wall_media`) to the shared `UserAvatar` component - the same instant
  cache now serves the wall, comments and the messenger uniformly, and avatar
  updates propagate everywhere at once.

## [1.109.0]  (Wall post cache: instant profile open)

### Added
- **Wall post cache** (`store/wallCache.ts`): opening a profile now renders
  instantly from cached posts (localStorage `sn.wallCache.v1`, LRU up to
  20 authors × 50 posts), then silently checks for updates in the background.
  A thin accent progress bar (`wall-bg-refresh`) indicates the silent refresh -
  no spinner, no feed flash.
- **Session media cache**: wall images (data-URLs) and audio/video object-URLs
  are now cached in the Pinia store keyed by CID (content-addressed →
  immutable), so navigating away and back to a profile no longer repeats the
  IPC round-trip and base64 → blob conversion. On-disk byte caching was already
  handled by Rust (`social_cache/`, `filesdata/`), so nothing is re-downloaded
  from the network either.

### Fixed
- **WallFeed crashed on mount**: `mergeNew`, `loadMore`, `loadingMore`,
  `sentinel` and `feedObserver` were referenced but never defined (lost in a
  previous refactor) - the component threw `ReferenceError` on mount. All
  restored: live-event merge keeps paginated posts intact, infinite scroll
  works again via the bottom sentinel + `IntersectionObserver`.

## [1.108.1]  (Fix: Storages & Media cards melting into the background)

### Fixed
- **Torrents / My Files / Music cards on the Storages page** blended into the
  page background on DEFAULT themes: `.st-card` uses `var(--surface)`, and the
  `--surface` variable was never defined anywhere (transparent), while DEFAULT
  themes also made `--border` transparent - so the cards lost both fill and
  outline. `--surface` is now defined in both DEFAULT themes (`#2D2E31` dark /
  `#FFFFFF` light) and the cards get the standard elevation shadow + 12px
  radius. Legacy themes are untouched (they still show the bordered look).

## [1.108.0]  (Telegram-round chat bubbles + human copy in USER mode)

### Changed
- **Chat bubbles are rounder, Telegram-style:** base radius 16→18px, softer
  joined-corner radii inside message groups (tail corners preserved).
- **"Annihilate logs" is USER-friendly now:** in USER mode the button reads
  "Clear history" with a plain-language confirmation
  ("Chat history will be permanently deleted from this device");
  DEV/GEEK keep the brutal 💀 wording.
- **Human copy pass for USER mode (all three locales):**
  message input placeholder "Message (visible only to participants)…" instead of
  "Encrypted message (E2EE)…"; network-features list rewritten from benefit
  perspective - Social: "posts cannot be deleted or blocked by anyone",
  Messenger: "messages go directly, without servers or intermediaries" (no more
  iroh-gossip), Storage: "everything is stored with you and always at hand".
  DEV/GEEK still see the full technical descriptions.

## [1.107.1]  (Light theme: USER/DEV/GEEK switcher fixed)

### Fixed
- **Mode switcher on DEFAULT LIGHT:** the pill container used a dark
  `rgba(0,0,0,.28)` film (looked like a grey blob) and the active mode was
  pale tinted text with poor contrast. Now the container is a white card with
  a soft shadow, and the active mode is a solid filled chip with white text -
  green for USER, blue for DEV, amber for GEEK (Adminex-style).

## [1.107.0]  (Omnibox search from the address bar + store card covers)

### Added
- **The address bar now works like Chrome's omnibox.** Typing a plain name or
  phrase without a protocol (`cosmos`, `cosmos sandbox`) opens the built-in
  app-store search (`// // DISCOVER APPS_`) with the query pre-filled.
  Precedence is preserved: `sth:// api:// u:// f:// dev:// sn:// cdn://` links
  and magnet links keep their behaviour; a bare word is first checked against
  app ids (S-address) and the `.sth` name registry - only when nothing matches
  does it fall through to search. `sth://`-prefixed unknown domains still stay
  in the bar (no silent search).
- **Store cards got the showcase covers** - the same deterministic colour
  gradient (from the app id) as the Dashboard top-apps, flush with the card
  top, with the app logo or identicon centred on it (shared helper
  `lib/coverArt.ts`).

### Fixed
- USER-mode green online dot on store cards (instead of the "● N seeds"
  counter) - the earlier change had been lost in a later edit; re-applied.

## [1.106.0]  (DEFAULT LIGHT restyled after the Adminex reference)

### Changed
- **DEFAULT LIGHT now follows the admin-dashboard reference:** soft blue-grey
  canvas `#f0f2f7`, pure-white cards with a gentle layered shadow, white
  sidebar and top bar (the tab strip stays grey so the active white tab reads
  like Chrome), and ~10px rounding for buttons and text inputs instead of
  pills (`--radius-btn: 10px`; tile "Launch →" buttons are 10px in light,
  still pill in dark).
- **"Network features" rows fixed on light:** they used a hard-coded dark
  `rgba(0,0,0,.18)` film designed for dark themes and looked muddy - now
  white cards with a light hover.

## [1.105.1]  (Fix: blank Dashboard after leaving Profile; store buttons less round)

### Fixed
- **Blank page when navigating Profile → Dashboard.** The router view was
  wrapped in `<transition mode="out-in">`: the incoming page waited for the
  outgoing (heavy Profile) view's leave animation to finish, and when that
  leave hook stalled the next view never mounted - an empty client area.
  The transition no longer serialises enter/leave (the leaving view is taken
  out of flow with `position: absolute`), so the next page always mounts
  instantly. Verified with 12 rapid Profile/Store ↔ Dashboard round-trips -
  no blank state.
- **Global Vue error handler** (`[vue-error]` in the console) so any future
  view render failure is visible instead of a silent blank page.

### Changed
- **App-store buttons are less round** on DEFAULT themes (~10px, per the
  reference), instead of the global pill shape.

## [1.105.0]  (Always-visible day/night toggle, showcase top cards, Stealth tile)

### Changed
- **☀/☾ toggle is now always visible** between the ♫ and ⚙ icons, on every
  theme. Clicking it switches straight to DEFAULT DARK / DEFAULT LIGHT - even
  if a custom theme (Focus, Neon, …) is active, one click brings you to the
  new default look. (Previously the button only appeared while a DEFAULT theme
  was already selected, which made it undiscoverable.)
- **"Popular on the network" cards became a showcase.** Each top-app card now
  has a colourful cover - a deterministic gradient derived from the app id -
  with the app logo (or identicon) centred on it, and the name + online dot
  underneath. Three covers in a row read like a storefront.
- **Stealth Mode moved into "// SITES YOU SEED_"** as the sixth block
  (DEV/GEEK only), matching the tile grid instead of floating in the banner.

## [1.104.0]  (Dashboard polish: omnibox highlight, top apps, NameService tile, first-visit hints)

### Added
- **Top-3 network apps under the search box.** The freed space below the
  Dashboard omnibox now shows "Popular on the network" - the three network apps with
  the most seeders as compact cards (logo/identicon + name + green online dot;
  DEV/GEEK also see the seeder count). Click opens the app if installed,
  otherwise jumps to the store with the search pre-filled.
- **First-visit hints.** A gentle dismissible one-time hint banner (💡-style,
  accent-tinted, "OK" button) on the first open of the app store
  ("This is the app store…") and the cloud/storages page ("This is your
  cloud…").

### Changed
- **NameService moved into "// SITES YOU SEED_"** as the first tile, restyled
  to match the service tiles (`// NAMES` tag, names-in-network status, your
  `u://name` or a "register name" action) - the banner is now lighter.
- **Search box is impossible to miss:** elevated surface with a soft teal
  ring, brighter ring + shadow on hover/focus (Google omnibox behaviour).
- **"Network features" block stands out:** lighter elevated background, subtle
  border and a real shadow - its edges no longer melt into the banner.
- **Tile "Launch →" buttons are pill-shaped** on DEFAULT themes.
- Note: the ☀/☾ toggle has been between the ♫ and ⚙ icons since v1.101.0 -
  it only appears while a DEFAULT theme is active.

## [1.103.0]  (First-run onboarding + USER store search-only)

### Added
- **First-run onboarding.** A friendly three-step intro shown once on first
  launch (privacy → apps → cloud): "Absolute privacy", "Apps without servers"
  and "Your personal cloud" - written in plain human language with soft
  circular icons, step dots and Skip / Next / Get-started buttons. Dismissing
  it stores `sth_onboarding_seen`, so it never re-appears; native child
  webviews are hidden underneath while it is open.

### Changed
- **App store in USER mode is search-only.** The #tag filter strip is now
  DEV/GEEK-only - a regular user gets just the search box and clean cards.
- Note: the "TRACKERLESS MODE ACTIVE · N seeders via DHT" pill on the
  Dashboard was already USER-hidden since v1.101.0.

## [1.102.0]  (Clean USER sidebar, simple app store cards, smooth theme transitions)

### Changed
- **Cleaner sidebar in USER mode.** The "Network map" entry is now DEV/GEEK-only
  (a P2P mesh visualisation only overloads a regular user), and "STORAGES" is
  renamed to a human word - "Cloud" - in USER mode (DEV/GEEK still
  see STORAGES). On DEFAULT themes the sidebar loses its hard right border and
  the active item becomes a soft teal 10% pill fill with rounded corners,
  Google-drawer style.
- **Simpler app-store cards in USER mode.** The "● 5 seeds" counter is replaced
  by a small green online dot (grey when no seeds), the size-in-MB row and the
  tag row are hidden, and the text "DETAILS" button is replaced by a compact
  ⓘ icon button. DEV/GEEK keep the full technical card.
- **Smooth theme switching.** Changing themes (including the ☀/☾ toggle) now
  cross-fades via the View Transitions API on Chromium/WebView2, with a CSS
  colour-transition fallback (~300ms) for other webviews - like modern
  browsers, no more hard palette snap.

## [1.101.0]  (New DEFAULT theme: Google-style dark/light + USER-mode dashboard cleanup)

### Added
- **DEFAULT theme family (new default).** Two new skins in the Google/Material
  spirit: `DEFAULT DARK` (canvas `#202124`, cards `#2D2E31`, text `#E8EAED` -
  deep grey instead of "hacker black", cards separated by tone + soft shadow,
  no hard borders) and `DEFAULT LIGHT` (canvas `#F8F9FA`, pure-white cards with
  a barely-visible `0 1px 3px` shadow). Rounded corners: 12px cards, pill
  (202026-06-24px) buttons. The brand teal accent is reserved for CTA buttons and
  active elements only. `DEFAULT DARK` is now the default theme for fresh
  installs; users with a previously saved theme keep their choice.
- **Day/night toggle (☀/☾)** in the top bar, placed right before the Settings
  gear. Visible only while a DEFAULT theme is active - switches dark ↔ light
  instantly. Custom themes are still managed from Settings, where both DEFAULT
  presets now appear first in the appearance grid.
- **Chrome-like tabs.** On DEFAULT themes the tab strip mimics Google Chrome:
  borderless tabs rounded only on top, hover tint, the active tab picks up the
  surface colour with a soft elevation shadow, and the close button gets a
  circular hover.
- **New card/button tokens** (`--radius-card`, `--radius-btn`, `--shadow-card`)
  with backwards-compatible fallbacks, so all six existing themes are pixel-
  identical to before.

### Changed
- **USER mode hides the tech internals on Dashboard** (radical "normie" mode):
  the trackerless/DHT pill, the Stealth-mode block and the "+ PUBLISH APP"
  button are now DEV/GEEK-only; the hero subtitle becomes "Your personal,
  secure and free internet"; the network-features list shows
  "Absolute privacy" with a plain-language subtitle instead of
  "secp256k1 + AES-256-GCM" jargon (full tech copy remains in DEV/GEEK).
- Dashboard search field is rendered as a pill "omnibox" on DEFAULT themes.

## [1.100.1]  (Global offline freeze audit: no network must never stall the app)

### Fixed
- **Full offline-freeze audit across backend and frontend.** When the internet
  connection dropped, dozens of background network calls kept firing, hanging
  in multi-second timeouts and piling pressure on the WebView2 IPC channel -
  the root cause of the "app stops responding for a while" freezes. Every
  periodic network loop is now gated on the Rust TCP-probe (`netwatch::is_online()`
  in the backend, `useNetStore().online` in the frontend) and skips its tick
  while offline, resuming automatically when the probe sees the network again.
- **Rust background loops now offline-aware** (previously none of them checked
  connectivity): wallet balance watcher (12s), naming indexer (90s), .sth DNS
  indexer (90s), file-registry indexer (60s), private-inbox scanner (45s),
  subscriptions indexer (180s), chat key indexer (120s) and paid-message
  indexer (60s), tracker announce loop (30s), node auto-selector (10min),
  DevHub author resync (30min), oracle price sync (24h; retries every 5min
  while offline), and the Earn-STH seeding supervisor (heartbeat POST +
  per-tick tracker uptime/boost fetches with 8s timeouts are skipped offline -
  local metrics keep flowing to the UI).
- **Frontend pollers hardened**: "My Files" no longer queries on-chain sales
  counts every 5s while offline (each call fetched up to 500 wallet txs from a
  node) and pauses entirely when the window is hidden; File Exchange refresh
  pauses when hidden; the background Cloud-Seeder replicator and the Public
  Profile replicator skip OAuth refresh + WebDAV uploads offline; the client
  update checker waits for the next period instead of hanging offline; the
  Dashboard DHT meters pause while the window is hidden.
- Boot semantics preserved: until the first probe completes the network state
  counts as *online*, so first-run indexing and the SyncSplash stage marks
  (`naming`/`dns`/`subs`/`chat`/`oracle`) behave exactly as before.

## [1.100.0] - 2026-07 (Wall lazy-load, offline-safe UI, cloud-seeder update publish, discovery cache)

### Added
- **Wall lazy-load pagination.** `WallFeed.vue` now loads the microblog in
  chunks of 20 posts on demand instead of pulling the entire history at once
  (previously the profile could stall on wallets with 1000+ posts). New posts
  arriving live via events are **prepended on top without resetting the scroll
  position**, matching the VK-style feed behavior requested by the user.
- **Rust pagination surface** in `social.rs`: `list_wall(author, before?, limit)`
  returns oldest-cursor pages; the frontend keeps a `beforeTs` cursor and calls
  it as the user scrolls to the bottom of the feed. Live-append is routed
  through a separate `prepend_new` path so the cursor state is never touched
  by realtime events.
- **Cloud-Seeder update fallback publish.** Trusted seeders (from `.env`) now
  fetch the latest `vendorField` update pointer from the SmartHoldem blockchain
  REST API (`https://node1.smartholdem.io/api/transactions?recipientId=…`),
  encrypt the string with the seeder envelope and publish it to WebDAV as
  `/ntfryseed/update.enc` (`cloudseeder.rs`, `updater.rs`). Clients that cannot
  reach the blockchain API (DPI, dead nodes, mobile carrier filters) can still
  discover the latest release string from any trusted seeder - closing the
  last remaining single-source dependency in the update path.
- **Discovered-Apps last-known-good cache** (`store/discovery.ts`). The last
  successful P2P/cloud scan result is persisted to `localStorage` under
  `sth_discovery_apps` and rehydrated **synchronously** on module load, so the
  DApp search page shows the previously seen catalog immediately on cold boot
  instead of a blank screen while the background scan is still finding
  seeders. The cache is transparently overwritten by every fresh scan.

### Fixed
- **UI freeze in Offline Mode.** When `useNetStore().online` flipped to
  `false`, several background pollers (peer discovery, DApp scanner, wall
  live-refresh) kept firing every few seconds and stacked up unresolved
  requests, freezing the whole client UI. All affected intervals now gate on
  `useNetStore().online` before firing. Cloud-Seeder fallback pollers are
  kept alive when physical internet is available but P2P is dead (the two
  connectivity signals stay independent - P2P health ≠ internet health).

### Notes
- Behavior of the existing 30-min network-state + 1-h profile replicators is
  unchanged; they were already gated on wallet-unlock and were not part of the
  offline-freeze regression.
- No schema changes. `sth_discovery_apps` is a client-local `localStorage`
  key; wiping it just forces a fresh scan on next boot.

## [1.99.1]  (R1 audit: heavy Tauri commands moved off the main thread)

### Fixed
- **Main-thread freeze on network transition (offline → mobile) eliminated.**
- **Fix:** the heavy/blocking commands are now declared `#[tauri::command(async)]`
  so Tauri executes them off the main thread (bodies unchanged, no API/behaviour
  change): `verify_app`, `list_apps`, `announced_apps`, `app_logo`, `read_readme`,
  `blob_store_usage`, `dedup_stats`, `dht_providers_summary`, `discovered_peers`
  (polled every 4 s), `swarm_top_seeders`, `dht_nodes_list`, `list_usernames`,
  `list_market_orders`, `list_my_domains`, `domains_by_owner`, `weekly_top`,
  `get_rooms`, `get_messages`, `read_update_manifest`, `resolve_installer`,
  `run_p2p_installer`, plus the native-dialog / filesystem commands `pick_file`,
  `pick_folder`, `pick_image_file`, `pick_audio_files`, `save_file_dialog`,
  `save_file_as`, `open_file_native`, `wipe_all` (the `blocking_*` dialog API is
  designed to be called off the main thread - this also removes a latent
  GTK modal-loop deadlock risk).
- Network calls in sync commands: **none found**. All wallet/chain REST
  (`node_get`/`node_post`/`fetch_*`) is already wrapped in `spawn_blocking` or runs
  on dedicated threads with timeouts; no blocking `ureq` inside any async context.
- `cargo check --features p2p`: clean (warnings only).

## [1.99.0]  (Profile: 60/40 wall, responsive horizontal profile bar)

### Changed
- **Wall proportions:** center microblog vs right sidebar is now **60/40**
  (`flex: 3` / `flex: 2`), and the right sidebar is capped at **380px** so it
  can't get wide - the feed is always the widest column.
- **Profile column split into 4 sections** (`.id-sect`): (1) avatar → address →
  name (+ actions), (2) rewards - reputation, subscriber/following counters,
  medals, (3) wallet balance + u:// link + QR, (4) visibility toggle.
- **Responsive (`≤1280px`):** the left profile column reflows into a **horizontal
  bar on top** (4 sections split by vertical dividers); below it the right column
  shows the two columns - microblog 60% + subscribers/playlists 40%. The wall
  only stacks to a single column below **820px** (was 1100px).



### Changed
- **`UserProfile.vue` Wall tab layout.** Subscribers and Friends/Following are
  now a **single sidebar panel** (two labelled groups with a divider), each
  showing **max 4 avatars per row** (`.slice(0, 4)`; "Show all" opens the full
  graph modal). Removed the `≥1600px` rule that expanded the sidebar to a 560px
  2-column block and squeezed the feed.
- Right sidebar is now a fixed **narrow 340px** column on all widths; the center
  **microblog feed is always the widest** column (`flex:1`, max-width 1080px).
- Sidebar avatars use minimal chrome (no card border) to match the reference.



### Changed
- **`UserProfile.vue`: the "My Domains" block moved from the left column into the
  DAPPS tab** as a separate full-width block, ordered between "Published dApps"
  (order 1) and "Contribution Stats" (order 3) via `.right-col` flex `order`
  (`.domains-dapps` = order 2). Shown only on the DAPPS tab; markup/logic
  unchanged (renew, show-to-all, open bound dApp).



### Added
- **`TRANSLATOR_MODEL_CLOUD` - public Mail.ru Cloud link as a model source**,
  now the **primary** attempt in `run_bootstrap`. Priority is
  **cloud (Mail.ru) → HTTPS → iroh**. The cloud source is tried first with a
  ~6s availability timeout (resolve + connect); if Mail.ru doesn't respond in
  time, bootstrap falls through to HTTPS, then iroh.
- New anonymous resolver `ntfry::seeder::mailru_public_direct_url()` turns a
  `https://cloud.mail.ru/public/<weblink>` file link into a direct download URL
  (dispatcher `weblink_get` base + download key) so the GGUF **streams to disk**
  with progress (no full in-memory buffering) via `download_file`.
- `download_file` gained an optional connect-timeout (used only by the cloud
  path; HTTPS/tokenizer keep the previous unbounded behavior). Downloaded model
  is validated to start with the `GGUF` magic (guards against HTML error pages).

### Changed
- `run_bootstrap` now fills whatever is still missing per source and re-checks
  `is_installed` between steps (cloud fetches the model only; candle tokenizer,
  if needed, still comes from HTTPS/iroh).



### Added
- **DONATE button on the Dashboard topbar** (left of the MeshMeter pill, pulsing
  green pill) opening a terminal-styled **support modal** (`DonateModal.vue`):
  "// SUPPORT THE NETWORK_" heading, the manifesto text, and 7 crypto addresses
  (BTC, ETH, BTS, STH, GRAM/TON, DOGE, LTC) - each with a one-click Copy button
  (`navigator.clipboard`, per-row "✓ COPIED" feedback). Pure frontend, no
  backend. Verified: `yarn build` clean + isolated visual render of the modal
  (the live Dashboard sits behind the Tauri vault gate, unavailable in web preview).



### Added
- **Settings → Storage → "AI translation cache" now shows a translation-engine
  badge**: `GPU CUDA` / `GPU METAL` / `CPU + BONSAI` / `CPU`. New command
  `translator_engine_label()` derives it from the build accelerator +
  `cfg!(feature = "bonsai")`; frontend `translatorEngineLabel()` bridge wrapper,
  colored pill in `Settings.vue` (i18n `storage.trCacheEngine*`).



### Added
- **`bonsai`-built clients now translate DApp pages locally on CPU** (not just
  "cache-only" over P2P). New command `translator_dapp_capable()` = GPU build
  **OR** `cfg!(feature = "bonsai")`. `dapp_translate_batch` now gates on this
  capability instead of `build_accel == "cpu"`, so a CPU+bonsai client runs the
  1-bit Q1_0 model for page auto-translate (the address-bar 🌐 button). Plain
  CPU builds without bonsai stay in cache-only mode as before.
- Frontend: `translatorDappCapable()` bridge wrapper; `AppLayout.vue` derives the
  "cache-only" badge/tooltip from capability rather than raw accelerator string.



### Fixed
- **Bonsai (llama.cpp) translations could run away and freeze the client.** The
  llama.cpp generation loop only stopped on `is_eog_token`; if a Bonsai GGUF does
  not mark `<|im_end|>` / `<|endoftext|>` as EOG, generation never stopped early
  and ground out the full `MAX_NEW_TOKENS` on every translation - appearing as a
  hang. The loop now also breaks on the explicit `QWEN_EOS` token ids
  (`151645`, `151643`), exactly like the candle path. Inference still runs inside
  `spawn_blocking` (main thread / P2P never blocked); no `.await` under any
  `std::sync::Mutex` (R1 compliant).



### Added
- **`bonsai-4b` preset now carries a real iroh ("torrent") source.** Filled
  `model_iroh_cid` = `b75cf8c8…f3b3e3fe`, `model_iroh_node` = `DEFAULT_IROH_NODE`,
  and the exact `model_size` = `572_270_624` bytes, taken from the File-Exchange
  `sn://file/…Bonsai-4B-Q1_0.gguf` seed. HTTPS stays primary; iroh is the P2P
  fallback (and joins the swarm) once HTTPS is unavailable.

### Docs
- Reminder captured in `docs/13-Model-Seeding-Guide.md`: `TRANSLATOR_MODEL`
  takes the **preset name** (`bonsai-4b`), not the HF repo name.



### Added
- **Full Bonsai (Q1_0 1-bit) support for the local Wall/DApp translator via an
  FFI backend to `llama.cpp`** (crate `llama-cpp-2` 0.1.153). candle 0.9 cannot
  read the 1-bit `Q1_0` GGUF block type; per PrismML docs `Q1_0` is now upstream
  in llama.cpp (the PrismML fork is only needed for ternary `Q2_0`), so a recent
  bundled llama.cpp runs Bonsai-8B / Bonsai-4B directly.
- **New engine dispatch** in `translator_config.rs`: `EngineKind { Candle,
  LlamaCpp }` on every preset. Qwen presets stay on candle unchanged; `bonsai-4b`
  / `bonsai-8b` route to the llama.cpp backend.
- **New feature-gated module** `translator::llama_engine` (`#[cfg(feature =
  "bonsai")]`): initializes the llama backend once, keeps the loaded `LlamaModel`
  in a static, and creates a fresh context per translation (clean KV cache).
  Greedy sampling, ChatML prompt reused from `build_prompt`, EOS via
  `is_eog_token`; all inference runs inside `spawn_blocking` and is serialized.
- **Cargo features**: `bonsai` (CPU), `bonsai-cuda`, `bonsai-metal` - off by
  default (heavy C++/cmake build), mirroring the existing `cuda`/`metal` gating.
  Build with e.g. `yarn tauri build -- --features p2p,bonsai`.

### Changed
- Bonsai presets: `supported = cfg!(feature = "bonsai")` - the UI now enables
  Bonsai only in a `bonsai`-built client and shows a clear "rebuild with
  `--features bonsai`" message otherwise (instead of the old "needs PrismML
  fork" text, which was inaccurate - Q1_0 is upstream).
- Bonsai presets no longer reference a separate `tokenizer.json`: llama.cpp
  reads the vocabulary straight from the GGUF, so `is_installed`, `https_fetch`
  and `iroh_fetch` skip the tokenizer download for the llama.cpp engine.
- `.env.example`: Bonsai preset notes updated (feature-gated, not unsupported).

### Tests
- `translator_config` unit tests: Bonsai presets → `LlamaCpp` + empty tokenizer
  + `Q1_0.gguf` + `supported == cfg!(feature="bonsai")`; Qwen presets stay on
    candle and remain supported.

### Build / verification
- Default build verified in-container: `cargo check --features p2p` → **0
  errors** (only pre-existing warnings). The `bonsai` feature compiles llama.cpp
  from C++ (cmake) and, like `cuda`/`metal`, is built and E2E-verified on the
  developer machine (the CI container lacks disk headroom for the C++ toolchain
  build + the ~1.15 GB model download).



### Fixed
- **Crypto Vault delete no longer lies about a successful deletion.** When the
  backend returns `VaultError::GcFailed` (the file *was* removed from the vault
  index, but some orphan chunks could not be garbage-collected from the cloud),
  the UI used to treat it as a total failure - it rolled the row back into the
  list **and** raised a red error, even though the file was actually gone.
  Now `CryptoVault.vue` (`removeRow` / `removeRecent`) detects this partial-
  success case, keeps the optimistic removal, refreshes status, and shows a
  clear warning toast ("deleted, but not all chunks removed from the cloud - GC
  will finish later") instead of a misleading error. True failures still roll
  back and surface the error as before.

### Verified (code review)
- Breadcrumb duplication guard confirmed intact: `crumbs` derives solely from
  `cwd`, and both `openRow`/`goCrumb` short-circuit on the `navBusy` lock, so a
  rapid second click is dropped until the first `refresh()` resolves.

## [1.94.0] - 2026-07 (Translator model config via .env: minimal knobs, presets carry everything)

### Added
- **New Rust module** `frontend/src-tauri/src/translator_config.rs` - the
  translator model definition (filename, HTTPS URL, iroh CID/node, size,
  architecture, chat template, EOS tokens, required disk space,
  supported flag) is no longer hardcoded in `translator.rs` but lives in
  a preset catalog. Adding a new model = one entry in `preset()`.
- **Two environment variables - that's it:**
  - `TRANSLATOR_MODEL=<preset>` - pick a preset (default `qwen25-1.5b`).
  - `TRANSLATOR_MODEL_URL=<https>` - override the HTTPS mirror of the
    active preset (for operators with a private HuggingFace mirror).
    Everything else (iroh CID, size, architecture, tokenizer, EOS tokens,
    chat template) is baked into the preset - no byte-counting required.
- **Model presets** (built-in, `TRANSLATOR_MODEL=<name>`):
  - `qwen25-1.5b` - default, Qwen2.5-1.5B-Instruct GGUF Q8_0 (~1.9 GB),
    same iroh seeder + HuggingFace HTTPS as v1.93.
  - `qwen3-1.7b` - Qwen3-1.7B GGUF Q8_0 (~1.9 GB), HTTPS-only.
  - `qwen3-4b` - Qwen3-4B GGUF Q4_K_M (~2.5 GB), balanced.
  - `qwen3-8b` - Qwen3-8B GGUF Q4_K_M (~4.7 GB), max quality.
  - `bonsai-4b`, `bonsai-8b` - Bonsai 1-bit (Q1_0) - **listed as
    `supported=false`**: Q1_0 is a PrismML-proprietary GGUF quantization
    requiring their [llama.cpp fork](https://github.com/PrismML-Eng/llama.cpp);
    candle 0.9 can't parse it. The preset is present so a user gets a
    clear human-readable message ("use qwen3-4b/8b - same Qwen3
    architecture, works out of the box") instead of a cryptic GGUF
    parser crash. When an FFI-based llama.cpp backend lands, flipping a
    single `supported` flag activates Bonsai.
- **Second architecture backend** - `ModelBackend` enum dispatches
  between `candle_transformers::models::quantized_qwen2::ModelWeights`
  and `quantized_qwen3::ModelWeights` (same `from_gguf` + `forward` API).
- **New IPC command** `translator_presets` - returns the full preset
  catalog (`{name, description, supported, unsupported_reason}`) so a
  Settings UI can render a picker.
- **`TranslatorStatus` extended** with `model_name`, `model_file`,
  `model_description`, `model_supported`, `model_unsupported_reason`
  so the UI can display the active model and warn about unsupported
  selections.
- **`.env.example`** - added a small `TRANSLATOR_*` block documenting
  the two knobs and the preset list.

### Changed
- **Bootstrap priority reversed** to **HTTPS URL first → iroh CID
  fallback** (previous: iroh first, HTTPS as a separate manual command).
  Rationale: HTTPS is universally reachable; iroh (SmartNet's P2P /
  "torrent" transport) is now the anti-censorship backup - matching the
  CloudSeeders philosophy. Fallback triggers on any HTTPS error with a
  syslog entry. `translator_download` is a thin wrapper around
  `translator_bootstrap`.
- **`fetch_blob_with_progress`** now takes `iroh_node` as a parameter
  (was a hardcoded constant).
- **`build_prompt`** switches on `ChatTemplate` from config; current
  variant `ChatMlQwen` covers Qwen2/2.5 and Qwen3 (same
  `<|im_start|>/<|im_end|>` markers).
- **EOS-token check** in `run_inference` reads the `eos_tokens` slice
  from config instead of the two hardcoded Qwen2 constants.
- **`ensure_engine`** refuses to load models with `supported=false`
  with a clear error message before attempting to parse GGUF metadata.

### Migration notes
- No user action required - the default preset `qwen25-1.5b` reproduces
  the pre-1.94 constants byte-for-byte (same file name, same HTTPS URL,
  same iroh CID/node/size, same EOS tokens, same chat template).
- To switch model, set `TRANSLATOR_MODEL=qwen3-4b` in `frontend/.env`
  (or system env) and restart the client. First run downloads the new
  GGUF via HTTPS (~2.5 GB for Qwen3-4B).
- **Bonsai is a placeholder for a future FFI feature**, not a working
  option in this release.

## [1.93.0] - 2026-07 (Public Profile Seeders: profiles, microblogs and attachments over cloud)

### Added
- **Public Profile Seeders** - a new opt-in sub-layer on top of the existing
  Cloud Seeder (`/ntfryseed`) that mirrors the author's profile and microblog
  (plus the top-10 followed profiles) into Yandex.Disk / Mail.ru Cloud.
  Enables **cloud-only reading** of SmartNet social content when P2P is
  blocked (closed UDP ports, DPI, dead trackers). Reuses the same
  HKDF-SHA256(pubkey) → ChaCha20-Poly1305 + secp256k1 signature envelope as
  the network-state seeder, so no new crypto surface is introduced.
  Publisher scope:
  - **My profile** (`ProfilePublic` only) + wall_posts limit **25** + comments
  - **Top-10 follows** (first 10 addresses from `following_addresses`) + wall_posts
    limit **10** each + comments
  - **Attached files** (`PostFile.cid` - BLAKE3 content-address) - only those
    referenced by the replicated posts
    Cloud layout:
  ```
  /ntfryseed/users/
    manifest.enc                       ← index {userId → {hash, files, updated}}
    <userId>.enc                       ← UserBundle: profile + posts + comments
    <userId>/files/<blake3-cid>.enc    ← attached files (immutable, dedup)
  ```
- **`ntfry::seeder` - generic subpath API**: `yandex_upload_subpath`,
  `mailru_upload_subpath`, `yandex_delete_subpath`, `mailru_delete_subpath`.
  MKCOL-chains create missing parent dirs; upload path reuses the exact same
  hardened `resilient_put` pipeline (90 s idle watchdog, no `read_timeout` on
  the streaming body, 3 attempts with 2 s / 4 s backoff, permanent-error gate,
  sha256 pre-check on Yandex, size fallback on Mail.ru).
- **`profile_seeder.rs`** - new Rust module (`frontend/src-tauri/src/`) that
  builds `UserBundle` snapshots (v=1), signs them, and drives delta
  replication via a sled snapshot map (`cfg:profile_seeder_state`).
- **Delta replication**: bundle is uploaded only if its content-hash
  (sha256 of the bundle JSON minus the volatile `updated` field) changed;
  files are uploaded only for new CIDs (BLAKE3 immutability + Yandex sha256
  pre-check give free cross-user dedup); files that dropped out of the bundle
  (post deleted or edited) trigger a targeted `DELETE
  /ntfryseed/users/<uid>/files/<cid>.enc` on the seed disk so orphans do
  not accumulate.
- **Cheap local hash-check** (`profile_seeder_has_changes`) - before making
  any network call the client rebuilds all target bundles and diffs their
  hashes against the sled snapshot; if nothing changed, the replication
  cycle exits without touching the cloud (zero network requests, honours
  the user's request "before replication check if there are any changes").
- **8 new Tauri IPC commands** wired through `lib.rs` and `bridge.ts`:
  `cloud_seeder_publish_profiles`, `cloud_seeder_fetch_profiles_manifest`,
  `cloud_seeder_fetch_profile`, `cloud_seeder_fetch_profile_file`,
  `profile_seeder_autorep_get`, `profile_seeder_autorep_set`,
  `profile_seeder_last_replicated`, `profile_seeder_has_changes`.
- **Background scheduler** (`seederRep.ts::startProfileReplication`): ticks
  every 5 min but only touches the cloud when auto-rep is on AND
  `now - lastReplicated >= 1h` AND `profileSeederHasChanges()` returned
  true. Wired from `AppLayout.vue` right after wallet unlock, next to the
  existing 30-min network-state replication.
- **Progress events** (`profile-publish-progress`): fields `phase`
  (`collect | user | file | manifest | done`), `index`, `total`, `userId`,
  `fileCid`, `sizeBytes`, `uploadedBytes`, `kbps`, `attempt`, `status`,
  `error`. Wired into UI live.
- **Seeders.vue** - new UX block **"Replicate Profiles"** under
  "Auto-replication every 30 min":
  - Toggle `data-testid="profile-replicate-toggle"` (default OFF, opt-in).
  - `PUBLISH NOW` button for manual runs (`data-testid="profile-replicate-publish-now"`).
  - Live progress line with retry badge (attempt N/3), KB/s, phase.
  - Result stats: users published / skipped, files uploaded / dedup-skipped,
    orphans deleted.
- **Cloud-profile reader modal**:
  - `👤 FETCH PROFILES` button in the Reader section fetches
    `users/manifest.enc` from any seeder URI and lists all published users
    with post counts and file counts.
  - `VIEW` opens a modal with the full `UserBundle`: profile header, wall
    posts, comments per post, attachment metadata.
- **Inline attachment preview & download inside the modal**:
  - `▶ LOAD` button (only for `image/*`, `audio/*`, `video/*`) fetches the
    file via `cloud_seeder_fetch_profile_file` (base64 of the decrypted
    plaintext), converts it to a `Blob`, creates a session `objectURL`,
    and renders it inline as `<img>` / `<audio controls>` / `<video controls>`.
  - `⇩ DOWNLOAD` button - universal for any MIME. Reuses the cached
    objectURL if already loaded, otherwise fetches on demand, then
    triggers `<a download>`.
  - Deduped per-session (repeat clicks on the same CID never hit the
    network twice).
  - `URL.revokeObjectURL(...)` on modal close, on profile switch, on
    `Fetch profiles` re-run, and on component unmount - no leaked Blob
    URLs even after a long browsing session.
  - Per-file inline error surface (a broken/tampered `<cid>.enc` shows an
    error on its own row without blocking the sibling files).
  - Test selectors: `cloud-file-<cid>`, `cloud-file-load-<cid>`,
    `cloud-file-download-<cid>`, `cloud-file-preview-<cid>`.

### Documentation
- **New**: `docs/12-Cloud-Seeders-Guide.md` - 20 Russian sections covering the
  full anti-censorship layer: URI format, `/ntfryseed` layout, deterministic
  public-encryption crypto, mobile-network resilience, chain-replication,
  cloud fallback, VK-ID workaround, and the full Profile Seeders subsection
  with inline media preview.
- **Updated**: `docs/11-Crypto-Vault-Guide.md` - version bumped to `1.93.0+`,
  Mail.ru provider section added, cross-link to guide 12.
- **Updated**: `docs/README.md` - table of contents now covers documents
  09–12 (the ToC previously stopped at 08).

### Security / Privacy
- **Only `ProfilePublic` is ever published** - the wallet's
  `ProfilePrivate` (`encrypted_secret`, private handles) is not sent to
  the module at all. Enforced at bundle-build time in `profile_seeder.rs`.
- **Deleted posts and hidden comments are filtered out** at bundle build
  (`social::is_deleted`, `social::is_comment_hidden` from
  `social:del:*` / `social:hidden:*` sled keys) so a deleted post never
  reaches the cloud even in a follower's bundle.
- **Signature enforcement unchanged**: every `<userId>.enc`, every
  `<cid>.enc`, and the `manifest.enc` are sealed with the author-seeder's
  secp256k1 key and verified by the reader before decryption; content
  tampering by the cloud provider is rejected via AEAD tag or ECDSA verify.
- **Publishing requires an unlocked wallet** (`signing_key()` reads
  `SESSION_SEED`); a stolen OAuth token alone cannot publish signed
  bundles.

### Compatibility
- No breaking changes. The existing `sn://seed/...` URI format is reused
  as-is; readers on older clients simply won't see the `users/` subtree.
- `resilient_put` internals unchanged - all 11 existing `ntfry::seeder`
  unit tests still pass (crypto roundtrips, tamper detection, signature
  forgery rejection, deterministic seal, precheck skip, resume-without-
  reupload, streaming-body progress).

## [1.92.0] - 2026-07 (dApp tab shows its install source: ☁ Cloud Seeder vs ⇄ direct P2P)

### Added
- **Install-source badge in the dApp tab** (`AppLayout.vue`, `tabs.ts`): every
  app tab now shows where the installed copy came from - "⇄" for a direct
  iroh-blobs P2P install, "☁" for a Cloud Seeder archive (anti-censorship
  fallback). The tooltip carries the storage provider (yandex/mailru), the
  installed version and the install timestamp, making the fallback layer
  visible to the user at a glance.
- **Persistent install-source record** (`p2p.rs::install_app`,
  `cloudseeder.rs::install_from_seeder`): each successful install/update
  writes `install_src:<id>` to Sled (`kind`, `provider`/`node`, `uri`, `at`,
  `version`). New `app_install_source` IPC reads it back; the record is
  purged with the app in `delete_app`. Updates that switch transport
  (P2P ↔ seeder) refresh the badge live via `discovery.applyUpdate`.

## [1.91.1] - 2026-07 (FIX: 404 page after installing a dApp from Cloud Seeders)

### Fixed
- **Stale native webview showed a cached 404 after install** (`webview.rs`,
  `AppLayout.vue`): `open_embedded_webview` reused an existing `embed-<id>`
  child webview *as-is* (only show/position). If a webview for that app id
  survived from an earlier failed open (e.g. the app was not yet installed),
  the freshly installed dApp tab kept displaying the old 404 page forever.
  The frontend now passes `fresh=true` whenever it believes the webview does
  NOT exist yet; on a label collision Rust force-navigates the survivor back
  to the dApp root instead of silently showing stale content.
- **App id now travels in the URL query too** (`webview.rs`, `protocol.rs`):
  every dApp URL gets a `?sthapp=<id>` marker. If the platform WebView drops
  or mangles the custom-scheme authority (host), `resolve_request` recovers
  the app id from the request query or the Referer query as a last resort -
  the initial page load can no longer 404 just because the host was stripped.
- **Windows Referer host form `sth.<id>` was not recognized** (`protocol.rs`):
  wry 0.55 maps `sth://<id>/` to `http://sth.<id>/` (scheme becomes a host
  PREFIX). `host_app_id` only stripped the `<id>.sth.localhost` suffix form,
  so the Referer-based fallback resolved to the bogus id `sth.<id>`. Now the
  `sth.` prefix is stripped as well.

### Added
- **SYSCON diagnostics for every sth:// 404** (`protocol.rs`): each unresolved
  request now logs the raw host/path/Referer, the resolved app id and the
  actual contents of `data/dapps` under scope `dapp`, so a 404 on a user
  machine is immediately diagnosable from the System Console.

## [1.91.0] - 2026-07 (Broken-dApp protection + cloud seed disk cleanup on local delete)

### Added
- **Broken-archive guard on seeder install** (`cloudseeder.rs`): after
  unpacking a Cloud Seeder archive, the installer now requires `index.html`
  at the dApp root. A broken archive (the cause of the reported "404 page
  right after installing from seeds") is rejected with a clear error, the
  half-unpacked folder is removed, and `cloud_seeder_install_any` moves on
  to the next seeder instead of registering an unopenable app. Unpack /
  manifest-parse / id-mismatch failures now also clean up the target folder
  so a corrupt `data/dapps/<id>` never lingers to be served as a 404.
- **Publish guard for developers** (`apps.rs::ingest_app`): creating,
  previewing, publishing or updating a dApp now fails fast with
  "no index.html in dApp root…" when the build folder has no root
  `index.html` - broken apps can no longer be published and distributed.
- **Cloud archive gate on seeder publish** (`cloudseeder.rs::build_network_state`):
  a dApp folder without `index.html` is skipped as an archive candidate
  (logged to SYSCON) and advertised with `"archive": false` in `dapps.enc`,
  so peers never download a known-broken archive.
- **Cloud seed disk cleanup on local delete** (`cloud_seeder_delete_dapp`
  IPC, `ntfry::seeder::{yandex,mailru}_delete_seed_dapp_file`): when the
  user deletes a dApp locally, its encrypted archive is also removed from
  the user's own Cloud Seeder drive (current `dapps/<id>.enc` plus legacy
  layouts), the per-app upload marker is reset, and the network state is
  immediately re-published so `dapps.enc` no longer lists the removed app.
  Wired through `apps.ts::remove()` → `seederRep.deleteDappFromCloud()`
  (best-effort, silent when the user has no published seeder).

### Verified
- Full pipeline reproduced in an isolated Rust harness: tar → `seal_bytes`
  → `open_bytes` → unpack → case-insensitive `sth://` serve - all OK; a
  broken archive (no `index.html`) is rejected and cleaned up.

## [1.90.10] - 2026-07 (FIX: self-discovery poisoned the peers table - cloud scan never fired)

### Fixed
- **Client discovered ITSELF as a live P2P peer** (`p2p.rs`): the mDNS browser
  receives the node's own service announcement and inserted it into `PEERS`
  with `via_seeder=false`. Consequences in fully-offline (cloud-only) mode:
  `has_direct_peers()` was always `true`, so BOTH the backend cloud fallback
  and the v1.90.9 frontend cloud-drive scan never ran; DISCOVER APPS showed
  only the user's own installed dApps ("1 seed" each, no ☁ badge) and never
  the latest versions available on Cloud Seeder drives. Now `start_discovery`
  records the own identity (wallet address + iroh NodeId) in `OWN_IDENTITY`
  and `is_self_peer()` filters the own announce in the mDNS resolve handler,
  in `merge_tracker_apps` (trackers echo our announce back) and in
  `merge_seeder_nodes` (foreign nodes.enc may contain a snapshot of us).
- **Dead LAN peers were never evicted** (`p2p.rs`): the browse loop only
  handled `ServiceResolved`, so a peer that went offline stayed in `PEERS`
  forever, keeping `has_direct_peers()=true` and blocking the switch to the
  cloud layer mid-session. `ServiceRemoved` is now handled via a new
  `MDNS_FULLNAMES` (fullname → peer key) map; `stop_discovery` clears it.

### Impact
In cloud-only operation the client now correctly detects "no regular
seeders", immediately scans the cloud drives of all known Cloud Seeders and
surfaces the LATEST published dApp versions in DISCOVER APPS (☁ via seeder),
with auto-update picking newer cloud archives.

## [1.90.9] - 2026-07 (Discover Apps: on-demand Cloud Seeder drive scan when P2P seeders are down)

### Added
- **`cloud_seeder_scan_apps` Tauri command** (`cloudseeder.rs`): scans the
  cloud drives of ALL known Cloud Seeders on demand (fetches their signed
  `dapps.enc`), merges discovered dApps into the peers table as synthetic
  `via_seeder` peers and returns counters (`seedersOk/seedersTotal/nodes/dapps`).
- **Discovery store cloud fallback** (`store/discovery.ts`): when the
  Discover page opens (or the SCAN button is pressed) and there are **no
  live P2P peers**, the store immediately triggers a cloud-drive scan; the
  4-second background poll re-scans at most once per 2 minutes (throttled to
  avoid hammering Yandex/Mail.ru APIs). Cloud scans run ONLY while regular
  seeders are unreachable - live P2P never triggers cloud traffic.
- **Discover UI indicators** (`views/Discovered.vue`): a matte-blue
  "☁ Scanning cloud drives of known Cloud Seeders…" progress strip, a note
  when the whole catalog is assembled from cloud drives, and an empty-state
  hint pointing to the Seeders page when nothing was found (en/ru/id i18n).

### Fixed
- **Background cloud fallback no longer dies permanently** (`cloudseeder.rs`):
  the `spawn_cloud_fallback` loop used to `break` forever the first time a
  live P2P peer appeared - if P2P dropped later in the session, the client
  never returned to the cloud layer. The loop now idles while P2P is alive
  (30 s re-check) and automatically resumes 5-minute cloud polling when P2P
  goes dark, logging each state transition once.

### Impact
Clients whose regular seeders are unreachable now see the full dApp catalog
in DISCOVER APPS (badged "☁ via seeder") and can install directly from
encrypted cloud archives - previously installs were only possible from the
PUBLIC NETWORK SEEDERS page in that scenario.

## [1.90.8] - 2026-07 ("Become a seeder" hint after 1 hour in Cloud mode)

### Added
- **Ambient "Publish your Cloud Seeder" hint** (`components/ConnectionBar.vue`):
  a subtle glass-panel popover that surfaces above the Cloud-mode plate when
  the client has been running through the cloud fallback continuously for
  more than **1 hour** and has **no own seeder published yet**
  (`ownSeeder()` from `seederRep.ts` returns `null`). The hint reads
  "Publish your Cloud Seeder - become part of the fallback infrastructure.
  The network relies on peers like you" with a CTA button that jumps to
  `/seeders` and a close (×) button.
- **7-day snooze**: dismissing the hint stores `sth_cloud_hint_dismissed_at`
  in `localStorage`; the hint stays hidden for a week (both explicit ×
  and CTA click count as dismissal - CTA also navigates to Seeders).
- **Continuous Cloud-mode session tracking**: `cloudModeSince` timestamp is
  set the moment the bar switches to `cloud=true` and cleared on any exit
  (P2P back OR seeders go dark). A 1-minute ticker drives the reactive
  threshold check without measurable render cost.

### Impact
Turns passive cloud-fallback consumers into active seed providers exactly
when they've proven they benefit from the layer - the ask lands after a
full hour of visible reliance, not on first launch. Once dismissed, the
prompt stays quiet for a week; publishing a seeder makes it never appear
again (`ownSeeder()` gate).

## [1.90.7] - 2026-07 (Cloud-mode status bar + Discovery updates via Cloud Seeders + MY_APPLICATIONS sorting)

### Added
- **Clickable "Cloud mode" status bar** (`components/ConnectionBar.vue`):
  when SmartNet P2P is offline **but** at least one known Cloud Seeder answered
  successfully (either `ok:true` or `lastOk < 15 min`), the bottom bar switches
  from red "No connection" to a matte-blue "Cloud mode →" plate
  (`linear-gradient(180deg, #142236, #0b1524)`, border `#2a4a75`, dot `#60a5fa`,
  text `#93c5fd`). Left side keeps `SmartNet // OFFLINE`. The bar is now a
  keyboard-focusable button - click / Enter / Space navigates to
  `/seeders#active-seeders` so the user can immediately see which seeders are
  holding the network. Seeder health is polled every 20 s while the bar is
  visible.
- **KNOWN SEEDERS focus mode** (`views/Seeders.vue`): opening the page with
  `#active-seeders` scrolls the registry into view, applies a soft blue
  card glow (`box-shadow` + border), and pulses "live" seeders (three-cycle
  animation). Live seeders (`ok=true` or `lastOk < 15 min`) are hoisted to the
  top of the list and get a `☁ HOLDING` pill; the section header shows a
  compact `☁ N holding the network` counter.
- **App size on `MY_APPLICATIONS` cards** (`views/MyApplications.vue`):
  both draft and published cards now display `Size: X MB` (sum of
  `files[].size`, formatted via existing `fmtSize` helper) alongside the file
  count. Test IDs: `app-size-<id>`, `draft-size-<id>`.

### Changed
- **MY_APPLICATIONS ordering**: `filteredPublished` and `filteredDrafts`
  are now sorted by `updated` (fallback `created`, treated as unix-seconds)
  in descending order - the most recently updated dApp always floats to the
  top of the list. Sorting is applied after the search filter so results
  stay chronological.
- **Discovery: updates via Cloud Seeders** (`store/discovery.ts`):
  `aggregate()` no longer requires an iroh blob hash (`a.hash`) to flag
  `updateAvailable` - updates advertised by seeder-only peers (where the
  iroh blob may be absent) are now surfaced correctly. `updateProvider`
  is populated only for live P2P peers (`a.hash && peer.nodeId &&
  !peer.viaSeeder`); for seeder-only updates it stays `null`.
  `applyUpdate()` is now a two-step: (1) try the live P2P provider when
  present; (2) fall back to `cloudSeederInstallAny(id)` - the newer version
  is pulled from the encrypted cloud archive. Integrity gate
  (`verifyApp` + signed merkle) runs after either path. `handleUpdates()`
  no longer prefetches for seeder-only updates (nothing to prefetch over
  iroh - the archive is fetched at apply time).

### Impact
- Clients with **zero direct P2P connectivity** (behind NATs, corporate
  firewalls, or fully offline WANs with only cloud-storage reachable) now
  receive new dApp versions automatically through the Yandex/Mail.ru
  encrypted archives - the anti-censorship fallback layer completes its
  original design goal.
- The status bar communicates network topology at a glance: users no longer
  see a scary red "no connection" when the cloud fallback is doing its job.

## [1.90.6] - 2026-07 (Seeder publish: pre-upload sha256 check - unchanged archives skipped with zero traffic)

### Added
- **Pre-upload cloud check in `resilient_put`** (`ntfry/src/seeder.rs`):
  before the first PUT the client probes the target file's metadata; if an
  archive with the exact size **and sha256** already sits on the disk, the
  upload is skipped entirely (0 bytes of traffic) - covers e.g. a failed
  merkle-gate record after an interrupted previous publish. `verify(strict)`
  now has two modes: strict (pre-check, hash match required) and lenient
  (post-silence, size fallback while Yandex is still computing the hash).
  Mail.ru never passes the strict pre-check (WebDAV exposes no hash;
  skipping on size alone could silently keep a stale archive in the cloud).
- Upload functions now return `bool` (`true` = bytes were transferred,
  `false` = identical archive already in the cloud); the publish log states
  "identical already in cloud (sha256 matched) - upload skipped".

### Changed
- **`seal_bytes` nonce is now deterministic (SIV-style)**:
  `sha256("ntfryseed-nonce-v1" ‖ key ‖ zstd(payload))[..12]` instead of a
  random nonce. Identical content now produces a byte-identical envelope,
  which is what makes the sha256 pre-check possible at all. Safety: a nonce
  can only repeat together with the very same plaintext (AEAD-safe); the
  only thing an observer learns is "content unchanged", and the seeder layer
  is deterministically decryptable by pubkey anyway. ECDSA signing is
  RFC 6979 deterministic and zstd is deterministic, so the whole pipeline is
  reproducible. Envelope format (v3) and readers are unchanged - old
  random-nonce blobs in the cloud remain fully readable.

### Tests
- `seal_bytes_is_deterministic_for_same_content` - same content → same blob,
  different content → different blob;
- `resilient_put_precheck_skips_upload_entirely` - strict pre-check hit →
  zero network PUTs, returns `uploaded=false` (target URL is unreachable on
  purpose, any PUT attempt would fail the test);
- 30 tests total, all green.

## [1.90.5] - 2026-07 (Seeder publish: verify-after-silence - no more 40 MB re-uploads when only the PUT response is lost)

### Added
- **Post-attempt cloud verification in `resilient_put`** (`ntfry/src/seeder.rs`).
  User-confirmed scenario: disk history showed every upload actually
  **landed successfully** on Yandex, yet the client kept looping - the file
  was stored, only the HTTP response to the PUT was lost on the mobile
  network, so each retry re-uploaded the full 40 MB archive from scratch.
  Now after every failed non-permanent attempt the client performs a cheap
  metadata probe of the target file on the disk:
    - **Yandex**: `GET /v1/disk/resources?fields=size,sha256` - success is
      declared when size **and** sha256 of the sealed archive match (sha256 of
      a just-written file may still be computing server-side; in that case a
      size match is accepted - an older file of the same name would already
      have its hash);
    - **Mail.ru**: WebDAV `PROPFIND Depth:0` `getcontentlength` - size-only
      (WebDAV exposes no hash; PUT is atomic, so an exact-size file means a
      complete upload landed);
    - any probe error → `false` → the normal retry flow continues.
- `storage::mailru::xml_num` promoted to `pub(crate)` for reuse.
- New test: `resilient_put_accepts_verified_file_without_reupload` - local
  HTTP server swallows the body and answers 503; with a positive verify the
  call succeeds after exactly **one** PUT (re-upload counter asserted).
  28 tests total, all green.

## [1.90.4] - 2026-07 (Seeder publish: stall-guard instead of read_timeout - slow 3G uploads no longer aborted at 300s)

### Fixed
- **"Yandex publish keeps resetting and never finishes on slow mobile
  networks."** Smoking gun from the user's screenshot: attempt 2 died at
  exactly **33.8 MB of 40.8 MB at 113 KB/s** - that is 113 KB/s x 300 s =
  33.9 MB. Root cause: `reqwest::ClientBuilder::read_timeout(300s)` measures
  time since the **last byte received from the server**, but during a long
  HTTP PUT the server sends nothing until the whole body is uploaded. Any
  upload taking longer than 300 s (40 MB needs ~370 s on 3G) was therefore
  killed mid-transfer even though bytes were flowing fine, and every retry
  hit the same 300-second ceiling - publish could never complete.
- Fix in `ntfry/src/seeder.rs`:
    - new `upload_client()` **without** `read_timeout` used for large archive
      PUTs (Yandex + Mail.ru); `http_client()` keeps `read_timeout(300s)` for
      downloads and short control calls where it is correct;
    - new `put_with_stall_guard()`: watches **actual stream progress** instead
      of server silence - the attempt lives as long as bytes keep leaving the
      socket (no speed ceiling on EDGE/3G), aborts only if the stream makes no
      progress for 90 s (`UPLOAD_STALL`, dead TCP) or the server stays silent
      for 300 s **after the last byte** (`FINALIZE_WAIT`, server-side
      checksum/replication window);
    - shared `resilient_put()` (3 attempts, backoff, permanent-4xx
      short-circuit) now backs both `yandex_upload_seed_file` and
      `mailru_upload_seed_file`, removing ~120 lines of duplicated retry code;
    - new `StorageError::Stalled` variant, classified as transient
      (retryable) in `storage::is_transient`.
- New test: `resilient_put_streams_body_reports_progress_and_succeeds` -
  real HTTP PUT against a local TCP server verifying streaming, progress
  callback reaching 100% and 201 handling (27 tests total, all green).

### Added
- **Upload ETA in the publish progress bar** (`Seeders.vue`): the live line
  now shows the estimated time remaining at the current speed - "45.2 MB /
  93.3 MB (−69%) · 113 KB/s · ~2 min remaining…". Once the last byte has been
  sent it switches to "server processing…" so a slow 3G upload is clearly
  alive rather than frozen during the server-side finalization window.

## [1.90.3] - 2026-07 (Detect external cloud wipe & auto-reset Sled index cache)

### Fixed
- **"Deleted /ntfry folder in the cloud, and it reappeared on the next
  open."** Root cause in `Vault::open`: when `remote_index_sha256()` returned
  `None` (index.enc absent because the user wiped the folder from the
  provider's web UI), the code silently kept the stale encrypted index from
  the local `sled` cache. `init_vault_dir()` then re-created `/ntfry`, and
  the very next `push` / `mkdir` / `save_index` re-uploaded the ghost index
  with references to already-deleted chunks - resurrecting the deleted
  folder with unusable phantom files.
- Fix: `Vault::open` now distinguishes three cloud states explicitly:
    1. `Ok(Some(rsha))` - cloud has index.enc → normal validate + sync (unchanged);
    2. `Ok(None)` + `cache.synced_sha().is_some()` - we DID sync before but the
       cloud is now empty → **cloud reset detected externally**, wipe the
       local `sled` cache via new `Cache::reset()` and start a clean vault;
    3. `Err(_)` - transient network error, keep the local cached index
       (offline mode), never wipe on connectivity blips.
- New public API: `Cache::reset()` - clears `index_enc`, `synced_sha`, and
  `stored_bytes` keys atomically.
- Regression test `open_detects_external_cloud_wipe_and_resets_cache`
  covers the exact user-reported flow (init → push → external
  `delete_vault` → re-open → assert empty index + fresh `sled` state).
  All 26 `ntfry` unit tests pass.
- Per-account isolation of the `sled` cache directory was already in place
  (`cloudvault.rs::cache_name(provider, account)` → `yandex-user@example.com`),
  so no changes needed for the second half of the plan.

## [1.90.2] - 2026-07 (HOTFIX: revert broken Yandex chunked upload)

### Fixed
- **Yandex Disk seeder upload was looping indefinitely**, with the progress
  bar oscillating between advancing and resetting for 30+ minutes on a
  single 40 MB archive. Root cause: v1.90.0/.1 introduced a chunked
  resumable upload assuming Yandex Disk supports multi-request PUTs with
  `Content-Range`. **It does not.** Per official docs, the upload URL
  expects a SINGLE PUT with the entire file body; `202 Accepted` in that
  response set means "file received by the server, not yet moved to the
  user's Disk" (async server-side processing), NOT "partial chunk accepted".
  `412 Precondition Failed` is returned when `Content-Range` does not cover
  the whole file. Multi-chunk PUTs were therefore either rejected (looping
  through `412 → new URL → 412 …`) or overwrote each other. Reverted
  `yandex_upload_seed_file` to a single WebDAV PUT with 256 KB streamed
  progress, retry × 3, and the `read_timeout(300s)` from v1.89.6. Removed
  the `ResumeStore` trait, `YandexResumeState`, `SledResumeStore`, and the
  stale `seeder_resume:y:*` sled entries left over from the broken build
  (one-shot cleanup on first publish).
- True resume for Yandex remains architecturally impossible via the Disk
  REST API. If we need real cross-restart resume in the future, it would
  require **Yandex Object Storage** (S3-compatible multipart upload), a
  separate paid service.

## [1.90.1] - 2026-07 (Cross-restart resume for Yandex chunked seeder upload)

### Added
- **Persistent-resume for Yandex Disk chunked uploads** - chunk offset is
  saved to the shared `sled` DB after every confirmed 4 MB chunk. When the
  app is closed / crashed / lock-screen-terminated mid-upload, the next
  publish call for the same file resumes from the last confirmed byte
  instead of restarting at zero. State is a `YandexResumeState` JSON blob
  (`href`, `offset`, `total`, `data_hash`, `url_created_at`) under key
  `seeder_resume:y:<disk-path>`. Guards: SHA-256 `data_hash` must match
  (a re-sealed archive with fresh network state invalidates resume) and
  `url_created_at` must be < 29 minutes old (Yandex's upload URLs die at
  30 min). Stale / mismatched entries are wiped on next attempt. On
  completion / permanent 4xx / 412 range-mismatch the entry is cleared.
- `ntfry::seeder::ResumeStore` public trait (byte-oriented `load`/`save`/
  `clear`) - keeps the ntfry crate free of a sled dependency at this layer;
  the tauri host wires in `SledResumeStore` backed by the shared `DB`.

## [1.90.0] - 2026-07 (Browser-like nav bar + Yandex chunked resumable seeder upload)

### Added
- **Chrome-style Back / Forward / Reload buttons in the address bar.** Three
  round SVG icon buttons rendered on the leftmost side of the top bar (inside
  `.tb-center`, with a vertical divider), matching the visual convention of
  every mainstream browser. When a dApp tab is active, the buttons drive the
  history of the native child webview itself (SPA route stack inside
  `embed-<addr>` / `app-<addr>` via `history.back/forward()` and
  `location.reload()`), so users navigate exactly the dApp they see. On system
  pages the buttons fall back to `vue-router` history and `window.location.reload()`.
  New Tauri commands: `dapp_history_back`, `dapp_history_forward`, `dapp_reload`.
  New bridge helpers: `dappHistoryBack/Forward/Reload`. i18n keys:
  `topbar.back`, `topbar.forward`, `topbar.reload` (ru/en/id).
- **`open_app_webview_new` Tauri command + `openAppWindowNew` bridge helper**
  - always spawns a NEW native window with a unique label (`app-<addr>-<nanos>`)
  for the same dApp. Reserved for future "open in new window" flows; the
  address-bar `⧉` button still uses the classic single-window open.
- **Yandex Disk seeder upload is now chunked & resumable** (4 MB chunks over
  the official `GET /v1/disk/resources/upload` → `PUT href` REST API with
  `Content-Range: bytes X-Y/TOTAL`). Broken chunks retry independently with
  exponential backoff - already-confirmed chunks are NEVER re-uploaded on
  transient network drops. Only when three chunk retries fail does the client
  request a fresh upload URL (30 min validity) and restart from zero. On a
  40 MB archive over 3G/EDGE, a mid-upload disconnect now wastes at most one
  4 MB chunk instead of the whole 40 MB. `Content-Range` support is
  officially documented by Yandex Disk (`412 Precondition Failed` on mismatch).
  Progress inside each chunk is streamed at 256 KB granularity for a smooth UI
  progress bar; the "attempt N/3" badge only advances on a full URL reset.

### Fixed
- **Yandex Cloud Seeder retried uploads that had already reached 100%.**
  `seeder.rs::http_client()` had `read_timeout(60s)`, which fired while the
  server was performing post-body finalization (checksum, replication) on
  large encrypted archives - reqwest returned an error and the whole file was
  re-queued. `read_timeout` raised to `300s`; dead sockets still caught by
  `tcp_keepalive(20s)` + `connect_timeout(15s)`. Together with the chunked
  resumable upload above, spurious retries are effectively eliminated on
  reliable connections and drastically reduced on mobile ones.

## [1.89.3]  (Mail.ru seeder: seamless one-time public-link flow)

### Changed
- **Mail.ru seeder publishing is now a single one-time step for VK ID
  accounts.** WebDAV upload (identical to Crypto Vault) always runs; only the
  "publish folder" call fails for VK ID logins (Mail.ru does not accept app
  passwords on its web API - verified: o2 rejects the app password, web login
  requires a captcha). When it fails, the UI shows an "Open ntfryseed folder"
  button + a link field: the user enables the public link once, pastes it, and
  the weblink is remembered forever - every later publish and background
  auto-replication runs fully automatically over WebDAV (no o2/api-m1 calls).
  `cloud.mail.ru` added to the external-link allow-list so the folder opens in
  the system browser.

## [1.89.2]  (Mail.ru seeder publish: User-Agent fix + manual weblink fallback)

### Fixed
- **o2.mail.ru rejected token requests with `missing user-agent header`** -
  the seeder HTTP client now always sends a User-Agent, so the unofficial
  publish API works for regular Mail.ru accounts.

### Added
- **Manual weblink fallback for Mail.ru publishing** - if automatic folder
  publishing is unavailable (e.g. VK ID logins), the user shares the
  `ntfryseed` folder once in the Cloud web UI and pastes the public link
  (`https://cloud.mail.ru/public/...`) on the Seeders page. The weblink is
  extracted, remembered, and every subsequent publish (including background
  auto-replication) goes over WebDAV only - no o2/api-m1 calls at all.
- Verified live against a real published folder: anonymous listing and
  chunk-file download (via dispatcher `weblink_get` + redirect follow) work
  without any authentication; missing files return proper 404.

## [1.89.1]  (Mail.ru seeder publish: real API errors surfaced)

### Fixed
- **`HTTP: error decoding response body` when publishing a seeder to Mail.ru
  Cloud** - o2.mail.ru and api/m1 responses were parsed with strict JSON
  decoding, so any non-token answer (e.g. `{"error":"invalid username or
  password"}` for VK ID logins, or an HTML page) crashed with a meaningless
  decode error. Token/publish/meta responses are now parsed leniently and the
  actual server response body (first 220 chars) is included in the error
  message, so the real cause is visible in the UI.

## [1.89.0]  (Mail.ru Cloud Seeders + configurable chunk size)

### Added
- **Mail.ru Cloud as a Public Network Seeder provider** - new seed URI format
  `sn://seed/m@<folder_weblink>@<public_key>`. Authors publish the encrypted
  network state to `/ntfryseed` over WebDAV and the folder is made public via
  the unofficial `api/m1` publish endpoint (o2.mail.ru password-grant with the
  app password). Peers fetch files anonymously through the public weblink
  (`weblink_get` base + optional download key) - no login required. Fetching,
  dApp cloud-install and the registry fully support `m@` URIs.
- **Provider picker before publishing a seeder** - the Seeders page now lists
  every connected Yandex Disk and Mail.ru Cloud account from Crypto Vault;
  background auto-replication re-publishes to the same provider as the
  original seeder.
- **Configurable vault chunk size** - new `.env` key `CHUNK_SIZE` (in KB,
  clamped to 64–4096, default **512**). 512 KB cuts HTTP round-trips 4× versus
  the old 128 KB, keeps deduplication effective and gives ZSTD more context
  per block. Existing 128 KB chunks remain fully readable (decompression
  bound raised to a provider-agnostic 4 MiB safety cap).

## [1.88.2]  (Crypto Vault: vault-level retry for all cloud providers)

### Added
- **Automatic retry of transient cloud errors for every provider** - chunk
  upload/download and index upload in the vault engine now retry transient
  failures (network/transport errors, HTTP 429/5xx) up to 3 times with
  backoff (1 s → 3 s → 7 s). This extends the Mail.ru-specific hardening from
  1.88.1 to Yandex Disk, Google Drive and RAID mirrors, so a single dropped
  connection no longer aborts a large multi-chunk transfer.

## [1.88.1]  (Crypto Vault: resilient Mail.ru WebDAV transfers)

### Fixed
- **Large-file upload failure on Mail.ru Cloud** - pushing a big file
  (hundreds of 128 KB chunks over 6 parallel PUTs) made Mail.ru WebDAV drop
  connections (reset / stale keep-alive, 429/5xx); a single
  `error sending request` aborted the whole upload. Chunk PUT/GET/DELETE now
  retry transient failures up to 4 times with exponential backoff
  (0.5 s → 4 s), and the HTTP client's `pool_idle_timeout` (25 s) is kept
  below Mail.ru's server-side keep-alive so dead pooled sockets are never
  reused.

## [1.88.0]  (Crypto Vault: visible delete errors, safe chunk GC, instant UI)

### Fixed
- **Silent delete failures on Yandex Disk / Google Drive** - `Vault::delete`
  discarded every result of the cloud chunk-GC futures (`let _ = gc.next()`),
  so a 401/403/5xx from the provider was swallowed: the file disappeared from
  the index while its encrypted chunks stayed on the cloud drive forever.
  GC results are now inspected per chunk; failures surface to the UI as
  `VaultError::GcFailed` with the exact provider error (status + body).
- **Permanent chunk orphaning** - dedup entries were removed from the index
  *before* the cloud delete was attempted. On failure the chunk became
  untracked and could never be GC'd again. Entries are now removed only
  after a successful `delete_chunk`, so a retry of the delete re-runs GC.
- **Breadcrumb path duplication** - rapid double-clicking a folder row while
  the previous `refresh()` was still in flight appended the same segment to
  `cwd` twice (`/docs/docs`). `openRow()`/`goCrumb()` are now guarded by a
  `navBusy` navigation lock.

### Changed
- **Optimistic file deletion** - the deleted row disappears from the list
  instantly; chunk GC runs in the background. On error the previous listing
  is restored and the exact error is shown.

## [1.87.0]  (Crypto Vault: fully async Sled cache + non-blocking IPC on drive switch)

### Fixed
- **Drive-switching freeze in Crypto Vault** - synchronous `sled::open()` +
  every sled read/write (`load_index_enc`, `save_index_enc`,
  `set_synced_sha`, `set_stored_bytes`, `synced_sha`, `stored_bytes`) was
  running on the tokio IPC thread, blocking the whole Rust command runtime
  while `Vault::open` cold-opened a per-account Sled cache (recovery walk
  can take hundreds of ms). Symptom: switching drives in the vault drive
  switcher froze the UI for a noticeable moment; the "occupied by vault"
  batch scan across all drives compounded the freeze.

### Changed
- **`ntfry::cache::Cache` is now fully async**: every method
  (`open`/`load_index_enc`/`save_index_enc`/`synced_sha`/`set_synced_sha`/
  `stored_bytes`/`set_stored_bytes`) wraps sled I/O in
  `tokio::task::spawn_blocking`. `sled::Db` is `Send + Sync + Clone`, so
  the handle is cheaply cloned into the blocking task; every fsync/flush
  now happens off the main runtime thread.
- **`Vault::open` / `Vault::init` / `Vault::save_index`** - all Cache
  calls awaited (no more sync sled ops from an async context).
- **`cloud_vault_open` / `_reset` / `_open_raid` / `_reset_raid`**:
  BIP32/HKDF/secp256k1 key derivation (`vcrypto::derive_flexible`) is
  CPU-bound and was running on the IPC thread - moved into
  `spawn_blocking`.
- **`detect_onedrive_folder`, `cloud_vault_seal`, `cloud_vault_unseal`**:
  converted from sync `#[tauri::command] fn` to `async fn` + inner
  `spawn_blocking`. 
- **`cloud_vault_cached_stored`**: simplified - the new async `Cache` API
  handles blocking internally, so the manual `spawn_blocking` wrapper
  around `Cache::open + stored_bytes` was removed.

### Verified
- `cargo check --lib` in `ntfry` - clean; `cargo test --lib` - **22/22
  passed**. `cargo check` in `smartnet-client` - clean (only pre-existing
  warnings; no new errors).


## 1.86.0 
- CRYPTO VAULT: new "Mail.ru Cloud" provider (official WebDAV webdav.cloud.mail.ru):
  - ntfry/storage/mailru.rs: MailruCloudProvider - Basic authentication (email + app password), full StorageProvider trait (chunks, index.enc, user.dict, meta, GC, RAID compatibility)
  - Quota: RFC 4331 PROPFIND (quota-used/available-bytes) + fallback to unofficial o2.mail.ru → api/m1/user
  - Timeouts based on seeder-fix lessons: connect 15s, read 60s, tcp_keepalive 20s
  - Clear errors: 401 → "check app password", 402/403 → "WebDAV Mail.ru only on paid Mail Space plans"
  - UI: "＋ Mail.ru Cloud" button + email/password form with credential validation BEFORE adding account, hint about app password and paid plan; Mail.ru added to "Add by token (advanced)" (format email:password)
  - ⚠ NOT VERIFIED with a real account: the user does not have a paid Mail Space subscription (WebDAV Mail.ru is paid)


## [1.86.0]  (Crypto Vault: Mail.ru Cloud provider via WebDAV)

### Added
- **Mail.ru Cloud (`cloud.mail.ru`) provider in Crypto Vault** - new
  `MailruCloudProvider` in `ntfry::storage::mailru` implementing the full
  `StorageProvider` trait over `webdav.cloud.mail.ru/`. Uses HTTP Basic Auth
  (`email:app_password`, base64-encoded on every request); chunk upload/
  download/exists/delete via WebDAV `PUT`/`GET`/`PROPFIND`/`DELETE`; folder
  init via `MKCOL` (409 = exists = ok). Same encrypted layout as Yandex/
  OneDrive (`/ntfry/{index.enc, chunks/, meta, user.dict}`), so RAID-eligible
  and fully interchangeable with existing providers.
- **Mail.ru UI in `CryptoVault.vue`**: new `＋ Mail.ru` button next to
  Yandex/Google/OneDrive with a two-field form (email + app password). The
  frontend concatenates `email:app_password` into a single `token` string
  before calling `cloud_vault_open`, so the Rust IPC surface stays uniform
  across providers. Provider badge `MR`, orange/blue accent matching the
  Mail.ru brand.
- **`make_provider("mailru")`** wired in `cloudvault.rs` factory; `mailru`
  added to the account list serializer and `account_info` login resolver
  (returns the email address as login, total/used quota via `PROPFIND` on
  the root - Mail.ru does not expose a REST quota endpoint).

### Notes
- Mail.ru WebDAV requires a **paid Mail Space** subscription and an
  **App Password** (created in Mail.ru account security settings). The
## 1.85.0 
- UI: badge "Attempt N/3" in the archive upload progress line on the Seeders page - when mobile connectivity is lost, the user sees that the client is retrying the upload rather than being stuck:
  - ntfry: progress callback `on_progress(sent, attempt)` now passes the attempt number
  - Rust IPC: `attempt` field in the `seeder-publish-progress` event (attempt changes are propagated to the UI immediately, bypassing 450ms throttling)
  - Seeders.vue: pulsating amber badge (data-testid="seeders-publish-retry-badge") when attempt > 1

## [1.85.0]  (Cloud Seeders: live retry badges in the publish UI)

### Added
- **Live "Attempt N/3" retry badges** on the Seeders publish progress:
  each per-archive line now surfaces the current retry attempt when the
  Rust backend is inside its 3-attempt upload loop (added in 1.84.0 for
  mobile-internet resilience). `SeederPublishProgress` payload gained an
  `attempt` field (`1..=3`); the badge renders amber during retries
  (`Attempt 2/3 · retrying…`) and green on the final success frame, so
  users see the reconnect logic actually working instead of a frozen
  progress bar.

### Changed
- `cloud_seeder_publish` (Tauri IPC) dispatches per-attempt progress
  events on retry boundaries; `bridge.ts` `SeederPublishProgress`
  interface + `useSeedersStatsStore` updated accordingly.

## 1.84.0 
- FIX: Mobile connectivity loss during Cloud Seeder publish no longer hangs the upload indefinitely:
  - `http_client`: added `read_timeout(60s)` and `tcp_keepalive(20s)` (reqwest has no default request timeout - a broken TCP would hang forever)
  - dApp archive upload: total attempt timeout = 60s + size/8KiB/s, up to 3 attempts with backoff (2s/4s/6s); 4xx (except 408/429) - no retries
  - Meta files (dapps/nodes/seeders.enc): timeout 60s + 3 attempts
  - All control requests (MKCOL, publish, meta, DELETE, href): timeout 30s
- AUDIT (P1): confirmed - `cloud_seeder_publish` requires an unlocked wallet (`signing_key()` → SESSION_SEED) BEFORE any cloud operations; the token alone does not allow publishing


## [1.84.0]  (Cloud Seeders: origin badges, sidebar heartbeat, legacy cleanup)

### Added
- **Sidebar SEEDERS nav item with live "⇈ N" badge** (DEV/GEEK modes):
  every successful background Cloud Seeder replication (see
  `useSeedersStatsStore.tickReplicated()`) bumps a session counter that
  renders as the item's badge. Gives the operator a visual heartbeat of
  the fallback layer without waiting for the toast. Counter is reset on
  `lock()` so it always reflects the current session.
- **Origin badge for every entry in the KNOWN SEEDERS registry**: each
  URI now carries one of `own / manual / gossip / seeders.enc / bootstrap`
  so users understand where the seeder was learned from. New Sled table
  `cfg:cloud_seeder_origins` and IPC `cloud_seeder_registry_origins_get`
  back the UI; `registry_add_internal` records origin at insertion time
  (`"own"` always overrides). `cloud_seeder_registry_set` reconciles the
  origin map when the user removes URIs (drops orphans, tags new manual
  additions).
- **Auto-delete legacy `dapp-<id>.enc` from root of /ntfryseed**: after
  every successful publish cycle we iterate archive candidates and issue
  `yandex_delete_seed_root_file(token, "dapp-<sanitized_id>.enc")`. 404
  is idempotent no-op; any other errors are logged but never fail the
  publish. Cleans up the pre-1.83 flat layout so only the structured
  `/ntfryseed/dapps/<id>.enc` remains.

### Changed
- `registry_add_internal(&[...])` → `registry_add_internal(&[...], source)`
  - all three call sites (publish/gossip/seed_file merge) pass their
  origin; the bootstrap defaults are stamped in `known_seeders_internal`.

## [1.83.1] - Mobile-network resilience for Cloud Seeder uploads

### Fixed
- Cloud Seeder publish hanging **forever** after a silent mobile-network
  drop: the ntfry HTTP client now sets `read_timeout` (60s) and TCP
  keepalive (20s); all short control requests (MKCOL, publish, meta,
  DELETE, public href) get a 30s timeout.
- Meta files and dApp archives are uploaded with a per-attempt timeout
  (archives: 60s + size at 8 KiB/s minimum speed) and up to **3 retries**
  with backoff; permanent 4xx errors are not retried.
- Background replication no longer dead-locks: `cloudSeederPublish` is now
  guaranteed to resolve, releasing the `inFlight` guard so the 60s tick
  retries automatically.

### Security
- Verified: `cloud_seeder_publish` requires an unlocked wallet session
  (`SESSION_SEED` via `signing_key()`) - publishing is impossible without
  a valid session.

## [1.83.1]  (Cloud Seeders: clickable replication toast)

### Added
- **Clickable "⇈ SEEDER - replicated to cloud" toast**: the background
  replication toast now carries an `onOpen` handler that closes any active
  dApp tab (`tabs.goHome()`) and routes to `sn://seeders`. ToastHost already
  renders a "click to open →" hint and hover accent for
  clickable toasts, so users get one-click access to the Cloud Seeders
  dashboard right after a successful publish.

## [1.83.0]  (Cloud Seeders: auto URI exchange, /dapps subfolder, version-aware install)

### Added
- **Global "⇈ SEEDER - replicated to cloud" toast**: background Cloud Seeder
  replication (30-min timer and the instant post-update trigger) was fully
  silent; AppLayout now listens to `seeder-replicated` and shows a toast
  with the dApps/archives counters, so users see the update actually landed
  in /ntfryseed.
- **Instant cloud replication after every dApp change**: `replicateNow()`
  was wired only into `finalize` and the simple `update` flow. The prepaid
  wallet-driven update (`updateFromPath`) and free metadata edits
  (`editMetadata`) silently fell back to the 30-minute background timer.
  Both now trigger an immediate Cloud Seeder publish (merkle gate still
  skips unchanged archives, so only the updated dApp is re-uploaded).

### Added
- **Automatic seeder-URI exchange over P2P gossip**: a new system topic
  (`SMARTNET-SEEDER-ANNOUNCE-BUS-v01`) carries signed announces
  `{v,kind,uri,ts,sig}` - ECDSA over sha256("seeder-announce|uri|ts") with
  the wallet key; receivers verify the signature against the pubkey embedded
  in the URI itself (spam with foreign URIs is rejected) and a ±1h freshness
  window, then merge the URI into their local registry. Announce fires
  immediately after every successful publish and every 5 minutes while the
  overlay is alive, so a brand-new seeder becomes visible to the whole
  network without manual URI sharing.
- **seeders.enc merge on EVERY fetch**: previously foreign seeder lists were
  merged into the registry only during Cloud Fallback; now manual FETCH BY
  URI, "check all" and cloud installs merge them too (chain reaction). The
  UI live-refreshes via the new `seeder-registry-updated` event.
- **Structured cloud layout - /ntfryseed/dapps/**: encrypted dApp archives
  are now published into a dedicated `dapps` subfolder as `<appId>.enc`
  (e.g. `dapps/SPVSeEN3ZXTUH2Y6EwoPhR2sj6TxYoQZXo.enc`, MKCOL on publish);
  fetch/install tries `dapps/<appId>.enc` first, then legacy
  `dapps/dapp-<id>.enc` and the flat root `dapp-<id>.enc` (backward
  compatible with older seeders).
- **Version-aware cloud install**: `APP_SOURCES` now tracks the dApp
  `version` advertised by each seeder's dapps.enc; `cloud_seeder_install_any`
  sorts candidates by version (newest first), excluding stale archives from
  lagging seeders. dApp version + merkle were already stored in the cloud
  metadata; the author's archive is always overwritten in place so only the
  latest version lives in each seeder's folder.

## [1.82.0]  (Cloud Seeders: live upload speed)

### Added
- **Live cloud-upload speed on the Seeders page**: dApp archive uploads to
  Yandex Disk are now streamed in 256 KiB zero-copy `Bytes` chunks
  (`reqwest::Body::wrap_stream`, explicit `Content-Length` for WebDAV), with
  a per-chunk progress callback throttled to ~2 events/sec. The publish
  progress line shows real-time transferred bytes and speed - "uploading
  45.2 MB / 93.3 MB (−69%) · 2.4 MB/s…" - and the progress bar advances by
  actual bytes sent within the current archive, matching the Private Cloud
  UX. `SeederPublishProgress` gained an `uploadedBytes` field.
- **Session average upload speed chip** in the /ntfryseed session summary
  strip: "⇈ SPEED · X KB/s", volume-weighted across all archives uploaded
  this session (manual publish and background replication alike).

## [1.81.0]  (Cloud Seeders: per-session /ntfryseed summary bar)

### Added
- **Session-scoped /ntfryseed statistics on the Seeders page**: a compact
  summary strip appears once any dApp archive has been uploaded in the
  current session - "N archives · ~X MB · saved Y MB · avg. compression
  −Z%". Aggregation is fully local, driven by the existing
  `seeder-publish-progress` events (both manual publish and background
  30-min replication feed it); archives are de-duplicated by `appId` so
  re-uploads never inflate the count. Values reflect what actually landed
  in `/ntfryseed` on Yandex Disk after zstd + ChaCha20-Poly1305, plus the
  raw-vs-packed delta to make the compression payoff visible at a glance.

## [1.80.0]  (Discover: one-click "via seeder" install)

### Added
- **Cloud-archive install fallback in Discover**: the INSTALL button now
  transparently falls back to encrypted dApp archives from known Cloud
  Seeders - (1) when a discovered app has no live P2P providers, and
  (2) after every P2P provider fails. New `cloud_seeder_install_any(appId)`
  command tries seeders that are known to host the app's archive first
  (in-memory `APP_SOURCES`, filled on every seeder fetch), then the whole
  registry; integrity is enforced by `verify_app` after unpack. Storage-limit
  errors keep triggering the standard "increase storage" prompt.

## [1.79.0]  (Cloud Seeders: dApp archives in the cloud + statuses)

### Added
- **Encrypted dApp archives in /ntfryseed**: `cloud_seeder_publish` now also
  uploads `dapp-<id>.enc` - a tar of each non-draft dApp folder, sealed with
  the same public-deterministic pipeline (HKDF key from the author's pubkey,
  ChaCha20-Poly1305, ECDSA author signature; new binary envelope v2
  `seal_bytes`/`open_bytes` in `ntfry::seeder`, 5/5 tests). Unchanged apps
  are skipped via a merkle gate (`cfg:seeder_uploaded`), so the 30-minute
  background republish stays cheap. Per-archive failures are logged and do
  not fail the publish.
- **Install dApps straight from a cloud seeder** (no P2P needed): new
  `cloud_seeder_install(uri, appId)` command - anonymous download, signature
  verification, storage-quota check, unpack + Sled registration; the UI then
  runs `verify_app`. FETCH BY URI on the Seeders page lists the seeder's
  dApps with an "⛁ archive" badge and an INSTALL button.
- **Instant replication on publish/update**: finalizing or updating a dApp
  triggers an immediate seeder republish (`replicateNow()`), respecting the
  auto-replication toggle.
- **Known Seeders health**: every fetch (manual or fallback) records
  uri → {ok, checkedAt, lastOk, error} (`cfg:cloud_seeder_status`); the
  registry shows a green/red/gray dot + last successful fetch time and a
  "CHECK ALL" button.

### Changed
- `tar` is now a non-optional dependency of the Tauri crate (cloud install
  works without the `p2p` feature).

## [1.78.0]  (Cloud Seeders Phase 3b)

### Added
- **Cloud Fallback (auto)**: 7 s after discovery starts, if no live P2P peers
  are present, the client fetches every known Cloud Seeder URI from the Sled
  registry, merges nodes into the PEERS table (`seeder:` key prefix, marked
  `viaSeeder`), surfaces the publisher's dApps on the Discover page with a
  teal "via seeder" badge, and appends newly learned seeder URIs to the
  registry (chain reaction). Re-checks every 5 minutes until P2P recovers;
  every step is logged to the System Console (`seeder` scope).
- **Background chain replication**: while the wallet is unlocked, a frontend
  timer republishes the user's network state to their own `/ntfryseed`
  Yandex folder every 30 minutes - only if a Yandex account is connected in
  Crypto Vault AND the user has published a seeder before. ON/OFF switch +
  last-publication timestamp on the Seeders page (`sth_seeder_autorep`,
  `sth_seeder_own`, `sth_seeder_last_rep` in localStorage; shared helpers in
  `lib/seederRep.ts`, started from `AppLayout`).
- **QR codes for Seeder URIs**: `[ QR ]` button next to `[ COPY ]` for the
  freshly published URI and for every entry in the Known Seeders registry
  (inline render via the existing `qrcode` package).

- **Bootstrap seeder anchors from `.env`**: new `CLOUD_SEEDERS=<uri>[,…]`
  key in `frontend/.env`. Native: runtime env var wins, otherwise the line is
  baked into the binary at build time (`include_str!`); seeded into the Sled
  registry only on first run (user removal is respected). Web fallback seeds
  the same list into localStorage via Vite env.

### Changed
- `PeerInfo` gained a `viaSeeder` flag (Rust + TS); Discover aggregation
  marks an app "via seeder" only when ALL of its providers came from the
  cloud fallback.

## [1.76.3] - 2026-07-07

### Fixed
- **Non-p2p build unblocked (E0433 in `p2p.rs` + `devhub.rs`).**
  Three call sites referenced modules that only exist under
  `#[cfg(feature = "p2p")]` - `crate::dht_dapp::album_infohash` inside
  `announce_album_keys` / `album_keys_discover` (`p2p.rs`), and
  `crate::chat::gossip::app_handle` inside `handle_gossip_announce`
  (`devhub.rs`). Default `cargo check`/`cargo build` (feature off) failed
  with "cannot find `dht_dapp` in the crate root" / "cannot find `gossip`
  in `chat`", which blocked global workspace builds and CI on the
  `smartnet-client` crate. Fix: gate the p2p bodies with
  `#[cfg(feature = "p2p")]` and add no-op stubs under
  `#[cfg(not(feature = "p2p"))]` (returning `(Vec::new(), None)` for the
  discover call). Production builds (`--features p2p`) are unchanged -
  same code path, identical behavior.

## [1.76.2] - 2026-07-07

### Added
- **OneDrive auto-detect: one-click add of the standard synced folder.**
  Clicking "＋ OneDrive" now first calls a new Rust command
  `detect_onedrive_folder` which checks the official `%OneDrive%`,
  `%OneDriveConsumer%`, `%OneDriveCommercial%` env vars (Windows) and the
  home directory for `~/OneDrive` and any `~/OneDrive - <tenant>` variants
  (macOS/Linux WSL/SMB mount). First existing folder wins and is added in
  one click; falls back to the native picker if nothing is detected.
  Bridge: `detectOneDriveFolder()` wrapper. Hint text updated in the UI.

## [1.76.1] - 2026-07-07

### Changed
- **OneDrive rework: local synced folder instead of Graph API (user decision).**
  The Microsoft OneDrive desktop client already mirrors a local folder to the
  cloud, so no API/OAuth is needed at all. "＋ OneDrive" now opens the native
  folder picker (like Local Safe Disk): the chosen folder is registered as
  provider `onedrive` (token = path, label `OneDrive · <name>`), `/ntfry` with
  encrypted chunks is created inside and OneDrive itself uploads them.
  Rust: `make_provider("onedrive")` → `LocalStorageProvider` over the path;
  `cloud_vault_account_info` measures the folder's disk partition
  (`local_disk_info`), same as local drives. RAID-eligible, dedupe by path.

### Removed
- Graph API provider `ntfry/storage/onedrive.rs` and OAuth commands
  `cloud_vault_onedrive_oauth`/`_refresh` (+ bridge wrappers, manual-token
  option, `NTFRY_ONEDRIVE_CLIENT_ID`) - introduced in 1.76.0, superseded by
  the folder-based approach before ever shipping to users.

### Verification
- `cargo check`/`cargo test` in `ntfry` - clean, 14/14 PASSED; full workspace
  `cargo check` - no errors in changed code (only the 3 known pre-existing
  `E0433` in `p2p.rs`/`devhub.rs`); `yarn build` - clean; web-preview smoke of
  the Settings modal (button + updated hint) OK.

## [1.76.0] - 2026-07-07

### Added
- **Crypto Vault: OneDrive provider (Microsoft Graph API v1.0).** New
  `ntfry/storage/onedrive.rs` implements the full `StorageProvider` trait via
  path addressing (`/me/drive/root:/ntfry/…`): chunk upload/download (simple
  `PUT :/content`, 250 MB limit - ntfry chunks are 128 KB), index/dict/meta,
  cheap `chunk_exists`, tolerant delete (404 = ok), folder init with
  `conflictBehavior: fail` (409 = exists), drive quota via `$select=quota`.
  `remote_index_sha256` returns `None` (Graph exposes only sha1/quickXor).
  `account_info` = quota from `/me/drive` + login (userPrincipalName) from `/me`.
- **OneDrive OAuth (public client, PKCE, no secret).** New Tauri commands
  `cloud_vault_onedrive_oauth` / `cloud_vault_onedrive_refresh`: system browser
  → `login.microsoftonline.com/common` (work + personal accounts) → loopback
  `http://localhost:{ephemeral port}` → code exchange with `code_verifier`.
  Scope: `Files.ReadWrite offline_access User.Read`. Client ID comes from the
  `NTFRY_ONEDRIVE_CLIENT_ID` env var; a clear setup-instruction error is shown
  when it is missing. UI: `＋ OneDrive` button, `OD` provider badge, manual
  token option, auto-refresh 60 s before expiry - fully symmetric with
  Yandex/Google drives, RAID-eligible.
- **Clickable "⛁ … in vault" chip → "new since last sync" filter.** The vault
  usage chip in the drive dropdown is now a button: click switches to that
  drive and shows a flat recursive list of files uploaded after the drive's
  previous connect (per-drive `syncMarks {prev,last}` persisted in
  `ntfry.settings`; `modified` in the index is the push timestamp). Filter bar
  shows the cutoff date and file count with a "✕ show all" reset; rows show
  full VFS paths (click = open containing folder), download and 2-step delete
  work directly from the filtered list. Filter clears on reconnect/drive switch.

### Verification
- `cargo check` + `cargo test --lib` in `ntfry` - clean, 14/14 PASSED.
- Full workspace `cargo check`: **no errors in new code**; only the 3 known
  pre-existing `E0433` errors in `p2p.rs`/`devhub.rs` (feature-gated modules).
- `yarn build` - clean; UI smoke-verified in web preview (Settings modal shows
  the OneDrive button and provider hint).

## [1.75.1] - 2026-07-06

### Added - Crypto Vault: "in vault" line in the drive list
- **"Occupied by vault" row**: each drive row in the vault drive-switcher dropdown
  now shows a second line under the free-space label - how many bytes the
  Zero-Knowledge vault currently occupies on that specific disk. Instant read
  from the local Sled index cache - no network round-trip and no need to
  open the vault on that drive.
- **Live update**: the active drive (or every member of the active RAID set)
  is updated in real time after every `push` - the number reflects the fresh
  `stats.storedBytes` from the running session without waiting for the
  dropdown to be re-opened.
- **`ntfry/cache.rs`**: new `stored_bytes()`/`set_stored_bytes(u64)` helpers.
  A small plain-text hint (8-byte LE integer) is written next to the encrypted
  `index_enc` blob. Metadata only - no vault contents are revealed.
- **`ntfry/vault.rs`**: `save_index()` and the initial cloud-sync branch in
  `open()` update the hint after every mutation of the index.
- **Rust IPC**: new command `cloud_vault_cached_stored(provider, account) -> u64`.
  For the currently active session the value is served from RAM; otherwise a
  short-lived Sled handle opens the per-account cache directory, reads the
  hint and drops the handle so subsequent `open()` calls don't hit the file
  lock. Missing cache directories return 0.
- **Frontend (`CryptoVault.vue`, `bridge.ts`)**: `cloudVaultCachedStored()`
  wrapper; `refreshAllQuotas()` fans out a parallel read of the hint for
  every drive in the list; a `watch` on `status.stats.storedBytes` keeps the
  active drive/RAID members up to date instantly.

## [1.75.0] - 2026-07-06

### Added - Crypto Vault: "+ Local Safe Disk" (folder as encrypted virtual drive)
- **Local Safe Disk**: the "+ Local mock" button is replaced by
  "+ Local Safe Disk". Clicking it opens the native folder picker; the chosen
  folder becomes a full encrypted virtual drive - a `/ntfry` directory with
  `index.enc`, `meta` and ChaCha20-Poly1305-encrypted ZSTD chunks is created
  inside it, exactly like on a cloud drive (Zero-Knowledge all the way down).
- **Any number of local disks**: each picked folder is an independent drive
  (dedup by path; re-adding the same folder just switches to it). The drive
  list shows the folder path as the account login and the free space of the
  partition hosting the folder (`sysinfo` longest-mount-point-prefix match).
- **RAID with local disks**: Local Safe Disks are RAID-capable - they get the
  ⛓ chip and can be combined with cloud drives in RAID-0 (stripe) or RAID-1
  (mirror) sets.
- **Rust (`cloudvault.rs`)**: `make_provider("local")` now roots
  `LocalStorageProvider` at the folder passed in `token`; empty token keeps
  the legacy `data/ntfry/mock_cloud` sandbox, so previously added mock drives
  keep working. `cloud_vault_account_info("local", path)` returns
  login = path + partition total/used via `spawn_blocking` (non-blocking).

### Removed
- The "+ Local mock" button (legacy mock accounts continue to work).



### Added - Profile: domain expiration + inline "Renew"
- **Expiration dates in profile "My Domains"**: every `.sth` row now shows
  `DD.MM.YYYY · X d.` with an amber (`≤ 30 d`) / red (`≤ 7 d`) tint mirroring
  the accent used in `DomainManager` / `NodeTelemetryPanel`. Works both on
  own profile (`listMyDomains`) and on visited profiles (`domainsByOwner`).
- **Inline "Renew" button** (own profile only), rendered strictly inside the
  backend renewal window (30 days before `expires` and up to 30 days grace
  after). Click reveals a confirm form with the network `renewalPrice()`,
  balance check, and a one-shot `renewDomain(name, appId)` call - reuses the
  existing `nodePanel.domRenew*` i18n keys so no new locales required.
- Renew click doesn't bubble to `openDomain` anymore - the dApp bound to the
  name no longer pops open when the user only wanted to extend the lease.

### Fixed - Details modals no longer bleed the page through
- **Discovered app details modal** (`Discovered.vue`): the `.disc-modal`
  surface used a non-existent `var(--surface)` and rendered fully
  see-through. Switched to an opaque `var(--panel, #14171c)` with a stronger
  backdrop blur - background page no longer bleeds through the card.
- **Node telemetry panel** (`NodeTelemetryPanel.vue`): dApp side-panel now
  uses a solid `#0d1014` surface instead of a semi-transparent gradient,
  fixing the ghosted look when opened over the dApp iframe.



## [1.73.0] - 2026-07-25

### Added - Crypto Vault: RAID-1 re-sync + 15s connect timeout
- **RAID-1 re-sync** (`ntfry`): new `StorageProvider::chunk_exists` (cheap
  metadata probes for Yandex / Google Drive / local) and `resync` on
  `RaidProvider` - for every chunk of the index it finds mirror members
  missing a copy and re-uploads the data from any healthy member, then
  re-replicates `index.enc` and `meta`. Unreachable members are skipped
  (never abort the repair). Stats: `checked / copied / failed / lost`.
- **`cloud_vault_raid_resync`** Tauri command (progress via `vault-progress`,
  op = `resync`) + `cloudVaultRaidResync()` bridge binding.
- **RE-SYNC chip** in the vault header: shown ONLY when the mirror is
  degraded (`ok < total`); hidden entirely in the OK state. Shows toast with
  repair stats and re-checks set health when done.
- **15-second connect timeout**: if a drive connection hangs longer than 15s
  the drive chip switches from `loading…` to `offline · retry?` and a red
  `⟳ retry` chip appears next to it; both backend cloud clients now use a
  15s `connect_timeout` so hung sockets fail fast.
- New Rust test `raid_mirror_resync_restores_lost_chunks` (full drive-loss →
  resync → read back exclusively from the repaired drive). 14/14 tests pass.

## [1.72.0] - 2026-07-25

### Added - Crypto Vault: RAID health indicator + per-drive quota chip
- **RAID set health chip** in the header (`⛨ MIRROR 3/3` / `⛨ SET 2/3`):
  pings every drive of the set via `account_info`, green when all reachable,
  pulsing red when degraded (tooltip lists offline drives); click re-checks,
  auto-recheck every 60s. Per-drive quotas refresh with each check.
- **Active drive quota chip**: in RAID mode the header now shows both the
  aggregate quota (`TOTAL · X free of Y`) and the currently selected drive's
  own free space (`<label> · X free`).

## [1.71.0] - 2026-07-25

### Added - Crypto Vault: RAID member selection + RAID-1 (mirror) mode
- **Selectable RAID set**: every cloud drive row in Settings has a ⛓ chip -
  only checked drives join the RAID set (e.g. 3 of 5 connected). Minimum 2;
  the rest keep working as independent vaults.
- **RAID type selector**: RAID-0 (stripe, summed quota - default) or
  **RAID-1 (mirror)**: every chunk is uploaded to ALL drives of the set,
  the vault survives the total loss of any drive; usable quota reported as
  the member with the least free space. `RaidMode` enum in ntfry,
  `mode` param on `cloud_vault_open_raid`/`reset_raid`, session key =
  `mode:sortedIds`. New unit test `raid_mirror_replicates_and_survives_drive_loss`
  (kills drive A, pulls the file back from drive B) - 13/13 GREEN.
- Header chip now shows `⛓ RAID-0/1 · N drives`; dropdown row shows the
  active type and `N of M` selected drives.

## [1.70.0] - 2026-072026-06-24

### Added - Crypto Vault: PKCE, RAID mode, folder drag&drop queue, .env overrides, RU docs
- **PKCE (RFC 7636)** in both OAuth flows: `code_challenge S256` in the
  authorize URL + `code_verifier` on token exchange (Yandex & Google) -
  intercepted authorization codes are useless without the verifier.
- **RAID mode (optional toggle)**: all cloud drives merge into ONE big vault.
  New `ntfry::storage::raid::RaidProvider` places chunks via rendezvous
  hashing (HRW, no mapping table), falls back to the next drive when one is
  full/down, replicates `index.enc`/`meta`/`user.dict` to every member and
  sums quotas. New IPC `cloud_vault_open_raid` / `cloud_vault_reset_raid`.
  UI: switch in Settings + "⛓ RAID - all disks together" row in the drive
  dropdown; picking a single drive turns RAID off. Unit test
  `raid_two_drives_distribute_and_roundtrip` (12/12 GREEN).
- **Folder drag & drop with upload queue**: native OS drops are scanned
  recursively (`cloud_vault_scan_paths`, hidden files skipped), empty
  folders are recreated via mkdir and all files enter a visible queue
  (N/M files, bytes progress, current file, cancel). Batch speed lands in
  the SAVINGS panel.
- **.env overrides for OAuth apps**: tiny env-file loader at startup reads
  `.env` next to the executable / in cwd; `NTFRY_YANDEX_CLIENT_ID/SECRET`,
  `NTFRY_GDRIVE_CLIENT_ID/SECRET` can now be swapped without rebuilding.
  Template `.env` shipped in `src-tauri/`.
- **Full Russian documentation**: `docs/11-Crypto-Vault-Guide.md` - setup,
  multi-account, RAID, drag&drop, security model, own OAuth apps, FAQ
  (including "why the client secret is not secret for desktop apps").

## [1.69.0] - 2026-072026-06-24

### Added - Crypto Vault: multi-drive / multi-account support
- **Multiple cloud drives side by side** (e.g. 2×Yandex + 2×Google): every
  connected drive is an independent encrypted vault with its own `/ntfry`
  container and its own local Sled cache (`cache/<provider>-<accountId>`).
  `cloud_vault_open`/`cloud_vault_reset` accept an `account` id;
  `cloud_vault_status` reports it so a live session is only reused for the
  same drive.
- **Drive switcher** in the header: dropdown listing all drives (provider
  badge, label, login, free space, active marker) with one-click switching.
- **Drives manager** in Settings: auto-detected account login (new IPC
  `cloud_vault_account_info` → Yandex `GET /v1/disk`, Google `about.get`)
    + user-editable label, per-drive quota, activate/remove (two-step) rows,
      "＋ Yandex / ＋ Google / ＋ Local mock" OAuth buttons and an advanced
      "add by token" row. Re-authorizing the same login updates tokens in place
      (dedup by provider+login).
- **Sealed multi-account settings**: the whole accounts array (tokens
  included) is stored ChaCha20-encrypted with the wallet master key; legacy
  single-drive settings (plaintext and sealed v1) migrate automatically.

## [1.68.0] - 2026-072026-06-24

### Added - Crypto Vault phase 5: WebDAV fast path, disk quota, transfer speed, ZSTD dictionary, sealed tokens
- **Yandex Disk WebDAV transport**: hot-path transfers (chunks, index.enc,
  user.dict, meta) switched from the two-phase REST flow (fetch href → send
  bytes) to single-transaction WebDAV PUT/GET against `webdav.yandex.ru` -
  one HTTP round-trip per chunk instead of two. Directories are created via
  MKCOL. REST is kept for what WebDAV cannot do: remote `index.enc` sha256,
  disk quota, and permanent (trash-bypassing) deletes.
- **Updated Yandex OAuth app credentials** (new app with both WebDAV API and
  REST API scopes granted).
- **Disk quota display** (`cloud_vault_quota` + `StorageProvider::quota`):
  Yandex `GET /v1/disk` and Google `about.storageQuota` report total/used
  bytes; the UI shows "X free of Y" as a header chip and in the settings
  modal.
- **Transfer speed metric**: upload/download MB/s of the last operation is
  measured in the UI and rendered as a SPEED chip in the SAVINGS panel.
- **ZSTD dictionary management** (`cloud_vault_dict_train/status`,
  `Vault::train_dict/set_dict`): trains a COVER dictionary on a user-picked
  folder (≥8 samples, in-RAM only), stores it as `/ntfry/user.dict`,
  auto-loads it on open (second device included). Replacing a dictionary is
  blocked while files compressed with it exist. New unit test
  `dict_train_set_and_roundtrip` (11/11 GREEN).
- **Sealed OAuth tokens** (`cloud_vault_seal/unseal`): tokens in
  localStorage are now ChaCha20-Poly1305-encrypted with the wallet master
  key; plaintext legacy settings are migrated transparently.

## [1.67.0] - 2026-072026-06-24

### Added - Crypto Vault phase 4: parallel transfers, Google OAuth, VFS mkdir/rm + GC, token auto-refresh, key fingerprint
- **Parallel chunk pipeline (up to 6 concurrent transfers)**: `push` fills a
  bounded pool of concurrent uploads (FuturesUnordered), `pull` uses an
  order-preserving `buffered` stream (concurrent downloads, sequential
  writes) - 3-6× faster on multi-chunk files, storage format unchanged.
  `Vault.storage` became `Arc<dyn StorageProvider>`.
- **"Sign in with Google"** (`cloud_vault_google_oauth`): RFC 8252 loopback
  flow on an ephemeral 127.0.0.1 port (Google forbids embedded webviews and
  allows any loopback port without registration), system browser consent,
  code exchange with the embedded desktop client credentials
  (`NTFRY_GDRIVE_CLIENT_ID/SECRET` override), returns access + refresh
  tokens. GoogleDrive provider is now selectable end-to-end (live test
  pending on the user's machine).
- **Token auto-refresh**: OAuth responses now include `refresh_token` +
  `expires_in`; new IPC `cloud_vault_yandex_refresh` /
  `cloud_vault_google_refresh`. The UI persists refresh tokens and silently
  renews the access token when it expires within 60s before connecting.
- **VFS mkdir / delete with chunk GC**: explicit (empty) directories stored
  in `VaultIndex.dirs`; `Vault::delete` removes a file or a directory
  recursively and garbage-collects orphaned chunks in the cloud (new
  `StorageProvider::delete_chunk`). CLI gains `ntfry mkdir` / `ntfry rm`.
  UI: "＋ Folder" inline creator and per-row two-step delete buttons.
- **Key fingerprint in `/ntfry/meta`**: a public one-way fingerprint
  (`sha256("ntfry-fp-v1"‖key)[..16]`) written on init and self-healed on
  open; a foreign-wallet vault is now detected instantly without
  downloading the index. New `download_meta`/`upload_meta` on providers.

### Fixed - sled cache lock on reconnect ("could not acquire lock on vault_index.sled")
Reported when navigating away mid-upload and pressing Connect: a second
Sled handle was opened while the old session still held the file lock.
`cloud_vault_open`/`cloud_vault_reset` now acquire the session mutex FIRST
(waiting for any in-flight push), drop the old session and only then open
the cache. The UI also reuses a still-alive backend session on page
re-entry instead of blindly reopening.


## [1.66.2] - 2026-072026-06-24

### Fixed - Crypto Vault: clear KeyMismatch error + "Reset cloud vault" recovery
The user hit a raw "AEAD error" on Connect: the cloud `/ntfry/index.enc`
(left over from the live CLI test) was encrypted with a different mnemonic
than the wallet key derived by the client.

- **ntfry**: new `VaultError::KeyMismatch` - `Vault::open` maps an AEAD
  failure while decrypting the freshly downloaded remote index into a
  semantic "the cloud vault belongs to another wallet" error. New trait
  method `StorageProvider::delete_vault()` (Local: `remove_dir_all`;
  Yandex: `DELETE resources?path=disk:/ntfry&permanently=true`; GDrive:
  folder delete by ID). Unit test: wrong key → KeyMismatch, then
  `delete_vault` + reopen succeeds (9/9 green).
- **Client**: new IPC `cloud_vault_reset` - wipes the remote `/ntfry`,
  clears the local Sled cache and re-inits an empty vault for the CURRENT
  wallet key. UI shows an amber recovery panel on KeyMismatch with a
  two-step confirm button ("Reset cloud vault (erase /ntfry)" → "Click
  again to ERASE").
- Housekeeping: the leftover test `/ntfry` was removed from the user's real
  Yandex Disk, so a plain Connect now initializes a fresh vault.


## [1.66.1] - 2026-072026-06-24

### Changed - "Sign in with Yandex" without a local server (embedded webview interception)
The user's Yandex OAuth app has a locked redirect URI
(`oauth.yandex.ru/verification_code`, "cannot be changed"), which rules out
both the loopback server and custom-scheme deep links. Replaced the flow:
the consent page now opens in an embedded Tauri webview window; an
`on_navigation` hook intercepts the redirect to
`verification_code?code=…`, grabs the code automatically (the final page
is never even shown), closes the window and exchanges the code for an
access token. Zero manual copying, zero Yandex app reconfiguration, no
localhost listener. Cancelling (closing the window) is detected via the
dropped oneshot channel; 5-minute timeout guards abandoned sessions.


## [1.66.0] - 2026-07-23

### Added - Crypto Vault phase 3: Yandex Disk LIVE-verified, "Sign in with Yandex" OAuth, GoogleDriveProvider
- **Yandex Disk verified against the real cloud**: full CLI E2E on a live
  user token - `init` (creates `disk:/ntfry/{index.enc, chunks/}`), `push`
  (3 encrypted chunks uploaded), `ls`, `pull` → byte-identical restore, plus
  a second-device scenario (fresh cache syncs the index from the cloud).
- **"Sign in with Yandex"** (`cloud_vault_yandex_oauth`): built-in OAuth
  authorization-code flow with a loopback interceptor. The system browser
  opens the Yandex consent page, the client catches the redirect on
  `http://127.0.0.1:8971/callback`, exchanges the code for an access token
  (client id/secret embedded, overridable via `NTFRY_YANDEX_CLIENT_ID/SECRET`)
  and returns it to the UI - zero manual token copying. The settings modal
  gains a "Sign in with Yandex" button that auto-fills the token, saves and
  reconnects. NOTE: the redirect URI must be registered in the Yandex app.
- **GoogleDriveProvider implemented** (Drive API v3, Bearer token): lazy
  folder-ID resolution for `/ntfry` + `/ntfry/chunks`, multipart create /
  media PATCH overwrite, download via `alt=media`, remote index validation
  through the native `sha256Checksum` field. NOT live-tested yet (needs a
  Google OAuth token); selectable in the UI with a manual token.
- tokio gains the `net` feature (loopback listener); ntfry tokio gains `sync`.


## [1.65.1] - 2026-07-23

### Changed - Crypto Vault: fully seamless key source (wallet phrase only)
The mnemonic input was removed from the Vault settings modal. The master
encryption key is now ALWAYS derived from the current SmartNet wallet
phrase (`SESSION_SEED`) - zero extra passwords, seamless by design. The
modal shows a green "KEY SOURCE · SmartNet wallet (seamless)" badge and
only asks for the storage provider + OAuth token.


## [1.65.0] - 2026-07-23

### Added - Crypto Vault phase 2: Tauri IPC integration, live VFS UI, savings metrics
The ntfry Zero-Knowledge vault is now wired into the SmartNet client - the
CryptoVault page went from a skeleton to a fully working encrypted file
manager backed by the Rust core.

- **Crate `ntfry` → lib + bin**: new `src/lib.rs` exposes the core to the
  client; the CLI keeps working via the library. `Vault::open` now
  idempotently ensures the remote `/ntfry/` layout. `push`/`pull` accept an
  optional progress callback `(done_bytes, total_bytes)`. New
  `crypto::derive_flexible`: a valid BIP-39 phrase uses the standard path,
  any other wallet secret falls back to `SHA-512("ntfry-flex-seed"‖secret)`
  as the BIP-32 seed (SmartNet wallets allow arbitrary passphrases).
- **Savings metrics** (`VaultIndex.stats`, `#[serde(default)]`): cumulative
  `raw_bytes` (logical), `stored_bytes` (physical ciphertext in the cloud)
  and `dedup_saved_bytes`, updated on every push. The CLI prints
  `vault totals: raw … · stored … · saved …%` after each push.
- **Backend (`cloudvault.rs`)**: new IPC commands `cloud_vault_open` /
  `close` / `status` / `ls` / `push` / `pull`. The open session lives in a
  tokio-Mutex singleton; an empty mnemonic falls back to the unlocked
  SmartNet wallet secret (`SESSION_SEED`). `pull` opens the native
  "Save As" dialog. Chunk progress streams via the `vault-progress` event;
  operations are logged to the SystemConsole under the `vault` scope.
  `tokio` is now a non-optional dependency (was p2p-gated).
- **Frontend (`CryptoVault.vue` + bridge)**: live VFS listing from the
  in-memory index with folder navigation and breadcrumbs, Upload via the
  native file picker (chunk → zstd → chacha20 → cloud), per-file
  "decrypt & save" download, animated per-chunk progress bar
  (encrypt/upload and download/decrypt), a "ZERO-KNOWLEDGE SAVINGS" panel
  (raw / in-cloud / saved% / dedup chips + ratio bar), connect/refresh
  controls, error surface and result toasts. Web preview shows a
  desktop-only banner. Bridge gains typed `cloudVault*` wrappers.


## [1.64.0] - 2026-07-23

### Added - `ntfry` Virtual Encrypted Cloud Storage (Zero-Knowledge) - phase 1
New standalone Rust crate `/app/ntfry/` + UI skeleton: an In-Memory Encrypted
Virtual File System over personal cloud drives. Files never touch the cloud
(or local disk) in plaintext - only ChaCha20-Poly1305 ciphertext chunks.

- **Crate `ntfry` (CLI, clap v4)**: commands `init` / `push` / `pull` / `ls`.
  Pipeline: 128 KB chunk split → ZSTD compression (optional custom
  `user.dict`) → ChaCha20-Poly1305 AEAD → upload to `/ntfry/chunks/<sha256>`.
  Chunk-level deduplication via pre-encryption SHA-256 map.
- **Key derivation (BIP-39/BIP-32)**: master encryption key at
  `m/999'/0'/0'/0` (HMAC-SHA512 + secp256k1), ETH-compatible blockchain
  address at `m/44'/60'/0'/0/0` for wallet binding.
- **`StorageProvider` trait** with three impls: `LocalStorageProvider`
  (zero-network mock), `YandexDiskProvider` (REST API, two-phase
  upload/download, remote sha256 validation) and `GoogleDriveProvider`
  (skeleton - phase 2). Remote layout: `/ntfry/{index.enc, user.dict, chunks/}`.
- **Sled local cache**: encrypted `VaultIndex` snapshot loads instantly into
  RAM on startup; background validation compares remote `index.enc` SHA-256
  with the last synced hash and re-syncs when the cloud copy is newer.
  Mutations persist to Sled first, then push to the cloud.
- **Unit/E2E tests**: BIP-39 derivation vector (`0x9858…da94`), AEAD
  tamper detection, ZSTD dict roundtrip, full pipeline roundtrip with dedup
  and a second-device sync scenario.
- **Frontend**: new terminal-style "VIRTUAL ENCRYPTED VAULT" card on the
  Storages page + `CryptoVault.vue` skeleton page (`/vault` route): virtual
  file list placeholder and a settings modal (mnemonic, provider,
  Yandex/GDrive OAuth tokens, persisted to localStorage). Wiring to the
  ntfry Rust core over Tauri IPC lands in phase 2.


## [1.63.20] - 2026-07-22

### Added - `verify_app` returns per-file reason + streams failures to SystemConsole
The red "content.json signature does NOT match" banner used to be silent -
you'd see the ⚠, but not WHICH file was broken (`missing-file` /
`file-hash-mismatch` didn't carry a path). Devs of large dApps had no way
to jump straight to the file at fault.

- **Backend (`apps.rs::verify_app`)**: on failure `reason` now carries the
  offending file path when applicable: `missing-file:assets/logo.png`,
  `file-hash-mismatch:index.html`, `id-mismatch:<other-id>`. Every failure
  (all seven exit points) also logs one line to `crate::syslog::log("dapp",
  format!("verify <shortid>… - FAIL: {reason}"))` - visible in the
  SystemConsole under the "dapp" scope filter. The old `verify_fail`
  helper is kept as `#[allow(dead_code)]` for backwards import safety.
- **Frontend (`store/tabs.ts`)**: the `Tab` type gains a `verifyReason?:
  string` field, filled in whenever `verifyApp` returns `verified=false`.
  Failures are also mirrored to `console.warn("[SmartNet verify] <id> -
  <reason>")` so the browser DevTools log picks them up too.
- **Frontend (`AppLayout.vue`)**: the ⚠ tab badge tooltip now appends the
  reason: `WARNING · content.json signature does NOT match the app address
  · missing-file:assets/logo.png`. No wall-of-text, just enough to
  pinpoint the file.

## [1.63.19] - 2026-07-22

### Fixed - Free logo replacement broke `content.json` signature ("signature does NOT match the app address")
Field report: a user opened a big (~300 MB) dApp for a FREE metadata edit,
swapped the logo, saved - and the next time they opened it the red banner
"WARNING: content.json signature does NOT match the app address" showed
up. Root cause in `edit_app_metadata_blocking` (`apps.rs`):

- The new logo bytes were written to `<id>-logo.<ext>` and the old logo file
  names (`<id>-logo.{png,svg}`, `logo.{png,svg}`) were `remove_file`'d from
  disk, but `manifest.files` was left untouched and `manifest.merkle` was
  NOT recomputed. The re-sign was then done over the OLD `merkle`.
- On the next `verify_app`, the walker iterated `manifest.files` and
  re-hashed each entry from disk - the old logo entry either resolved to
  the NEW file bytes (`file-hash-mismatch`) or, if the extension changed,
  resolved to nothing (`missing-file`). Either way → `verified=false` →
  red banner. This affected any dApp whose initial ingest src folder had
  contained a logo file (which is the common case).

Fix (`apps.rs`):
- New helper `rescan_dest_files_and_merkle(dest_root)` walks the destination
  folder recursively, hashes every file (SHA-256 for `manifest.files`,
  Blake3 for the merkle) and returns the fresh `(files, merkle)` - mirrors
  what `collect_and_copy` does on initial ingest, but reads from `dest_root`
  instead of the dev src folder. Skips `content.json` itself.
- `edit_app_metadata_blocking` now ALWAYS rescans + refreshes
  `manifest.files` and `manifest.merkle` from the on-disk state, then
  re-signs over the NEW merkle. Doing it unconditionally (not only when a
  new logo is uploaded) self-heals any dApp that was already broken by a
  prior buggy edit - the next FREE text edit fixes it, no re-upload needed.
- Cost: ~few seconds of disk IO for a 300 MB dApp on a metadata edit -
  acceptable for a rare operation. No change to publish/update paths.

Preserves existing behaviour: text-only edits produce the identical merkle
(disk state didn't change), so the signature only "moves" when the actual
content on disk changed.

## [1.63.18] - 2026-07-22

### Added - Reserved domain names moved to `reserved.json` + `DEVELOPERS` env bypass
Domain reservation list is no longer hard-coded in two places. Both the web
preview (`bridge.ts`) and the desktop client (`dns_manager.rs`) now read from
the same file `frontend/public/reserved.json`:

- **Frontend (`bridge.ts`)**: `const DOMAIN_RESERVED = [...]` replaced by a
  lazy `loadReservedDomainNames()` that fetches `/reserved.json` once and
  caches the result. Kicked off at module init so `webDomainLookup()` has the
  list ready by the time the user types a name.
- **Backend (`dns_manager.rs`)**: `RESERVED_NAMES` const array replaced by a
  `once_cell::Lazy<Vec<String>>` built at start-up from
  `include_str!("../../public/reserved.json")` - one source of truth, no drift
  between the two runtimes.
- **`.env` - new `DEVELOPERS=<addr>,<addr>` list** (exposed to the frontend
  via `vite.config.js` `envPrefix`). Addresses in this list bypass the
  reserved-name gate: they see reserved labels as `available` in the lookup
  and can pass the `register_domain` and `commit_record` checks. Everyone
  else still sees `reason=reserved`. Enforced on the Rust side via
  `can_register_reserved(name, addr) = !is_reserved(name) || is_developer(addr)`
  - belt-and-braces, so a raw on-chain tx from a non-developer can't hijack
  a reserved label either.

To edit the reserved list, update `frontend/public/reserved.json` and rebuild
the desktop client.

## [1.63.17] - 2026-07-22

### Added - "Check my relays" button next to the n1-relay URL field
Field request: after the 1.63.16 fix, users of custom n1-relays (self-hosted
`smartnet-relay` boxes) still had no cheap way to test whether their own
relay is reachable from the current network - the full Network Doctor sweep
takes ~30 seconds and probes DHT/DNS/QUIC on top. Added a targeted button
directly next to the n1-relay textarea in Settings > Network:

- **UI**: `Settings.vue` - new button `CHECK MY RELAYS`
  in `.tracker-actions` (data-testid `check-n1-relay-btn`). Results render in
  the same style as the main doctor's steps (green/amber/red dot + detail),
  one row per URL entered, with per-family v4/v6 timings (`v4 82ms · v6 74ms`).
- **Backend (Rust, `p2p.rs`)**: new `net_doctor_check_relays(urls)` Tauri
  command reuses the existing `probe_relay_tls443` (pure TCP/TLS 443, SNI by
  hostname, dual-stack A + AAAA with DoH fallback for the missing family) -
  no QUIC/UDP, so carriers that DPI-block UDP don't skew the answer.
  Emits per-URL `net-doctor-step` events (id `myrelay:<host>`) so the UI can
  paint results live as each probe finishes in parallel.
- **Bridge**: `netDoctorCheckRelays(urls)` in `lib/bridge.ts`.

Handles empty input (shows warn "list is empty"), URLs with or without the
`https://` prefix, and preserves the order the user typed them in. Blocking
`ureq` runs are wrapped in `spawn_blocking` - the async runtime stays clean.

## [1.63.16] - 2026-07-22

### Fixed - Network Doctor "Relays (TLS 443)" probe reported 0/2 on carriers that kill all UDP
Field report: a mobile carrier drops ALL UDP/QUIC (both v4 and v6), and the
TLS-443 relay probe returned `0/2` even though the SmartNet relay servers
(`relay-fsn7.sth.cx`, `relay-ru1.sth.cx`) were fully healthy over HTTPS.
Root causes on the client side (`p2p.rs`):

1. **Wrong relay list**: the probe only tested the two hardcoded n0 relays;
   the operator's SmartNet relays were consulted only when the opt-in
   n1-relay toggle was ON (it wasn't). Fixed: new built-in constant
   `DEFAULT_SMARTNET_RELAYS` (`relay-fsn7.sth.cx`, `relay-ru1.sth.cx`) and
   `effective_relay_urls()` = built-ins + user's n1 relays (when enabled).
   The doctor now probes SmartNet relays FIRST, then the n0 pair.
2. **IPv4-only resolution**: the probe used the system resolver via ureq's
   default path - carrier DNS could return garbage/A-only, and a blackholed
   IPv4 ate the whole 5s timeout so IPv6 was never tried. Fixed with an
   honest dual-stack probe (`probe_relay_tls443`): A and AAAA are resolved
   separately (system DNS first, missing family backfilled via DoH to the
   IP-literal `https://1.1.1.1/dns-query` - immune to carrier DNS), then
   IPv4 and IPv6 are probed IN PARALLEL and independently.
3. **Probe honesty guaranteed**: each family probe is a real `TcpStream`
   connect to the resolved IPs + real rustls TLS handshake with SNI set to
   the DOMAIN name (fixed ureq resolver overrides only the socket
   addresses, never the hostname) + HTTP GET; any HTTP status (even 4xx)
   counts as reachable. Strictly TCP 443 - no QUIC/UDP involved anywhere.
4. **Per-host verdict in the report**: detail line now shows each relay as
   `host: v4 51ms v6 32ms` / `v4✗ v6 28ms`. When IPv4 TCP is blocked but
   IPv6 TCP works, the probe is marked OK with an explicit
   "IPv4 TCP blocked - relays available via IPv6 TCP (bypassing CGNAT)" note.

### Changed - SmartNet relays are now built into the endpoint relay map by default
- `apply_smartnet_relays()` no longer requires the n1-relay toggle: the two
  built-in `relay-*.sth.cx` relays are ALWAYS inserted into the iroh relay
  map next to the n0 defaults (user relays remain opt-in). With UDP fully
  blocked, iroh's relay data path is WebSocket over TLS/443 (TCP), so
  traffic flows through these relays automatically - no manual switch
  needed. Stealth mode is untouched: it keeps its strict custom relay map
  (the apply helper is skipped when stealth is active).
- Doctor verdict text updated accordingly ("SmartNet relays are built in,
  traffic will route through them automatically").
- Settings hint (`peers.n1RelayHint`, RU/EN) now mentions the built-in
  relays; the textarea is for EXTRA user relays.
- Note on direct TCP between IPv6 nodes: iroh direct paths are QUIC/UDP
  only; with a carrier dropping ALL UDP, direct connections are impossible
  regardless of family - connectivity is carried by the TLS-443 relays.
  A custom TCP peer-to-peer transport (iroh `unstable-custom-transports`)
  is a separate future work item.

## [1.63.15] - 2026-07-22

### Added - Phase 2 mobile-network fallbacks: DoH resolver + pkarr HTTPS discovery (Rust backend)
Backend half of the mobile-internet hardening (Russian carriers: CGNAT,
DPI dropping UDP, DNS tampering). The Network Doctor (v1.63.13/14) can
now *diagnose* these conditions - this release makes the node actually
*route around* them, automatically or on demand, without touching
healthy networks.

- **New setting `cfg:mobile_fb`** (Sled) with three modes exposed via new
  Tauri commands `get_mobile_fallbacks` / `set_mobile_fallbacks`:
    - `auto` (default) - at endpoint bind time an async 2.5s probe resolves
      an n0 relay hostname through the *system* DNS; only if that fails are
      the fallbacks activated. Healthy connections keep the exact previous
      endpoint configuration - zero behavior change.
    - `on` - fallbacks always forced (for networks where DNS "works" but is
      poisoned/filtered selectively).
    - `off` - never activated.
- **DoH resolver** (`p2p.rs::doh_resolver`): custom iroh `DnsResolver`
  speaking DNS-over-HTTPS (RFC 8484) to IP-literal nameservers
  (Cloudflare `1.1.1.1`/`1.0.0.1`, Google `8.8.8.8` on :443, certs carry
  IP SANs). When active it carries ALL of iroh's DNS traffic - `_iroh`
  TXT discovery lookups and relay hostname resolution - over TCP/443,
  invisible to carrier UDP-DNS filters.
- **pkarr HTTPS discovery fallback**: `PkarrResolver::n0_dns()` added as
  an extra address-lookup service on the default endpoint. EndpointIds
  are then also resolved via direct HTTPS requests to the n0 pkarr relay,
  bypassing DNS entirely. Additive: lookups race concurrently with the
  N0 preset's DNS discovery, so nothing gets slower when DNS is fine.
- **TCP relay note**: iroh 1.0 relay data connections are already
  WebSocket over TLS/443 (TCP) - with UDP fully blocked, traffic flows
  through relays without extra configuration; the QUIC probes in
  net-report only affect diagnostics, not the data path.
- **Stealth interop**: in stealth mode only the DoH resolver is applied
  (helps resolve n1-relay hostnames on filtered DNS). The pkarr HTTPS
  lookup via n0 infrastructure is deliberately NOT added - stealth keeps
  its strict custom-relay isolation, stealth relays keep priority.
- Activation is logged to the System Console
  (`mobile fallbacks ACTIVE (mode=...)`).

### Added - Settings → Network: "MOBILE INTERNET (FILTER BYPASS)" segmented control
- New `AUTO / ON / OFF` segmented switch (`data-testid="mobile-fb-row"`,
  buttons `mobile-fb-auto|on|off`) between the relay-only toggle and the
  n1-relay block. Shows the "applies after restart" flash on change.
- `bridge.ts`: `getMobileFallbacks()` / `setMobileFallbacks()` +
  `MobileFallbacksMode` type (web fallback returns `auto`).
- i18n keys `peers.mobileTitle/mobileHint/mobile_auto/mobile_on/mobile_off`
  in RU / EN / ID.
- User guide: new section 18.2 documenting the mobile-fallback modes.

## [1.63.14] - 2026-07-22

### Added - Network Doctor UI card + "Copy report" button (Settings → Network)
The Network Doctor logic and CSS were shipped in v1.63.13 but the template
markup itself was accidentally omitted, so the card was invisible in the
built app. This release restores it and adds a one-click "Copy report"
button so users can paste the full diagnostic output into support chats,
GitHub issues, or the operator's Telegram without retyping every step.

- `Settings.vue`: new `<div class="net-doctor">` block placed right after
  the trackerless-DHT diagnostics grid. Renders the "Run diagnostics"
  button, the live per-step list (colored dot + probe title + detail),
  and a monospaced "Verdict" row at the bottom.
- New copy button (⧉) appears next to the title once results are
  available. `copyDoctorReport()` builds a plain-text report
  (`SmartNet Network Doctor - <iso>` header + client version + one
  `[STATUS] TITLE - detail` line per probe) and writes it to the OS
  clipboard via `navigator.clipboard.writeText`. Falls back gracefully
  with a red hint if clipboard access is denied.
- New i18n keys `peers.docCopy` / `peers.docCopied` / `peers.docCopyFail`
  (ru + en). Testids `net-doctor-run-btn`, `net-doctor-copy-btn`,
  `net-doctor-steps`, `net-doctor-step-<probeId>`, `net-doctor-copy-flash`.

### Added - User guide section explaining what the Network Doctor does
`docs/USER_GUIDE.md` now has a dedicated "Network Doctor"
section under the Settings chapter, describing each probe (IPv6 outbound,
dual-stack bind, UDP/QUIC, NAT type, QUIC/TLS relays, Mainline DHT,
DNS vs DoH) and how to read the three possible verdicts (direct-capable,
relay-only, dead) so end users can self-troubleshoot restrictive mobile
carriers without asking support.


## [1.63.13] -25

### Added - Network Doctor: mobile-internet / DPI diagnostics (Settings → Network)
Russian mobile carriers combine CGNAT (symmetric NAT), DPI that drops
"unrecognized" UDP (QUIC), and DNS tampering - the client could silently
fail to connect. A new one-click **Network Doctor** card runs ~30s of
probes with live per-step progress (`net-doctor-step` events) and a final
verdict, all mirrored into the SystemConsole (scope=net):

- **IPv6 outbound** - direct CGNAT bypass when both peers have v6.
- **Dual-stack bind audit** - actual `endpoint.bound_sockets()` dump.
- **UDP/QUIC** - iroh `net_report` QAD probes (new `unstable-net-report`
  feature flag on the iroh dependency).
- **NAT type** - `mapping_varies_by_dest()`: symmetric (mobile CGNAT) vs
  cone, plus the discovered public address.
- **Relays over QUIC** - reachable relay count + best latency.
- **Relays over TCP/TLS 443** - the last-resort transport when UDP is
  fully blocked (any TLS response counts, incl. configured n1 relays).
- **Mainline DHT** - live routing-table size + a direct BEP5 `ping` to
  bootstrap routers (detects UDP:6881-style blocking).
- **DNS vs DoH** - system resolver compared against Cloudflare DoH
  (detects carrier DNS filtering).
- **Verdict** - direct-capable / relay-only (enable n1/stealth) / dead.

New Tauri command `net_doctor_run`; frontend `netDoctorRun()` +
`NetDoctorCheck` in `bridge.ts`; card UI in `Settings.vue` (network tab)
with `net-doctor-*` test ids; i18n keys (ru/en).

### Fixed - dApp DHT identity beacon was IPv4-only (`dht_dapp.rs`)
The BEP5 seeder beacon bound `0.0.0.0` only, so two mobile nodes with
IPv6 could never verify each other directly. `bind_dual_udp()` now binds
a single `[::]` socket with `v6only=off` (explicit via `socket2` - the
Windows default is v6only=on) accepting both families, with a clean
v4-only fallback on systems without an IPv6 stack. Applies to the
ephemeral DHT beacon and the fixed LAN beacon ports; the client-side
`probe_ex()` now binds a family-matching socket for v6 beacon targets.

## [1.63.12] -25

### Fixed - ClearNet warning modal was hidden BEHIND the native dApp webview
Clicking an external `https://` link inside a dApp (e.g. `dex-trade.com`)
opened the "you are leaving to clearnet" warning dialog, but the native
child webview always floats above the Vue DOM, so the dialog appeared
underneath it and could not be interacted with.

- `AppLayout.vue` / `syncEmbedded()`: the active webview is now hidden
  while `clearnet.url` is set (same overlay pattern as `ui.overlayOpen`)
  and restored automatically when the modal is confirmed or dismissed.
  `clearnet.url` added to the sync watcher list.

## [1.63.11] -25

### Added - custom-scheme links from dApp webviews open as native host tabs
A dApp can now link to any SmartNet resource with a plain `<a href>`,
`window.open()` or `window.location.href` using the custom schemes
`sth://` (other dApps, incl. deep paths like
`sth://<APP_ID>/trading/PAIR`), `u://` (user profiles), `sn://` (system
pages / `sn://file/<cid>` / `sn://album/…`), `api://` (Netfory node
view), `f://<cid>` (file page) and `dev://` (DevHub). Instead of
navigating inside the sandboxed webview, the link now opens as a new
native tab/page in the host shell.

- `dapp.rs` bridge: `isHostLink()` + `forwardHostLink()` intercept
  custom-scheme hrefs in the capture-phase click handler and in the
  `window.open` override, emitting `dapp-open-link { app_id, url }`.
  `sth://` links to the dApp's OWN appId are left alone (internal SPA
  navigation).
- `webview.rs`: new `dapp_nav_guard()` installed via `on_navigation` on
  both embedded (`embed-*`) and windowed (`app-*`) dApp webviews - it
  catches programmatic `window.location.href = 'sth://…'` (which JS
  cannot patch), blocks the in-webview navigation and forwards the URL
  to the host with the same event.
- `AppLayout.vue`: listens to `dapp-open-link` and routes the URL
  through the existing address-bar logic (`openAddress()`), so every
  scheme behaves exactly as if typed in the URL bar. New `f://<cid>`
  branch (file page) and `tabs.goHome()` added to `api://`, `sn://file`,
  and `sn://<system-page>` branches so router pages are not hidden
  behind a floating native webview.

## [1.63.10] - 2026-07-22

### Added - live network speed indicator in the left sidebar
The static caption "SmartHoldem P2P Client · DSEC / net 63 · coin 255" at
the bottom of the sidebar is replaced by a live **▼ download / ▲ upload**
speed readout covering ALL machine network traffic (torrents, iroh QUIC,
DHT, `api://` calls, clearnet) - visible from every page.

- New Rust module `netstat.rs`: a 1-second sampler over `sysinfo::Networks`
  (non-loopback interfaces summed; loopback skipped so the local torrent
  stream server does not double-count) emitting `net-speed { rxBps, txBps }`.
  `sysinfo` gained the `network` feature flag.
- `AppLayout.vue`: `sidebar-netspeed` element (green ▼ / amber ▲, monospace)
  replaces the caption once the first sample arrives; web preview keeps the
  old caption as fallback (the caption remains as the tooltip).

### Added - SmartNet App SDK + one-click monetized publish (`/smartnet-sdk`)
A modern, fully English-commented Vue 3 template that turns any frontend
into a paid dApp in minutes:

- `useSmartNetWallet()` - connect (`getAccount`), `signMessage`,
  `sendTransaction`, `openWallet()`; shared reactive state.
- `usePayments()` - one-off purchases and subscriptions as STH transfers
  with machine-readable memos (`sub:pro:30d`, `item:skin-42`) plus on-chain
  verification helpers (`hasActiveSubscription`, `hasPurchase`) via any
  ARK-v3 node REST - the chain is the receipt database.
- `useStorage()` - P2P reads over `f://<cid>` (+ HTTPS gateway fallback)
  and a namespaced local KV for settings/save-games.
- `runtime.ts` - canonical `window.__SMARTNET__` detection and
  `api://`/HTTPS/WS endpoint switch (mirrors the ui-poker pattern).
- `example/MonetizedApp.vue` - a complete paywalled mini-app in one file.

### Added - `smartholdem.openWallet()` bridge method
dApps can now slide the host wallet panel out from their own menu:
`window.smartholdem.openWallet()` → Rust emits `dapp-open-wallet` →
`AppLayout` opens the WalletDrawer. No signing, not approval-gated. The SDK
falls back to a `getAccount()` handshake on older clients.

### Added - docs/10-SmartNet-App-SDK.md
Dedicated web-dApp integration guide: injected globals reference
(`__SMARTNET__`, `smartholdem`), Vue detection pattern, dynamic
decentralized address substitution, opening the wallet panel, payments &
subscriptions model, P2P storage, deep links and the publish checklist.

## [1.63.9] - 2026-07-21

### Fixed - `sth://<id>/<path>` deep-links with a dot in the route returned 404
Opening `sth://SYEm…/trading/XBTSX.STH_XBTSX.USDT` in a new tab showed the
cyberpunk 404 page ("Application not found") while the root
`sth://SYEm…` worked fine.

**Root cause.** The `sth://` scheme handler's SPA fallback (`protocol.rs::
serve`) treated ANY final path segment containing a dot as a static asset
(`Path::extension()` non-empty ⇒ asset ⇒ hard 404 when the file does not
exist). Trading pairs, versioned routes and similar dotted SPA routes never
reached `index.html`.

**Fix.** New `is_static_asset()` whitelist: only real web extensions
(js/css/png/woff2/wasm/… ~30 entries) count as assets. Every other missing
path - dotted or not - falls back to `index.html`, letting the dApp's
client-side router resolve the deep route. Missing genuine assets
(`/assets/x.js`) still 404 correctly.

### Fixed - periodic UI freezes around torrent activity (dev:// release pages)
On slower test machines (RDP clients) pages with live torrent data - the
dev:// release repo view in particular - periodically froze.

**Root cause.** In Tauri v2, synchronous `#[tauri::command]` functions run
**on the main thread**. ~15 torrent commands (`get_active_torrents` polled
every second, `pause/resume/remove_torrent`, `get_torrent_magnet`,
`torrent_list_files`, `get_torrent_file_bytes` (disk read + base64),
`get_profile_stats` (sled read), `get/set_torrent_limits`,
`get/set_torrent_net_settings`, `get_bt_trackers`, `get_torrent_dht_nodes`,
`torrent_stream_url`, `open_torrent_folder`) plus `devhub_feed` (full Sled
scan of all known dev indexes) executed mutex-guarded snapshots, librqbit
stats and disk I/O directly on the UI thread - any contention or slow disk
stalled every webview.

**Fix.** All of the above are now `async fn` + `spawn_blocking`, so no
torrent/devhub IPC ever touches the main thread. The frontend is unchanged
(`invoke` is promise-based either way). `album_export_torrent` (social.rs)
updated to `await` the now-async helpers.

## [1.63.8] - 2026-07-21

### Added - "Trust this host" button in the ClearNet warning modal
The ClearNet warning modal now offers a third (amber) action:
**TRUST `<host>` & OPEN**. Clicking it appends the host to a persistent
user trust-list (`localStorage: sth.trustedHosts`) and immediately opens
the link in the system browser. From then on that host behaves exactly
like an `ALLOW_HOST` entry - anchors, `window.open` and dApp-forwarded
links (`dapp-open-url`) open silently with no modal - without editing
`frontend/.env`.

- `clearnet.ts`: new `userTrustedHosts()` / `trustHost()` /
  `untrustHost()`; `allowHosts()` now merges env/default hosts with the
  user trust-list (deduplicated). The merged list is also what the
  hardened Rust `open_external` command validates against
  (defense-in-depth preserved - the trusted set is always explicit).
- `ClearNetWarningModal.vue`: amber `trust` button
  (`data-testid="clearnet-trust"`), shown whenever the URL has a valid
  hostname; works even after a `BLOCKED` refusal (trust → retry).
- i18n: `clearnet.trust` added to en / ru / id locales.

## [1.63.7] - 2026-07-21

### Fixed - external links from embedded dApps failed with `shell.open not allowed`
Clicking any external link inside an embedded dApp webview (`embed-*` /
`app-*`) - even one on the trusted `ALLOW_HOST` list - threw
`Uncaught (in promise) shell.open not allowed on window "main", webview
"embed-…"`. Nothing opened.

**Root cause.** Tauri-aware dApps call `window.__TAURI__.shell.open()` (or
`window.open`) directly from inside their webview. The `dapp-bridge`
capability deliberately grants dApp webviews ONLY `core:event:default` -
`shell:allow-open` belongs solely to the trusted host shell (`main`,
`p2p_chat_main`, `syscon`). The ACL therefore rejected the invoke before
the host's ALLOW_HOST / clearnet logic could ever run.

**Fix - ClearNet egress broker in the injected bridge (`dapp.rs`).**
The dApp bridge now intercepts all three egress paths inside the webview:
- `window.open(https://…)` for external hosts,
- `__TAURI__.shell.open` / `__TAURI__.opener.openUrl` (patched to no-op
  forwarders),
- capture-phase clicks on `<a href="https://…">` anchors,

and forwards the URL to the host shell via the permitted event channel
(`dapp-open-url`). `AppLayout.vue` listens for the event and applies the
standard policy: `ALLOW_HOST` hosts open silently in the system browser
via the hardened `open_external` command; everything else raises the
ClearNet warning modal. dApp webviews still cannot invoke `shell.open`
themselves - no capability was widened.

## [1.63.6] - 2026-07-20

### Fixed - `sth://<APP_ID>/<sub/path>` deep-links returned "APP NOT FOUND"
`sth://Sfu4R7…/trading/XBTSX.STH_XBTSX.USDT` (any deep-link into a dApp
sub-route) always resolved to "APP NOT FOUND" - even when the target dApp
was installed locally. Direct root `sth://<APP_ID>` links worked, but
sharing a specific screen was impossible. Users had to open the dApp and
navigate inside manually.

**Root cause.** `AppLayout.vue::openAddress` (URL bar) tested the whole
`raw = "APPID/trading/PAIR"` string against `looksLikeAppId` /
`apps.some(id === raw)` - none of these matched because the appId regex
expects a **pure** base58 authority and installed apps are indexed by
appId only. The Rust `open_app_webview` / `open_embedded_webview` also
hard-coded the target URL to `sth://<addr>/` regardless of any path.

**Fix (both frontend and backend).**
- `AppLayout.vue::openAddress` now splits `raw` at the first `/ ? #`
  into `authority` (appId / domain) and `deepPath` (sub-route + query +
  hash). All appId/domain lookups run against `authority`; `deepPath`
  is passed downstream.
- `AppLayout.vue::onSthOpen` (OS deep-link handler) does the same split
  and forwards the path to `openAppWindow(addr, addr, path)`.
- `Resolve.vue` reads the deep-link path from the `?p=` query and
  forwards it to `tabs.open()` after the network install succeeds - so
  a first-time visit to `sth://<APP_ID>/trading/PAIR` installs the dApp
  AND lands directly on the shared sub-route.
- New `Tab.pendingPath` in `store/tabs.ts` + `tabs.consumePendingPath()`
  (one-shot) so subsequent tab switches don't yank the user back to the
  deep-link.
- `openAppWindow(address, title, path?)` and
  `openEmbeddedWebview(..., path?)` bridge helpers pass the path to Rust.
- `webview.rs`: `open_app_webview` and `open_embedded_webview` build the
  URL as `sth://<addr>/<path>`. When the target webview/window already
  exists, they navigate it via `wv.eval(location.assign(...))` so
  re-opening a shared link doesn't just refocus the old page. New
  `normalize_dapp_path()` helper strips the leading slash and guards
  against protocol-injection (`//attacker.com`, `://` inside path).

Share-by-link UX now round-trips: paste
`sth://<APP_ID>/trading/XBTSX.STH_XBTSX.USDT` into the URL bar (or click
an OS deep-link, or `window.open` from another dApp) - the target dApp
opens straight on the requested screen.

## [1.63.5] - 2026-07-19

### Changed - ALLOW_HOST inverted: trusted hosts open silently, unknown hosts warn
Previously ALL external http/https links routed through the ClearNet warning
modal - even trusted hosts like `github.com` and `accounts.google.com`. The
allow-list was effectively an *identifier* but never *bypassed* the warning,
which made routine OAuth flows and README-link clicks annoying.

Now the logic is inverted: hosts in `ALLOW_HOST` (default:
`github.com, accounts.google.com, passport.yandex.ru, gitlab.com`, extendable
via `frontend/.env`) open **silently** in a new tab via `openNative()`. Only
non-listed hosts trigger the "you are leaving SmartNet, entering clearnet"
warning. Applied consistently across all three interception points:
- `clearnet.ts::installWindowOpenGuard` - programmatic `window.open` from
  dApp iframes / OAuth pop-ups
- `AppLayout.vue::onGlobalLinkClick` - global anchor-click delegate
- `ChatWidget.vue::onMsgBodyClick` - anchors inside chat messages

### Added - Left sidebar: `MY_APPLICATIONS` → `MY DAPPS`
Cleaner and clearer wording in the sidebar; matches how the rest of the UI
already refers to author-built dApps. Applied across `en.ts`, `ru.ts`,
`id.ts` locale files.

### Added - `Copy Magnet` button on release assets
Next to `⇩ Download (torrent)` on each release asset in `DevRepoView.vue`,
a new `⌘ Copy Magnet` button copies the full signed magnet-URI to the
clipboard. Shown only for v1.62+ assets (those that actually have a
magnet). Confirmation is inline (button text swaps to `✓ Copied` for
1.5 s) plus a short toast. Useful for pasting into external BT clients,
chat, monitoring dashboards.

### Changed - System Console: window-based dedup for ping-pong log spam
The DHT emitter alternates two messages (`merged peer DHT hints → N`,
`harvested live DHT nodes from torrent client`). Previous dedup only
compared with the *very last* entry in the ring buffer, so alternating
`A B A B` never coalesced and both messages accumulated forever.

Fix (`syslog.rs`): scan the last **6 entries** in the same scope; if a
match is found, remove it from its old position and re-push at the back
with `count += 1`. Frontend (`SystemConsole.vue::push`) mirrors the same
window logic - locates the matching row within the last 6, splices it
out, appends the fresh copy. Result: what used to be dozens of duplicate
`[dht]` rows collapses to two rows with growing `×N` badges.

## [1.63.2] - 2026-07-18

### Fixed - Release torrent download "did nothing" for 10 minutes (P0)
Downloader clients clicked "⇩ Download (torrent)" on a release asset and:
1. Nothing visibly happened,
2. No entry appeared in `sn://torrents`,
3. Clicking the asset in the dev:// feed produced a `⚠ dev:// not found` toast,
4. Ten minutes later the file quietly finished - no notification.

**Root cause.** The download click path was
`parseMagnetLink(magnet) → startTorrentDownload(infoHash)`. `parseMagnetLink`
uses `librqbit::session.add_torrent(list_only: true)` which **blocks until
metadata arrives from the DHT**. On a cold-start DHT (or when the sole
seeder is behind restrictive NAT) that first-metadata-fetch can take
minutes or fail entirely. The UI stayed on "Adding…" for the whole
duration; the actual download only started AFTER metadata arrived.

The `⚠ dev:// not found` toast on the feed came from a separate bug:
`DevDashboard.vue::downloadReleaseAsset` still called `startFileDownload`
(the old iroh f:// path). Post-v1.62 release assets are seeded via
BitTorrent, not iroh - so `startFileDownload` returned "not found".

**Fix.** New Tauri command `quick_start_magnet(magnet)` and TypeScript
helper `quickAddMagnet(magnet)`:
- Extracts info-hash + display-name from the magnet URI itself via regex
  (`xt=urn:btih:<40hex>` and `&dn=`), NO network call.
- Registers the magnet in `MGR.magnets` / `MGR.names` / `MGR.file_counts`
  immediately.
- Calls `start_download()` which passes the raw magnet to
  `session.add_torrent()` in **download** mode (not list-only) - librqbit
  fetches metadata as part of the download itself, so the torrent
  appears in `MGR.handles` and in the `torrent-progress` snapshot
  **immediately**.
- Returns `{ infoHash, name }` synchronously from the click callback so
  the UI can show a "download in progress" toast within milliseconds.

`DevRepoView.vue::downloadAsset` and `DevDashboard.vue::downloadReleaseAsset`
were both refactored to use `quickAddMagnet` for v1.62+ assets (magnet
present) and fall back to the legacy f:// path only when magnet is
empty. Toast duration was bumped to 4.5–5 seconds and now explicitly
mentions "see Torrents" so downloaders can find the progress bar.

### Added - Publisher WebSeed (BEP-19) mirror URLs for release assets
Zero-peer availability: authors can attach an optional HTTP(S) mirror
URL (GitHub Release, S3 bucket, own CDN) to each release asset. On
publish, `devhub_create_release` appends `&ws=<pct-encoded>` to the
signed magnet URI (BEP-19 WebSeed). Downloaders' librqbit picks up the
webseed automatically and fallbacks to it when the swarm has 0 live
peers - so a solo-developer release stays downloadable 24/7 without
requiring the author's machine to be online.

Trust is anchored in the author key: the mirror URL is part of the
signed IndexRecord (DevAsset), so no man-in-the-middle can inject a
malicious mirror.

- Rust: `DevAsset.mirror: String` field, `sanitize_mirror_url()` accepts
  only `http(s)://<host>...`, everything else silently ignored,
  `pct_encode_url()` for magnet-safe encoding.
- Frontend: optional `mirror` field on `DevAssetInfo`, per-asset mirror
  input in the release creation form (with a 🌐 icon + placeholder), and
  a 🌐 mirror pill link on the published release card.

### Fixed - Own avatar occasionally fell back to Identicon in various widgets
Follow-up to v1.63.1: `set_profile_avatar` now ALSO copies the compressed
PNG into `data/social_cache/<cid>.png` immediately after seeding. That's
the canonical path read by `author_avatar_cached` and `fetch_wall_media`,
so all widgets (UnlockLayer, Wall, Profile header, comment authors, etc.)
find the avatar locally on the very first frame - no more race with
iroh loopback reseeding.

## [1.63.1] - 2026-07-17

### Fixed - PIN-screen avatar showed Identicon for some clients
`author_avatar_cached` used to look only at `data/filesdata/<xx>/<cid>`
(the general file-exchange store). For a user's OWN uploaded avatar the
PNG is written to `data/users/<addr>/avatar.png` (never to filesdata),
so the lookup returned None and the UnlockLayer PIN screen fell back
to the deterministic Identicon. It worked on clients where the iroh
loopback happened to have re-fetched the avatar into
`data/social_cache/<cid>.png` by the time PIN was entered - hence the
"works on some clients, not others" symptom. The lookup now checks:
(1) own `avatar.png` if `author == my_address`, (2) `social_cache`
mirror, (3) `wall/images/<cid>.png`, (4) file-exchange store as legacy
fallback. First match wins - no network.

### Fixed - Installed apps linked to broken author profile
`Installed.vue::openAuthor` used to route with
`identity: authors[owner] || owner` - so if a `@username` was resolved
for the app owner, we routed with the username. `UserProfile.vue`
treats `route.params.identity` as an **address** (feeds it directly
into `followStats(id)`, wall queries, chat DM, rewards…) - passing a
username broke those calls and rendered an empty visitor profile.
Fixed to always pass the wallet address; username stays as the display
label in the badge only.

### Changed - GEEK sidebar cleanup
Removed the `settings` and `myfiles` entries from `NAV_ITEMS` entirely
(previously they were `modes: ['geek']`). Settings remains reachable via
the ⚙ topbar icon in every mode; MyFiles is reachable through
`sn://storages` (which groups Torrents/MyFiles/Music) and direct
`sn://myfiles` deep-links. GEEK sidebar is now less crowded and no
longer duplicates topbar functionality.

## [1.63.0] - 2026-07-16

### Added - Edit / Delete DevHub releases (owner-only)
GitHub-style release management: repository owners can now edit metadata or
delete releases they've published. Two new backend commands:
- `devhub_edit_release(repo, tag, title, notes)` - edits title + notes,
  bumps `seq+1`, re-signs and re-publishes the DevHub index. Tag and
  assets stay intact (changing assets requires delete+recreate to keep
  torrent magnets consistent with the signed manifest).
- `devhub_delete_release(repo, tag)` - removes the release from the
  signed index, removes the staging directory
  `data/release-seeds/<repo>/<tag>/`. Underlying files in
  `data/filesdata/<xx>/<cid>` are preserved (same CID may back other
  releases or be seeded by peers). Torrent sessions keep running until
  explicitly stopped in `sn://torrents`.

UI (`DevRepoView.vue`, Releases tab): each release header now shows
`✎ Edit` and `✕ Delete` pill buttons (visible only to the repo owner
via `isOwn` guard). Clicking Edit swaps notes for an inline
title+notes editor (Save changes / Cancel). Clicking Delete triggers an
inline two-step confirmation (`Delete "tag"? [Yes, delete] [Cancel]`) -
no modal - matching the light-touch pattern used elsewhere in the app.

### Added - Live per-asset progress bars on release cards
Users no longer need to switch to `sn://torrents` to watch downloads:
each release asset with an active torrent now shows an inline progress
bar with percent + download speed, right where the "⇩ Download" button
was. When the transfer completes the bar turns green (seeding). Paused
or errored torrents get textual pills next to the bar.

Implementation notes:
- `DevRepoView.vue` subscribes to the existing `torrent-progress` event
  bus (fired once per second by the librqbit progress emitter) and
  indexes the incoming `TorrentStatus[]` by `infoHash`.
- The mapping asset → infoHash is derived from the signed `magnet` URI
  via a regex on `xt=urn:btih:<40hex>`, so live progress works even for
  torrents the author is currently seeding (no need to click Download
  first) and for previously-started downloads persisted in librqbit's
  fastresume store.
- Subscription is torn down in `onBeforeUnmount` to avoid leaks between
  repo page navigations.

### Changed - English sweep (finishing the localization pass)
- `MyFiles.vue`: `Downloading` → `Downloading`, `Downloading…` →
  `Downloading…`, `No active downloads` → `No active downloads`,
  drop-zone overlay strings → English (`Drop to publish` / `file to
  SmartNet`).
- `bridge.ts` mock data (web-preview fallback): the six mock repository
  descriptions in `searchDevNetwork()`, the mocked repo description
  template in `devMockRepo()`, the mocked README preview in
  `devhubReadFile()`, the mocked user bio in `getUserRepositories()`
  and the error messages of `devhubCreateRelease` / `devhubEditRelease`
  / `devhubDeleteRelease` are all English now.

### Rust API
- New `devhub::devhub_edit_release` and `devhub::devhub_delete_release`
  Tauri commands (`devhub.rs`) registered in `lib.rs::invoke_handler`.

### Frontend API
- `bridge.ts`: new `devhubEditRelease(repo, tag, title, notes)` and
  `devhubDeleteRelease(repo, tag)` helpers alongside the existing
  `devhubCreateRelease`.

## [1.62.1] - 2026-07-15

### Fixed - DevHub release torrents were unusable (P0)
Publishing a release under v1.62.0 successfully generated a torrent, but
three critical defects made downloads fail for every non-author peer:

1. **Filename was the raw blake3 CID** (e.g. `9f5a...deadbeef`) instead
   of the human-readable release asset name (`MyApp-v1.exe`). Files on
   disk in `data/filesdata/<xx>/<cid>` are named after their content hash
   for dedup, and `librqbit::create_torrent` faithfully used that on-disk
   filename. Remote clients therefore downloaded a hash-named blob and
   nobody could tell what it was.
2. **Seeding path did not match the torrent name.** Even if the author's
   client had the file, remote clients requesting the file by its
   original name would not be served - the torrent name and seed name
   didn't match.
3. **Fallback magnet URI carried no trackers.** `magnet_of` returned
   `magnet:?xt=urn:btih:<hash>&dn=<name>` with **zero** `&tr=` entries,
   so remote clients relied purely on Mainline DHT to find the sole
   author-seeder. Cold-start DHT rendezvous can take minutes and often
   fails behind restrictive networks, matching the "download doesn't
   start" reports.

**Fix.**
- `devhub_create_release` (`devhub.rs`) now stages each asset under
  `data/release-seeds/<repo>/<tag>/<original_name>` via a **hardlink**
  (0 bytes overhead, cross-platform) to the file-exchange blob before
  creating the torrent. If hardlinking fails (FAT/network FS) the code
  falls back to a full copy. The torrent is then created from this
  staged path, so both the torrent's `name` field and the on-disk
  filename match the original asset name.
- `create_and_seed` (`torrent.rs`) now builds a full magnet URI with up
  to 8 public trackers appended (`&tr=<pct-encoded>`) and stores it in
  `MGR.magnets` + the persisted `TorrentRecord.magnet`. The `magnet_of`
  fallback (used for legacy records without a stored URI) also emits the
  same tracker list. Remote clients receive announce endpoints in the
  signed `DevAsset.magnet`, so peer discovery via HTTP/UDP trackers
  starts within seconds even on a cold-start DHT.
- `file_counts` is populated by `create_and_seed` (single-file vs
  multi-file), so if the author later re-adds their own torrent from
  the UI it still detects the single-file shape.

### Rust
- `devhub.rs::sanitize_seed_component()` - light sanitizer for
  release-seed path segments (Windows-forbidden chars → `_`, control
  chars stripped, non-empty).
- `torrent.rs::magnet_of()` and `create_and_seed()` - magnet URI now
  includes public trackers.



### Added
- **Release delivery via BitTorrent (full pipeline)**. Previously the button
  "Download" on the release page tried to pull an iroh-blob by CID and failed
  with "not found" for everyone except the original publisher (they had no
  local `FileManifest`). Now releases are distributed through the built-in
  BitTorrent client (librqbit), which is already integrated for torrents.
    - **On publish** (`devhub_create_release`): for each asset, the physical
      file is taken from the file store
      (`files::file_store_path(cid)`), automatically packed into a
      single-file torrent via the existing
      `torrent::create_torrent_from_path`, registered in librqbit for
      distribution and receives a `magnet:?xt=urn:btih:...`. The magnet URI is placed
      directly into `DevAsset { name, size, cid, magnet }` inside the signed
      `IndexRecord` - integrity is protected by the author's signature (secp256k1),
      impossible to forge without the private key.
    - **On the client side**: the "⇩ Download (torrent)" button on the
      Releases tab calls `parse_magnet_link` → `start_torrent_download` and
      opens the download in the existing torrent client. Progress is immediately
      visible in `sn://torrents`. For older releases (v<1.62) without a magnet
      link, the fallback to the previous `startFileDownload` via iroh is preserved.
    - Public BitTorrent swarm = millions of possible peers, WebSeed,
      quality parallel distribution, mature uTP - all the advantages
      that were missing from iroh-blob for static release binaries.

### Rust API
- `DevAsset`: new field `magnet: String` (opt, `#[serde(default)]` for
  backward compatibility with old signed indexes).
- `torrent::magnet_from_info_hash(info_hash) → String` - public
  helper for building magnet URI from the info hash of an active distribution.

### Fixed
- **MyFiles: QR code button not opening** after stage B (Repo→Ver hierarchy):
  in the template remained a reference to `f` (loop variable changed to `row`),
  so `@click="qrFile = f"` tried to assign undefined and QR modal didn't render.
  Replaced with `row.f`.

## [1.61.2] - 2026-07-15

### Added
- **MyFiles - bulk actions (C)**: checkbox column + "select all
  on page" checkbox in the table header + floating toolbar at the bottom ("N selected ·
  📋 Copy links · 🗑 Delete · ✕ Clear"). Copy links batch-copies to clipboard
  all `sn://f/<cid>` one per line. Delete goes through the already
  existing endpoints `deleteFile` (own) / `deleteDownloadedFile` (others),
  so seed separation remains in effect; bulk confirmation via
  modal with explicit ⚠ warning.
- **MyFiles - orphan warning (F)**: every file for which I am
  the only seeder (`seeds ≤ 1`) is marked with a yellow `⚠ orphan` pill
  right next to the name + soft yellow row background. Tooltip explains:
  "content may be lost when you go offline - add it to a release or archive
  a backup". Private (encrypted) files are not counted as orphan - they
  specially limited distribution.
- **Storages - pie-bar disk usage (G)**: above the Torrents/MyFiles/
  Music cards added a horizontal "pie" with 4 segments (Music / Dev / Images /
  Other) and legend chips `Music · 42 · 1.2 GB`. Categories are calculated from
  `parseFileTag(f.tag)` via `listMyFiles()`, the same source of truth as on
  the MyFiles page. Hidden if the user has no files.

### Fixed
- **Pagination 1.61.0 crashed MyFiles on empty screen**: using
  `watch` without import from Vue caused a runtime exception in setup.
  Import restored, also fixed `f.seeders` → `f.seeds`
  (correct field `FileManifest`).

## [1.61.0] - 2026-07-15

### Added
- **File exchange tagging system (Phase 1: A + B from plan)** - every file
  in storage now has an opaque JSON tag `FileManifest.tag`, which
  the frontend parses into `ParsedTag { cat, repo, ver, os, kind }`.
  Categories: `dev` / `music` / `image` / `other` / `private`. All new
  tracks uploaded as part of an album are auto-assigned `{cat:"music"}`;
  dev-hub release binaries receive `{cat:"dev",repo,ver,os}` with
  auto-detected OS by file extension (`.exe/.msi → win`, `.dmg/.pkg → macos`,
  `.deb/.rpm/.AppImage → linux`, `.apk/.aab → android`, `.zip/.gz/.tar → archive`).
  Old files without a tag = "other" (backward compatibility).
- **MyFiles category sub-tabs + search**: under the main "My Files" tab
  appeared a row of 5 pill buttons (All / Dev releases / Music / Images /
  Other) with a counter and total weight for each category ("Music · 42 · 1.2 GB"),
  so the user can immediately see where the gigabytes sit. To the right - a narrow
  search-input by file name / repo / version.
- **Repo → Version hierarchy in Dev tab**: in the table automatically
  inserted group headers `⎇ repo · vX.Y (N · size)`, files
  sorted by repo → version (newest first) → name. The feel of
  "user-ordered" without the mess.
- **DevRepoView Release DropZone**: old list "choose from already
  published files" replaced with a real DropZone. Drag&drop file
  or click → system picker → auto-upload to file store with tag
  `{cat:"dev",repo:"<repo>",ver:"<tag>",os:"…"}` and automatic addition
  to release assets. Price/privacy forced to `0/false`. Uploaded
  files are shown in a separate row with status (`⟳ uploading… → ✓ done`),
  ready ones with a colored OS badge and delete button.

### Changed
- `FileManifest` (Rust `struct` + TS `interface`) gained new field
  `tag: String` (opaque JSON), `#[serde(default)]` - old records in Sled
  are read with an empty tag without migration.
- `add_file(...)` Tauri command received optional parameter `tag:
  Option<String>`; frontend `addFile` - `tag?: string`. Three existing
  calls from `social.rs` (album track uploads) updated with tag
  `{"cat":"music"}`.

## [1.60.0] - 2026-07-14

### Changed
- **Soft-paid albums (revert strict E2EE for new paid uploads)**: field
  experience showed that per-buyer ECIES key delivery over P2P was
  unreliable - if the album owner went offline the buyer would sit for
  days on "album owner is currently offline - key not yet available", never
  actually getting to play tracks they had already paid for. `album_create`
  now publishes new paid albums the same way as free albums:
    - Full tracks are seeded via the file layer as plaintext (no AES-GCM
      wrapping).
    - Optional ~30-second preview (first 768 KB) is still seeded as a
      separate CID so non-buyers can sample tracks.
    - `Album.encrypted` is set to `false` for all newly published albums.
    - `social:albumkey:<id>` is no longer written for new albums.
      Access control is now purely on-chain: buyer sends STH with memo
      `albumbuy:<id>:<buyer>` → `album_access` confirms the payment against
      the seller's transaction history → `canPlay` immediately becomes `true`
      in the UI. This matches the classic paid-download model users expect
      (Bandcamp-style honor system): no DRM, no offline-owner deadlock.

### Preserved
- The strict E2EE codepath (`album_ak` generation, `album_fetch_key`,
  `album_sync_keys`, `Album.encrypted=true`, `crypto::aes_gcm_encrypt`)
  is **kept intact** and used ONLY for reading pre-1.60 albums that were
  published under the old model. The "⚿ Get E2EE Key" button and
  "🔐 Strict E2EE" badge in `AlbumView.vue` are still rendered - but only
  when `Album.encrypted === true` (legacy albums). New albums show a clean
  buy-and-play flow with no key-fetch step.

## [1.59.0] - 2026-07-13

### Added
- **USER | DEV | GEEK - three UI modes** (Apple/Google-style progressive
  disclosure). Three glowing badges above "# WELCOME TO NETFORY" on Dashboard,
  tap switches mode, state persists in localStorage (`sth_ui_mode`,
  `mode` key in `ui` store). Default is **USER**.
    - **USER** (for "regular users"): sidebar shows only Dashboard, Discover,
      Installed, File Exchange (via gateway), **new Storage page**, Network
      Map. `MeshMeter` on Dashboard starts collapsed into a pill in the upper right
      corner of the WELCOME block. Settings/Music icons - in the top bar.
    - **DEV** (for dApp authors): + My Applications, + DevHub.
    - **GEEK** (full tech mode): everything as before - all sidebar items,
      MeshMeter expanded, all hashes/CID/DHT hints in place.
- **`sn://storages` (Storage & Media)** - new umbrella page with three
  large cards (Torrents / My Files / Music) in Dashboard style.
  Replaces three separate sidebar items into one, reducing cognitive load.
  Direct alias: `sn://storages` (or `sn://storage`).
- **Top bar: "♫ Music" and "⚙ Settings" icons**. `Settings` and `Keys` no longer
  take up space in the sidebar (USER/DEV modes) - access is via the top bar
  icon. `Keys` nested as a separate "Keys" tab inside the Settings page
  (`settings.tabKeys`), localized in ru/en/id. Direct route
  `/keys` and `sn://keys` remain working.
- **`ui.meshMeterOpen` + MeshMeter pill toggle** in the upper right corner
  of "# WELCOME TO NETFORY". Click collapses/expands the large
  Global Capacity Meter block. State persists in localStorage
  (`sth_dashboard_mesh_open`), GEEK mode starts with it open by default.

### Changed
- **Discover ("Find Apps") - Apple/Google-style card redesign**.
  The top tech hint "Apps found from peers on the local network
  (mDNS). Installation pulls dApp via iroh-blobs and immediately verifies the signature"
  **was removed** - tech jargon scares regular users.
    - Card now shows only: **logo + name + seeder count +
      description + size + tags**. Everything else (paid/boost/republish/updated/
      sth://-address) moved to the "Details" modal.
    - Actions reduced to 3 buttons: **[Install/Open]** (primary),
      **[Details]** (opens modal), **[📋 Copy]** (square icon button,
      copies sn://-link).
    - **Click on app title** = copy link (quick share mechanic).
    - **"✓ INSTALLED" pill was removed**, replaced by a tiny green dot
      near the title (`data-testid="disc-installed-*"`) and soft green
      card border highlight.
- `nav.config.ts` component gained `modes?: NavMode[]` field for
  filtering sidebar items by active UI mode. Empty array ⇒ item
  visible in all modes (backward compatibility).

## [1.58.1] - 2026-07-12

### Added
- **`sn://` one-word aliases for every system page**: the address bar now
  routes each SmartNet service page from a single token. New aliases:
  `sn://dashboard`, `sn://apps`, `sn://discover`, `sn://installed`,
  `sn://network`, `sn://keys`, `sn://settings`, `sn://nodes`,
  `sn://profile`, `sn://torrents`, `sn://music`, `sn://dev` (+
  `sn://devhub` alias for parity with `dev://`). Existing aliases
  (`sn://nameservice`, `sn://domarket`, `sn://files`, `sn://myfiles`,
  `sn://file/<cid>…`, `sn://album/<id>…`, `sn://group/<secret>…`) unchanged.
  Legacy `u://` aliases (nameservice / domarket / files / myfiles) stay
  supported for old links, but every NEW system-page alias is `sn://` only
  to avoid clashing with `u://<username>` profile links. Consolidated the
  four hand-rolled exact-match branches in `AppLayout.openAddress` into a
  single `SN_ROUTES` lookup table so future additions are a one-liner.

### Fixed
- **`u://<username>` opens a ghost profile when the name is unregistered**:
  typing `u://dashboard` (or any unresolved username) used to fall through
  to `tabs.openProfile(target, …)` with `target` used as an "address",
  yielding a broken UserProfile tab. Now the address bar validates:
  username-shaped input that fails `resolveUsername` shows a clear toast
  "User with name 'X' not found in name registry" and stays put;
  non-username `u://<addr>` input must match the SmartHoldem Base58Check
  address regex (`S[1-9A-HJ-NP-Za-km-z]{25,40}`), otherwise an "Invalid
  u://-address" toast is shown. No more silent navigation to nowhere.

## [1.58.0] - 2026-07-11

### Added
- **⇅ Sync fork** (GitHub-style): fork owners can pull fresh commits from the
  original repository with a single click. New `devhub_sync_fork` command
  resolves the fork's `forked_from`, force-refreshes the source author's
  signed index (bypasses the 60s sync throttle for explicit user action),
  compares HEAD SHA of the default branch. If the fork is already up to date,
  no-op with a toast. Otherwise: swarm-downloads the fresh source bundle by
  its `bundle_hash`, checks out the working copy in the fork's directory,
  re-seeds a personal copy of the bundle (fork keeps distributing independent
  of the source), and updates `refs` / `log` / `tree` / `commits` /
  `bundle_hash` / `updated` in the fork owner's signed index (seq+1 →
  re-sign → publish). `releases` and `description` / `license` of the fork
  stay untouched (matches GitHub: sync only propagates commits + working
  files + refs, never releases). Precise "ahead by N commits" count derived
  from the sliding SHA window of the git log. UI: "⇅ Sync fork" pill button
  in the "forked from …" strip of the repository header, visible only to the
   fork owner. Toast feedback: "Fork is already up to date" / "⇅ Pulled N commits
   from user/repo" / error.

## [1.57.0] - 2026-07-10

### Added
- **Remote repo file tree from signed announce**: every `git push` / repo
  create / fork now embeds a lightweight file-tree manifest (`tree`: path +
  size, capped at 2000 entries, `skip_serializing_if` keeps old signatures
  valid) into the author's signed DevHub index. Peers render the "Code" tab
  of a remote repository instantly from the announce - no bundle download.
- **On-demand P2P file sync**: new `devhub_sync_files` command - swarm-downloads
  the repo bundle from seeders (via the signed `bundle_hash`) and checks out a
  working copy to disk. UI: "⇩ Synchronize (P2P)" banner button on remote
  repos; opening any file auto-syncs first. After sync the Code tab (README,
  file viewer, diffs) is fully local.
- **⑂ Fork** (GitHub-style): new `devhub_fork_repo` command copies the source
  bundle + files to the local account, adds the repo to MY signed index with
  `forked_from: "user/repo"`, re-seeds the bundle independently and publishes.
  UI: Fork button in the repo header (hidden on own repos), "⑂ forked from
  user/repo" link under the title navigates to the source.
- **Unlock screen identity**: the PIN entry screen now shows the owner's
  avatar (disk cache, works while locked) and @username above the form when
  a u:// name is registered.

## [1.56.0] - 2026-07-09

### Added
- **New Music Alerts (NAT-proof)**: a followed artist publishing a new album
  now pops a clickable toast - "🎵 New album · @artist released 'Title'"
  (paid albums get a 💎 badge). Click routes straight to `AlbumView`. Wired
  over the existing DevHub gossip bus via a compact `{k:"albnew", id, o, n, p}`
  announce: fired instantly on `album_create` (both paid and free), plus a
  24-hour periodic re-broadcast so freshly-online followers catch it. Idempotent
  on `social:albseen:<id>` (never spams twice). The old paid-album `k:"alb"`
  keyfile announce was extended with optional `o`/`n` fields - buyers behind
  NAT now also learn owner/title from the same message.
- **Feed Releases with version tags**: DevHub feed rows now surface the
  freshest tagged release for each repo - a golden **v1.2.3** pill next to
  the repo name (click → repository "Releases" tab) and a horizontal strip of
  ⬇ Download buttons for each release asset (streams straight through the
  existing f:// P2P file exchange, one click per asset). Backend
  `devhub_feed` command now returns `latestRelease: DevReleaseInfo?` on every
  item; web fallback shows two mock tags/assets.
- **Toast callbacks**: `useToastsStore.push(...)` now accepts an optional
  `onOpen` handler; `ToastHost` shows "click to open →" and routes
  on click (used by New Music Alerts).

### Housekeeping
- Version bump to **1.56.0** synchronized across `frontend/package.json`,
  `frontend/src-tauri/Cargo.toml` and `frontend/src-tauri/tauri.conf.json`
  (RULES R3 restored - every completed iteration bumps SemVer).

## [1.55.50] - 2026-07-08

### Added
- **Album key gossip (NAT-proof paid-music delivery)**: paid-album keyfiles
  were discoverable only via Mainline DHT + UDP identity-beacon - unreachable
  when the album owner sits behind NAT. Owners now broadcast a compact
  `{k:"alb", id, endpointId, keyfileHash}` announce on the shared system
  gossip topic: instantly on every keyfile publish and every 2 min for all
  owned albums. Buyers cache the owner's EndpointId + keyfile hash
  (`dht:providers:albkeys:` / `social:albgossip:xh:`) and `album_fetch_key`
  falls back to them, downloading over iroh (relays punch NAT). Announce is
  hint-only: the key itself is still ECIES-unwrapped with the buyer's secret.
- **DevHub release feed**: new "Updates Feed" section on the dev://
  dashboard - freshest repo pushes from all synced authors, sorted by update
  time, with a "Following / Whole network" toggle (auto-falls back to the whole
  network when you follow nobody), author avatars, last commit message,
  relative age and a golden ★ on followed authors. Backend `devhub_feed`
  command assembles it from verified signed indexes.

## [1.55.49] - 2026-07-07

### Added
- **Chat "typing…" indicator**: ephemeral E2EE envelope (`sys:"typing"`) on the
  room gossip topic - never stored in history. Receiver emits `chat-typing`;
  the chat header presence line shows a pulsing "typing…" for 3.5 s. Sender
  throttled to one signal per 2.5 s while composing (`chat_typing` command).
- **Avatars in DevHub**: `UserAvatar` (cached avatar system) now renders in
  DevHub dashboard rows (search results + network projects), the repository
  header (`@author/repo`) and the dev author profile (96 px). Backend
  `DevSearchResult`/`DevRepository` now carry the author `address`.
- **Instant profile followers**: `follow_stats` results are cached in Sled;
  new `follow_stats_cached` command lets `UserProfile` render subscriber/
  following lists instantly from the last snapshot while the chain fetch
  refreshes silently in the background.
- **Followed-author push toasts**: after a successful DevHub index sync, if
  the author is in my follow list, the app emits `devhub-author-push` and the
  UI shows a toast "@author updated repo - 'commit message'".

### Fixed
- Missing i18n key `profile.changeAvatar` (ru/en/id).

## [1.55.48] - 2026-07-06

### Fixed - DevHub authors invisible over WAN/NAT (P0, third recurrence - root causes found)
- **`known_endpoint_ids()` returned map KEYS, not EndpointIds**: the `PEERS`
  map is keyed by wallet address (mDNS) or `tracker:<eid>` strings - none of
  them parse as `iroh::EndpointId`, so the gossip overlay bootstrap set was
  silently EMPTY on every client (this also crippled the DHT-hints channel).
  Now returns the peers' actual `node_id` values (deduped).
- **BEP5 announce was a one-shot**: DHT records expire in ~15–30 min and the
  single startup announce could fire before the DHT bootstrapped (or before
  the beacon port existed) and was silently lost forever. The identity-beacon
  now runs a re-announce loop: every 10 min the whole served set (dApps,
  DevHub authors, album keyfiles) is re-announced to the Mainline DHT.
- **UDP identity-beacon is unreachable behind NAT** - added a NAT-proof
  discovery channel that needs no port forwarding:

### Added - DevHub gossip announce bus (works through NAT via iroh relays)
- New system gossip topic `SMARTNET-DEVHUB-ANNOUNCE-BUS-v01` (chat/gossip.rs):
  every client subscribes; authors broadcast a compact signed-index hint
  `{username, address, endpointId, indexHash, seq}` every 2 min and instantly
  on every `publish_index` (create/push/delete/release).
- Receivers (`devhub::handle_gossip_announce`) cache the author's EndpointId
  into the discovery provider cache and the index hash into
  `devhub:gossip:xh:<addr>`, then auto-sync the signed index when `seq` is
  newer. Anti-spoof: if the on-chain name registry knows the username, the
  address MUST match; announces are rate-limited (30 s/address) and
  size-capped. Index authenticity is still enforced by the wallet signature
  check in `store_remote_index` (announce is only a hint).
- `sync_remote` falls back to the gossip-announced index hash when the live
  v2 beacon reply is missing, and resolves usernames through the gossip cache
  when the on-chain registry has not indexed them yet (`devhub:gossipname:`).
- `known_authors()` now also includes gossip-discovered authors, so the
  "Refresh" button polls them too.

## [1.55.47] - 2026-07-05

### Added - real user avatars in the messenger and profile (instant cache)
- New IPC `author_avatar_cached`: INSTANT avatar data-URL from the local disk
  cache (avatar cid from Sled → blob in the file store), zero network.
- New Pinia store `users.ts` - reliable profile-info cache: shows the cached
  avatar immediately, then refreshes in the background (`author_avatar` cid →
  `fetch_wall_media` downloads AND persists the blob to the file store, so
  repeat lookups are offline-fast). In-flight de-duplication per address.
- New `UserAvatar.vue` (address, size): cached image → Identicon fallback.
- Wired everywhere the user asked: messenger (friend list, room list, chat
  header, message groups, online member list) and profile followers/following
  grids + modal.

## [1.55.46] - 2026-07-04

### Fixed - peers on the SAME LAN could not see each other's DevHub repos (P0)
- Root cause: discovery relied solely on Mainline DHT (`get_peers`), which is
  useless/slow on a local network; additionally the beacon's served-set was
  empty after a client restart until the first local change.
- **LAN beacon**: a second signed UDP beacon now listens on a FIXED port
  (42777, +10 fallback for multiple clients per host). `lan_probe` broadcasts
  hello to 255.255.255.255 and 127.0.0.1 and collects all verified replies.
  `discover_extra` runs DHT + LAN probes in parallel and merges the results -
  works for DevHub indexes AND album keyfiles.
- **Startup announce**: `spawn_resync` publishes the own index right at start
  (fills served/extra, seeds index.json); "Refresh from network" also
  announces self first - pressing it on two clients yields mutual visibility.

### Added - repository Settings tab (owner only)
- GitHub-style "⚙ Settings" tab with a Danger zone: delete repository (exact
  name confirmation required), removes it from disk and the signed index.

### Added - commit messages in profile events
- `DevEvent.msg`: push events now store the HEAD commit message (from the real
  git log); shown in the day-events popover of the contribution graph.

### Added - optional free 30-sec previews for paid albums (author's checkbox)
- New "30-sec preview" toggle in the Music Studio (paid albums): the first
  ~768 KB of each original track is seeded as a FREE unencrypted fragment
  (`AlbumTrack.preview`), while full tracks stay E2EE-encrypted.
- AlbumView shows "▶ Listen to preview (30 sec)" for non-buyers.

### Changed - accepting a PR now preserves the FULL git history
- `merge_pr_bundle`: clones the current bundle, applies the PR files, makes a
  real git commit (author = repo owner) and rebuilds the bundle with the whole
  history; repo log/commit count come from the actual `git log`. Falls back to
  the single-snapshot `bundle_from_dir` when git CLI is unavailable.

## [1.55.45] - 2026-07-03

### Fixed - ALL working files vanished from the client after a push (critical)
- Root cause: a push-bundle from `git bundle create` carries **no HEAD**, so
  `git clone <bundle>` exits 0 with an EMPTY working copy; `checkout_bundle`
  then wiped the old files and copied nothing.
- Fix: explicit `git checkout -f <head_sha>` (sha taken from the pushed refs,
  main/master preferred) after cloning + a hard guard: if the checkout yields
  no payload the old files are left untouched.
- Self-heal: opening the Code tab of a repo whose folder contains only service
  files (with a bundle present) re-extracts the working files automatically.

### Added - commit diff viewer (click a commit)
- New `diff_from_bundle` (git show --stat --patch from a bundle clone, 512 KB
  cap) + IPC `devhub_commit_diff`. Clicking a commit row expands a colorized
  diff (+green / −red / @@ blue) right in the Commits tab.

### Added - strict E2EE for paid albums (per-buyer wrapped AES keys)
- Paid album tracks are now encrypted with a per-album AES-256-GCM key; ONLY
  the ciphertext blob is seeded - without the key the content is dead
  (replaces the old soft gate where paid tracks were public blobs).
- Owner issues keys per buyer: `album_sync_keys` scans confirmed on-chain
  `albumbuy:` payments, ECIES-wraps the album key for each buyer's on-chain
  secp256k1 pubkey (the buyer paid ⇒ pubkey exists), publishes the keyfile as
  an iroh blob and announces it in the DHT (`album_infohash`, beacon v2 extra
  hash). Re-announced+re-seeded on every seed_all pass.
- Buyer: `album_fetch_key` discovers the owner in the DHT, downloads the
  keyfile, ECIES-unwraps its own key and stores it locally; playback decrypts
  transparently inside `social_file_bytes` (player untouched).
- UI (AlbumView): "🔐 strict E2EE" badge, "Get E2EE key" retry button when the
  owner hasn't issued the key yet; owner auto-syncs keys on album open, buyer
  auto-fetches after purchase. Legacy (unencrypted) paid albums keep working.
- Infra: generic `announce_extra`/`discover_extra` in the DHT layer
  (dev_discover now reuses them).

## [1.55.44] - 2026-07-02

### Added - commit history tab in the repository UI (GitHub-like)
- New `DevCommit { sha, message, author, date }`; `DevRepoMeta.log` carries the
  latest 50 commits inside the author's **signed index** - any peer sees the
  history without downloading the bundle.
- `checkout_bundle` now also returns `git log` (message/author/date, cap 50)
  from the cloned bundle; `apply_push` stores it in the repo metadata. Repo
  creation / legacy bundle self-heal seed the log with the initial commit;
  accepting a PR prepends the merge commit.
- `DevRepository.log` is exposed via `get_repository`; new "Commits" tab in
  `DevRepoView.vue`: message, author, date, and a click-to-copy sha7 chip.

## [1.55.43] - 2026-07-01

### Added - PR badge on the DEV HUB sidebar item
- New IPC `devhub_pr_open_count` (open incoming PRs for the local user).
- New Pinia store `devhub.ts`; nav badge source `devhub.openPrs` on the `dev`
  item. The layout refreshes the counter on mount, on every `devhub-pr-received`
  event (with a toast "New Pull Request from @author") and after Accept/Reject
  in the PR panel.

### Fixed - repository page showed stale files after `git push`
- `apply_push` stored the bundle/refs and bumped the stats, but never extracted
  the working files, so the Code tab kept showing the initial README/LICENSE.
  New `checkout_bundle` (devhub_git.rs): clones the bundle via the local git CLI
  (guaranteed present - the push came from `git-remote-dev`), wipes old working
  files (keeping `repo.bundle`/`refs.json`) and copies the fresh checkout in.
  Runs in `spawn_blocking`; a git failure only logs a warning (P2P cloning still
  works from the bundle).

### Changed - network discovery diagnostics
- `devhub_discover_network` now logs the number and names (first 10) of polled
  on-chain authors to the System Console, and hints when the name registry has
  not been indexed yet (~90 s after start).

## [1.55.42] - 2026-07-01

### Added - torrent video/audio streaming while downloading (librqbit streaming API)
- The "Sequential download" checkbox is now fully wired: `start_download` spawns a
  background drain reader over librqbit `FileStream`s (file by file, in order), so
  pieces are fetched linearly from start to finish. The drain is stopped on
  pause/remove and resumed on unpause (flag persisted in the torrent record).
- New loopback HTTP stream server (`127.0.0.1:<ephemeral>`, lazy start): endpoint
  `/stream/<info_hash>/<file_idx>` with full `Range` support (206 Partial Content).
  librqbit prioritizes pieces around the current read position, so seeking works
  mid-download. New IPC commands: `torrent_stream_url`, `torrent_list_files`.
- New ▶ button on every torrent row opens a streaming player modal
  (`TorrentStreamModal.vue`): video/audio plays while the torrent is downloading,
  media file picker for multi-file torrents (largest file autoplays).

### Added - decentralized Pull Requests in DevHub (`dev://`)
- New module `devhub_pr.rs`: a PR is a wallet-signed object carrying the full new
  content of changed text files (per-file 256 KB, 20 files, 1 MB total). Outgoing
  PRs are packed into `prs.json`, seeded as an iroh blob whose hash is published in
  the author's signed index (`prsHash`); each PR is independently signed
  (pubkey → address → on-chain name), so a forged `prsHash` yields nothing.
- `sync_remote` hook: when a peer's index arrives, their `prs.json` is fetched,
  every PR is verified and those addressed to the local user are stored in Sled
  (`devhub-pr-received` event, local statuses never overwritten).
- Accepting a PR writes the files, builds a new commit/bundle (`bundle_from_dir`),
  bumps `commits`, publishes the PR id in `mergedPrs` of the signed index - the
  contributor sees the "merged" status after their next sync. Rejection is local.
- New IPC commands: `devhub_create_pr`, `devhub_list_prs`, `devhub_accept_pr`,
  `devhub_reject_pr`. New "Pull Requests" tab in the repository UI
  (`DevPrPanel.vue`): incoming list with Accept/Reject for the owner, a create
  form (edit existing files, add new ones, mark deletions) for contributors.

## [1.55.41] - 2026-07-01

### Fixed - other network users were not visible in the `dev://` DevHub dashboard
- `known_authors()` (devhub.rs) now enumerates **all on-chain registered names** from the
  local `naming_svc:user:` Sled index (capped at 300, excluding self) instead of only
  cached remote indexes + starred authors. The name registry acts as the catalog of
  potential DevHub authors; authors without a published index fail cheaply
  ("no seeders found").
- New IPC command `devhub_discover_network`: immediate network poll that syncs every
  known author (400 ms spacing, 60 s/author throttle inside `sync_remote`). Exposed in
  the dashboard as a "⟳ Refresh from network" button with toast feedback.
- Background resync start delay reduced from 120 s to 25 s after launch.
- Dashboard now live-updates: listens for the `devhub-synced` event and reloads the
  popular/search lists with an 800 ms debounce.

### Added - latest commit SHA and branch in the repository UI
- `DevRepository` (dev_router.rs) now carries the signed git refs (`refs: Vec<DevRef>`),
  populated from the author's signed index.
- Repo header shows `⑂ <branch> · <sha7>` next to the repo hash - click copies the full
  commit SHA to the clipboard (`data-testid="dev-repo-head-sha"`).
- File browser breadcrumbs show `⑂ <branch> @ <sha7>` with the full SHA in the tooltip.

### Fixed - frontend build broke on missing bridge exports
- `devhubDiscoverNetwork()` was imported by `DevDashboard.vue` but never exported from
  `bridge.ts` (rollup error). Added the export plus previously undeclared types:
  `DevRefInfo`, `DevAssetInfo`, `DevReleaseInfo`, `DevRepoMeta`, `DevFileEntry`,
  `DevFileContent`; extended the `DevRepository` interface with `license`, `releases`,
  `refs`. Web-fallback mock now mirrors the backend and includes a deterministic ref.

## [1.55.40] - 2026-07-01

### Fixed - `git clone` of a UI-created repo returned an empty repository

`git clone dev://user/repo` of a repository created through the UI
printed "cloned an empty repository": README.md/LICENSE.md existed only
as working files on disk - there was no git history at all. The client
now generates a **real initial commit in pure Rust** (`devhub_git.rs`,
no dependency on an installed git): blob/tree/commit objects are built
by hand (SHA-1 ids, recursive trees with canonical git entry ordering),
packed into a packfile v2 (varint headers + zlib streams + SHA-1
trailer) and wrapped into a bundle v2 with `refs/heads/main` + `HEAD`.
The format was validated against real git (`git clone <bundle>` →
history, branch `main` and all files check out).

* "Create repository" now immediately produces the bundle, seeds it and
  signs `bundle_hash`/`refs`/`commits: 1` into the index - a fresh repo
  is instantly clonable with its README and LICENSE.
* **Legacy self-healing**: repos created before this fix get their
  initial commit generated on the fly at the first clone attempt (the
  IPC `/refs` and `/bundle` handlers detect an own repo with no bundle,
  build it from the working files, re-sign and republish the index).

## [1.55.39] - 2026-07-01

### Added - ⭐ signed stars: honest decentralized repo rating

Every DevHub index now carries a signed `starred` list
("username/repo" entries). A star is therefore a cryptographic
statement backed by the starrer's wallet key **and** their paid
on-chain username - inflating a rating requires buying names.
`devhub_toggle_star` re-signs the index (`seq+1`) and republishes;
`star_counts` aggregates stars across all locally verified indexes, so
counts grow as more authors are synced. The index signing payload was
extended to cover `starred` (**scheme v2** - indexes signed with the
pre-release v1 scheme are rejected and simply need one re-publish).
UI: a ⭐ Star / Starred toggle with live count on the repository page;
"Projects on the network" and global search now rank by stars, then seeders.

### Added - live global search: periodic re-sync of known authors

A background loop (`spawn_resync`, starts 2 min after launch, runs
every 30 min) re-syncs all known authors - cached remote indexes plus
authors of repos you starred - staggered 3 s apart and additionally
throttled inside `sync_remote` (≥60 s per author). Indexes are
kilobyte-sized blobs, so the network stays quiet while
`search_dev_network` results, ratings, seeder counts and contribution
graphs keep themselves fresh.

## [1.55.38] - 2026-07-01

### Added - DevHub P2P sync: clone foreign repos, live seeders (`sync_remote`)

The `dev://` layer is now fully peer-to-peer - **pull-based and
network-friendly** (no broadcasts: one BEP5 lookup + ≤12 UDP probes per
sync, throttled to once per minute per author):

* **Beacon v2 reply** (`STH\x03`, 140 B) - the UDP identity-beacon can
  attach an extra 32-byte hash to its signed reply. DevHub seeders use
  it to carry the blake3 hash of the author's signed index blob, so a
  reader learns *who seeds* and *what to download* in a single UDP
  round-trip. v1 clients remain compatible (`probe_ex` accepts both).
* **Index publishing** - every signed change (create/delete/push/
  release) seeds `index.json` as an iroh blob, registers its hash in
  the beacon and announces `dev_infohash` to Mainline DHT; the
  `seed_all` cycle re-seeds and re-registers after restart.
* **`sync_remote`** - DHT `get_peers` → beacon probes → swarm-download
  of the index blob → signature + monotonic-`seq` verification →
  Sled cache + `devhub-synced` event (profiles auto-refresh). Live
  seeder counts are stored per author and shown in the UI.
* **Foreign `git clone`** - repo metadata now carries a signed
  `bundle_hash` + `refs`; the IPC `/refs` and `/bundle` endpoints
  transparently sync the index and swarm-download the bundle, so
  `git clone dev://any-author/repo` works across the network.

### Added - clickable contribution graph days (event log)

Clicking a day cell in the green graph opens the day's event list -
which repositories were created / pushed / released / deleted
(`devhub_day_events`, events capped at 50/day). Own events live in
`data/devhub/<address>/events.json`; foreign events are derived by
diffing accepted index updates (`devhub:events:<addr>` in Sled).

### Added - release drafting UI (GitHub-style)

Repository owners get a "Create Release" form on the Releases tab: tag,
title, markdown notes and an asset picker listing the user's published
File-Exchange files (`f://` CIDs). `devhub_create_release` validates
the tag, re-signs the index (`seq+1`) and republishes. Assets are still
download-on-click only.

### Docs

`git-remote-dev/README.md`: build & install guide (PATH naming rules,
per-OS discovery file paths, common errors table).

## [1.55.37] - 2026-07-01

### Added - git-remote-dev: `git clone/push/pull dev://user/repo`

A new ultra-light Rust **Git Remote Helper** (`/git-remote-dev`) bridges
the standard `git` binary to the SmartNet P2P network - zero workflow
changes, native IDE integration (see `docs/09-DevHub-Git-Remote-Helper.md`):

* Git discovers the `git-remote-dev` binary in `PATH` for any `dev://`
  URL and drives it over the `gitremote-helpers(7)` stdin/stdout
  protocol (`capabilities` / `list` / `fetch` / `push`).
* **clone/pull** - the helper downloads the repository's full git
  bundle from the running client and unpacks objects via
  `git bundle unbundle`.
* **push** - the helper packs the pushed refs into a self-contained
  bundle (`git bundle create`), sends it to the client, which stores
  it under `data/devhub/<address>/<repo>/repo.bundle` + `refs.json`,
  updates branch/commit metadata, re-signs the index (`seq+1`,
  anti-rollback), announces to DHT and logs a contribution. Pushing to
  a not-yet-existing repo auto-creates it, GitHub-style.
* **Loopback IPC** (`devhub_ipc.rs`) - the client now runs a minimal
  HTTP server on `127.0.0.1:<ephemeral>` (`GET /refs`, `GET /bundle`,
  `POST /push`, 512 MB cap), guarded by a random `X-Auth` token; the
  port/token discovery file lives at
  `~/.config/smartnet/devhub-ipc.json` (or `%APPDATA%` on Windows).
* Repo file browser hides service artifacts (`repo.bundle`,
  `refs.json`); the repository page got a one-click **`git clone`**
  copy button next to the dev:// link.

### Fixed - address bar treats `dev://` as an sth:// app query

Typing `dev://user/repo` into the top address bar sent the query to the
sth:// resolver (searching `sth://dev://user/repo`). `openAddress()` in
`AppLayout.vue` now recognizes the `dev://` scheme before the sth://
fallback and routes to the DevHub pages: `dev://` → hub,
`dev://user` → author profile, `dev://user/repo` → repository page.

### Added - GitHub-style contribution graph

Developer profiles now render the green 52-week contribution grid
(`devhub_activity` command): every signed `seq` update of the author's
index (repo create/delete, git push) counts as a day-bucketed
contribution. Own activity is stored on disk
(`data/devhub/<address>/activity.json`); foreign authors' activity is
derived from accepted remote index updates (`devhub:activity:<addr>`
in Sled). UTC day-bucketing uses a chrono-free civil-date algorithm.

### Added - DevHub Phase 2, step 1: free signed repositories (`devhub.rs`)

The `dev://` protocol moves from Phase-1 mocks to a real, **completely
free** decentralized GitHub-like layer (no blockchain transaction is
needed to publish - the on-chain username already anchors identity):

* **Real resolver** - `get_user_repositories` / `get_repository` /
  `search_dev_network` now resolve `username → address` through the
  on-chain SmartHoldem naming registry (`naming.rs`) and read
  repositories from **signed DevHub indexes** instead of mock data.
* **Signed index** - `data/devhub/<address>/index.json` holds the
  author's repo list. Signed with the wallet key (legacy bip-schnorr)
  over `devhub:v1:<address>:<seq>:<updated>:<sha256(repos)>`. A new
  `verify_schnorr_legacy` (sth_node.rs) validates foreign indexes:
  pubkey → address match + signature check.
* **Anti-rollback** - a monotonic `seq` counter inside the signature:
  `store_remote_index` rejects any index whose `seq` is not greater
  than the cached one, so a malicious seeder cannot serve an old
  version of an author's index (replay protection without any fee).
* **Scalable storage** - repository files live on disk under
  `data/devhub/<address>/<repo>/` (working tree; git objects arrive
  with the upcoming git-proxy). Sled keeps only light metadata
  (`devhub:remote:<addr>` cache). Release binaries (exe/dmg/apk/deb…)
  are **never** stored in devhub - a release references `f://<cid>`
  File-Exchange assets which download **only on explicit click**,
  GitHub-style.
* **Commands** - `devhub_create_repo` (creates the folder + README.md
    + LICENSE.md from SPDX templates, bumps `seq`, re-signs, announces),
      `devhub_delete_repo`, `devhub_list_files`, `devhub_read_file`
      (path-traversal-safe, 2 MB text cap, binary detection).
* **DHT announce** - a new `dev_infohash = sha1("smartnet:dev:"+address)`
  is announced through the existing UDP identity-beacon + Mainline DHT
  (`announce_dev_user`, merged into the `seed_all` served-set).
* **GitHub-like UI** - new `/dev/new` page (owner/name/description,
  visibility, README toggle, license picker), reworked author profile
  (recent repositories, filter, "New" button, relative-time meta) and
  repository page (file browser with breadcrumbs, rendered README.md,
  Releases tab with per-asset `f://` download buttons, Issues/Wiki
  placeholders). Usernames `new` and `dev` are now reserved.

### Fixed - AI translator download stuck forever after laptop sleep

`fetch_blob_with_progress` (`translator.rs`) awaited `p2p::fetch_file`
without any watchdog. After a suspend/resume cycle the iroh QUIC
connections are dead and `download()` can hang indefinitely, so the
60-second retry loop never regained control - the UI kept showing the
model download banner forever. Each fetch attempt is now raced
(`tokio::select!`) against a stall watchdog that samples
`blob_progress` every 5 s: if the downloaded byte count does not grow
for 120 s, the hung attempt is dropped and a fresh download is started
after 5 s (the blob store keeps the partial data, so the transfer
resumes, not restarts).

### Added - Rust-side network connectivity monitor (`netwatch.rs`)

`navigator.onLine` in WebKitGTK does not fire `offline`/`online` events
after laptop sleep, so the UI never learned the network was gone. A new
lightweight Rust monitor probes well-known TCP anchors
(`1.1.1.1:443`, `8.8.8.8:53`, `9.9.9.9:443`, 3 s timeout) every 15 s
while online and every 5 s while offline, and emits a `net-status`
`{ online }` Tauri event on every state change (plus a System Console
`net` entry). `ConnectionBar.vue` now subscribes to this event in
addition to the browser events, so the bottom status bar reliably shows
"No connection" / "Connection restored" and re-triggers discovery
on reconnect. A `net_online` command exposes the current state.

### Added - System Console hint for socket.io "Invalid namespace"

The dApp bridge polyfill (`DAPP_BRIDGE_JS` in `dapp.rs`) now detects an
incoming socket.io `CONNECT_ERROR` packet whose payload contains
`Invalid namespace` (engine.io + socket.io wire prefix `44…`) on the very
first delivered text frame of a stream. When observed, the bridge prints a
one-shot, red System Console entry containing:

* the exact canonical `io()` call for the current stream, with `nodeId`
  and `provider` substituted from the normalized WS URL, and `path` moved
  into the option bag so socket.io-client no longer treats the provider
  segment as a namespace;
* the alternative server-side fix (`io.of("/<provider>")`);
* a pointer to `docs/08-dApp-Networking-Guide.md § Troubleshooting`.

The hint is emitted at most once per WebSocket instance (`self._nsHintShown`)
so it never spams the console, and it is orthogonal to the normal
`message` event - dApp code still receives the raw `CONNECT_ERROR` frame.
JS was validated with `node --check` after injection.

### Docs - Troubleshooting "Invalid namespace" for socket.io dApps

User verification of v1.55.36 confirmed the P2P WS tunnel is fully working
(HTTP 101, frames both ways). The remaining "Invalid namespace" error is a
dApp-side misconfiguration: passing a URL with a path as the first `io()`
argument makes socket.io-client treat the pathname as a namespace.
Documented the symptom, cause and one-line fix in
`docs/08-dApp-Networking-Guide.md` (new section + rule #5 for dApp authors).

### Fixed - dApp WebSocket tunnel blocked by Tauri ACL (v1.55.36)

The dApp WS polyfill previously called the `dapp_ws_open` / `dapp_ws_send` /
`dapp_ws_close` Tauri commands via `invoke`. Tauri's ACL **always** rejects
application commands for remote-origin webviews (`Command dapp_ws_open not
allowed by ACL`) - the `dapp-bridge` capability only grants
`core:event:default` to `sth://` pages. Result: socket.io dApps (Poker) could
never open a P2P WS connection even after the `rescueWsUrl` fix restored the
correct `ws://<nodeId>/...` URL.

* Bridge JS now routes `wsOpen` / `wsSend` / `wsClose` through the event
  channel (`dapp-request` → `dapp-response`), exactly like the `apiFetch`
  polyfill - no capability/ACL changes required.
* `process_dapp_request` (Rust) gained matching `wsOpen` / `wsSend` /
  `wsClose` handlers that delegate to the existing `p2p::dapp_ws_*`
  implementations and reply asynchronously.
* Incoming frames still arrive via the pre-registered `dapp-ws-message` /
  `dapp-ws-close` listeners with the v1.55.32 buffering, so no race regression.
* The `dapp_ws_*` invoke commands remain registered for the local host shell
  (api:// panel WS tester).

### Docs - Canonical WebSocket / socket.io usage for dApp authors

Extended `docs/08-dApp-Networking-Guide.md` with a new section
"WebSocket and `socket.io` via P2P (`ws://<nodeId>/...`)":

* Canonical form: `io('ws://<64-hex-nodeId>', { path: '/pokersth/api/socket.io' })`
    - first argument becomes the socket.io host and the NodeID is preserved.
* Anti-patterns (`io('ws://api/...')`, `io('api://...')`) explained: socket.io
  parses `api` as host, drops the NodeID, and the URL ends up as
  `ws://api/pokersth/api/socket.io/`.
* `rescueWsUrl()` (v1.55.35) documented as a best-effort safety net that only
  works inside the SmartNet client after a prior `api://` fetch - not a contract.
* FAQ table and debugging section updated to point at System Console
  `[sth://ws] open` vs `[sth://ws] rescue` / `passthrough native WS: ws://api/...`.

## [1.55.35] - 2026-07-01

### Fixed - Socket.IO loses the nodeId when given an `api://` base URL

**Symptom.** Diagnostics from v1.55.34 revealed the dApp's socket.io-client
builds `ws://api/pokersth/api/socket.io/?EIO=4&transport=websocket` - the
host is literally `api`. The socket.io URI parser does not understand the
`api://` scheme and mangles the authority, so the 64-hex NodeID is lost and
the polyfill passed the URL through to the native WebSocket (which goes
nowhere) → "websocket error".

**Fix (`dapp.rs` polyfill).** New `rescueWsUrl()`: when the WS host is not a
64-hex NodeID *and* has no dots/port (`api`, bare hostnames), the nodeId is
recovered from the `__STH_PROVIDER_BY_NODE__` map (populated by `api://`
fetches): the node whose provider matches the first path segment wins;
with a single known node it is used as-is. Provider segment is prepended
only when missing. Real external `wss://host.tld` URLs are untouched.
Rescues are logged as `[sth://ws] rescue <orig> → <rewritten>`.

Unit-tested the rescue logic in Node against the exact URL from user logs;
`node --check` on the extracted bridge JS GREEN.

## [1.55.34] - 2026-07-01

### Fixed - sth:// links from an external browser did not open the app content

**Symptom.** Clicking `sth://SSWrvDJBePS5ULH3GccdabjTN3Eo1Wu6xd` in a regular
browser showed the OS "Open URL:sth?" dialog, the client got focused (or
launched), but the dApp never opened.

**Root causes (two independent bugs).**
1. *Warm start (client already running).* On Windows/Linux a deep link
   arrives as an argv of a second instance. The `single_instance` callback
   only forwarded args starting with `dev://` - `sth://` and `api://` args
   were silently dropped.
2. *Cold start (client not running).* Nobody ever read the deep link from
   the first launch argv. Even if it were emitted during `setup`, the
   frontend `deep-link` listener registers much later (after Vue mount),
   so the event would be lost anyway.

**Fix.**
- `lib.rs`: the single-instance argv filter now forwards `dev://`, `sth://`
  and `api://`. Cold-start argv is scanned in `setup` and buffered in
  `PENDING_DEEP_LINK`; new `take_pending_deep_link` command hands it to the
  frontend exactly once.
- `bridge.ts`: new `takePendingDeepLink()` helper (web fallback → null).
- `App.vue`: deep-link routing extracted into `routeDeepLink()`; after
  registering the `deep-link` listener the frontend drains the cold-start
  buffer through the new command.
- `AppLayout.vue`: `sth-open` / `api-open` CustomEvents are additionally
  buffered on `window` (`__sthPendingOpen` / `__apiPendingOpen`) because
  AppLayout mounts only after the wallet is unlocked - the pending URL is
  consumed in its `onMounted`, so a dApp link clicked before unlock opens
  right after the user unlocks the wallet.

### Fixed - DApp WS polyfill: provider fallback + diagnostics

- `resolveWsUrl` no longer hard-fails when the nodeId→provider map is empty
  (WS opened *before* the first `api://` fetch): if the URL path already has
  segments (`ws://<node>/pokersth/api/…`) it is passed through as-is - the
  Rust side treats the first path segment as the provider name.
- Non-intercepted WS URLs (host is not a 64-hex NodeID) are now logged to
  the System Console as `[sth://ws] passthrough native WS: …` to reveal what
  URL socket.io actually builds inside a dApp.

Validated: `node --check` on the extracted bridge JS, clean
`cargo check --features p2p` (0 errors), `yarn build` GREEN.

## [1.55.33] - 2026-07-12

### Fixed - DApp WebSocket polyfill crashed with "Illegal invocation" (poker "websocket error")

**Symptom.** The WS tester inside the `api://` panel connected fine
(`handshake ok (HTTP 101)`, echo worked), but the same endpoint opened from
inside a dApp (`sth://…`, ui-poker Socket.IO) instantly failed with
"Connection error: websocket error" - the connection never even reached
the Rust bridge.

**Root cause.** The injected WS polyfill did
`StubWs.prototype = NativeWS.prototype`. All native `WebSocket.prototype`
properties (`onopen`, `onmessage`, `readyState`, `binaryType`, `url`, …)
are **native accessors** that throw `TypeError: Illegal invocation` when
invoked with a non-native receiver. So `self.onopen = null` inside the
`StubWs` constructor threw immediately; engine.io caught the constructor
exception and emitted the generic "websocket error". The `api://` panel
tester was unaffected because it calls `dapp_ws_open` via Tauri invoke
directly, bypassing the polyfill entirely - which is why the bug only
reproduced inside dApps.

**Fix (`dapp.rs`).**
- `StubWs.prototype` is now `Object.create(NativeWS.prototype, {...})` -
  an intermediate prototype that keeps `instanceof WebSocket` working but
  shadows every native accessor with plain writable data properties, so
  instance assignments (`self.onopen = null`, `ws.binaryType =
  'arraybuffer'`, `self.readyState = 1`) behave normally.
- Removed a stale `unlistenOpen` reference left over from the v1.55.32
  refactor.
- Added `tlog` diagnostics to the polyfill: URL interception/normalization,
  successful open (`stream=… HTTP 101`) and every error path are now
  streamed to the System Console for observability.

Validated with `node --check` on the extracted bridge JS and a clean
`cargo check --features p2p`.

## [1.55.32] - 2026-07-11

### Fixed - WS handshake race condition (DApp Socket.IO / WebSocket)

**Symptom.** dApp WebSockets (`new WebSocket('api://…')`, socket.io-client)
never fired `onopen` - poker `ui-poker` reported "websocket error" even
though the Rust provider tunnel successfully completed the upstream WS
handshake (visible as `WS[pokersth] upstream OK (HTTP 101)` in
netfory-provider logs).

**Root cause.** Two race conditions in the client-side JS pipeline:

1. Rust `p2p_impl::ws_open` emitted the `dapp-ws-open` Tauri event to
   signal the completed handshake, then returned `stream_id` via the
   invoke Promise. The event traveled to JS on a different channel than
   the IPC response and frequently **arrived BEFORE** the `.then()`
   assignment set the local `streamId` variable. JS listeners rejected
   the event (`if (!localStreamId) return`) → `readyState` was stuck at
   0 → 8-second timeout fired → `close(1006)`.
2. The DApp polyfill in `dapp.rs` registered `dapp-ws-message` listeners
   **inside** `.then(sid => …)`, i.e. AFTER the invoke resolved. This
   opened a window where the reader task in Rust could emit the first
   frame (e.g. Socket.IO `0{"sid":…}` OPEN packet) before any JS listener
   was attached → message frames were silently lost.

**Fix.**
- `p2p::dapp_ws_open` now returns `WsOpenResult { stream_id, status }`
  synchronously in the invoke Promise. No cross-channel race, the
  handshake HTTP status arrives with the same IPC response.
- `bridge.ts` `dappWsOpen()` returns `{ streamId, status }` (backward-
  compat: also accepts a plain string from older builds).
- The `dapp-ws-open` Tauri event is retained for legacy but is no
  longer required - new code ignores it.
- Both the SmartNet WS tester (`ApiView.vue`) and the DApp polyfill
  (`dapp.rs`) now **buffer** all `dapp-ws-message` / `dapp-ws-close`
  events that arrive before `streamId` is known, then drain matching
  events immediately after invoke resolves. Zero frame loss.
- Listeners in the polyfill are registered **before** invoke, not
  inside `.then()`.

### Fixed - Invalid close code 1006 forwarded to upstream WS

**Symptom.** poker WS server logged
`Invalid WebSocket frame: invalid status code 1006` and closed the
connection.

**Root cause.** Per RFC 6455 §7.4.1 the codes 1005 / 1006 / 1015 are
reserved for internal use and MUST NOT appear in a Close frame on the
wire. The client sometimes issued `dappWsClose(sid, 1006, …)` on
timeout, netfory-provider forwarded that verbatim to the upstream WS
server, tungstenite rejected it.

**Fix.**
- Client no longer sends 1006 as a close code (tester uses `1001` = Going Away).
- netfory-provider v0.6.2 sanitises reserved codes in the client→upstream
  path: `1005/1006/1015 → 1000 (Normal Closure)`.

### Bumped

- SmartNet client → `1.55.32` (`package.json`, `tauri.conf.json`, `Cargo.toml`).
- netfory-provider → `0.6.2`.

## [1.55.31] - 2026-07-11

### Added - Built-in WS tester in `api://` panel (8-second handshake timeout)

The `api://` view (`ApiView.vue`, opened via `api-view` route or by clicking
an `api://` link) now has a **mode switch: `api://` / `ws://`**. In WS mode
users can point at any `api://<nodeId>/<provider>/<path>` (the tester
auto-normalizes the scheme to `ws://`) and open a live tunnel over
`netfory/api/1` ALPN - the same code path that the DApp WebSocket polyfill
in `dapp.rs` uses. This makes it possible to diagnose WS/Socket.IO
handshakes (e.g. the `pokersth` provider used by the `ui-poker` DApp)
without launching a full DApp.

Features:
- Live status badge - `waiting` → `connecting…` → `online (HTTP 101)` →
  `closed` / `error`.
- **8-second handshake timeout** - if the provider does not emit
  `dapp-ws-open` within 8 s the tester force-closes the stream with code
  1006 (`handshake timeout`) and shows a red error.
- Frame log with timestamps + direction arrows (`←` in / `→` out / `·`
  system) - text frames are shown inline, binary is summarized as
  `<binary N bytes>`.
- Send-text input + `⇋ Open` / `⏻ Close` buttons.

### Added - Public WS bridge API in `bridge.ts`

New exports `dappWsOpen()`, `dappWsSend()`, `dappWsClose()`,
`onDappWsOpen()`, `onDappWsMessage()`, `onDappWsClose()` - thin wrappers
around the existing Tauri commands (`dapp_ws_open` / `dapp_ws_send` /
`dapp_ws_close`) so the main UI can consume the P2P WS tunnel directly,
not only through the JS polyfill injected into DApp webviews.

### Notes

- Backend and P2P layer unchanged - the tester reuses `p2p::ws_open` /
  `ws_send` / `ws_close` and the `dapp-ws-*` event bus.
- Socket.IO **prefix fix** on the DApp side (documented independently in
  `ui-poker 0.16.3`) - `io('api://NODE')` with `path: '/pokersth/api/socket.io'`
  now preserves the provider prefix in both browser and SmartNet modes.
  The new tester lets us confirm the QUIC upgrade path end-to-end.

## [1.55.30] - 2026-07-11

### Fixed - Socket.IO / WS: automatic provider name restoration

`socket.io-client` (and some libraries with manual `path` in options) builds
WS URL as `ws://<host>/<opts.path>` - takes **only host** from BASE
(our 64-hex NodeID) and path from its settings. The provider name
(`pokersth`), which was in `api://<nodeId>/pokersth`, is completely
lost. Our WS polyfill previously required seeing it in the URL and therefore
rejected the connection as `unknown provider`.

- **`src-tauri/src/dapp.rs` - fetch tap now tracks
  `window.__STH_PROVIDER_BY_NODE__[hexId] = provider`** on each
  successful interception of `api://` fetch. dApp only needs to make at least
  one api-fetch before opening socket.io - and the WS polyfill will automatically
  insert `/pokersth` into the WS URL for the same NodeID.
- **`src-tauri/src/dapp.rs` - `resolveWsUrl(url)`** in `StubWs`:
    - if the path already starts with a known `/<provider>/…` → do not
      duplicate;
    - otherwise, insert the provider as the first segment of the path;
    - fallback: `window.__SMARTNET__.provider` (dApp can set explicitly).
- If the provider could not be determined - StubWs honestly emits
  `error` + `close(1006, 'unknown provider for nodeId')` (does not stay silent).

### Added - Sec-WebSocket-Protocol forwarding

`new WebSocket(url, protocols)` now **transparently** forwards
subprotocols to upstream via the `Sec-WebSocket-Protocol` header in
the mesh-packet handshake. Server-side (netfory-provider 0.6.0) already
forwards headers through its whitelist → upstream receives the subprotocol
as usual and returns the selected one.

- **`src-tauri/src/dapp.rs` - StubWs:**
    - `protocols` string → `Sec-WebSocket-Protocol: <p>`.
    - `protocols` array → `Sec-WebSocket-Protocol: p1, p2, p3`
      (RFC 6455 §4.1).
    - `self.protocol` temporarily initialized to the first of the client protocols (in the future the provider will return the selected one in the `dapp-ws-open` payload - currently the upstream chooses, dApp just sees a familiar one).

### Migration notes

No changes in dApp code are required. Example with socket.io:

```ts
// dApp code - unchanged
const socket = io('api://cf389ca0.../pokersth', {
  path: '/api/socket.io',
  transports: ['websocket', 'polling'],
  auth: { token },
})
// first HTTP request (polling handshake or any api-fetch)
// automatically registers nodeId → provider = 'pokersth'
// subsequent WS upgrade Socket.IO will pick up the provider from the map
```

If your dApp **first** makes a WS without a prior api-fetch,
set the provider explicitly:

```ts
;(window as any).__SMARTNET__ = { provider: 'pokersth', /* ... */ }
```

## [1.55.29] - 2026-07-11

### Added - WebSocket support end-to-end (`ALPN /1`, dApp `new WebSocket(…)`)

Requires **netfory-provider ≥ 0.6.0** on the target side.

- **`src-tauri/src/p2p.rs` - dual ALPN in `ConnectionPool`:**
  `NETFORY_ALPN_V1` (`netfory/api/1`) preferred, `NETFORY_ALPN_V0`
  legacy fallback. `pool_get_or_connect()` returns `(Connection, is_v1)`.
  Same pool key (`EndpointId`) - the pooled entry remembers its ALPN.
- **`src-tauri/src/p2p.rs` - handshake framing:**
  `api_request_impl` writes `[LE_u32 len][JSON]` on `/1`, raw JSON+EOF
  on `/0` (backward compat with `netfory-provider < 0.6.0`).
- **`src-tauri/src/p2p.rs` - new Tauri commands
  `dapp_ws_open` / `dapp_ws_send` / `dapp_ws_close`:**
    - `ws_open(url, headers)` - parses `ws(s)://<hexNodeId>/<provider>/<path>`,
      requires a V1 pooled connection (returns explicit error otherwise -
      strategy (a): fail-fast so the dApp shows a clear "upgrade
      provider" state instead of silently hanging).
    - Frame reader/writer pumps run in their own `tokio::spawn` tasks
      with a bounded `mpsc(128)` between them (backpressure -
      slow-consumer eventually gets `writer closed` `Close 1006`).
    - Emits `dapp-ws-open` / `dapp-ws-message` / `dapp-ws-close` global
      Tauri events, filtered on the JS side by `streamId`.
    - Automatic `Pong` on `Ping` - dApps don't have to write keepalive.
- **`src-tauri/src/dapp.rs` - bridge JS `WebSocket` polyfill:**
  Overrides `window.WebSocket`. If URL matches `/^wss?:\/\/[a-f0-9]{64}\//i`
  (64-hex NodeID host) - routes through Tauri commands. Otherwise
  delegates to the native `WebSocket` untouched (so ordinary clearnet
  WS URLs keep working). Supports `.send(string|ArrayBuffer|TypedArray|Blob)`,
  `.close(code, reason)`, `.readyState`, `on{open,message,close,error}`,
  `.addEventListener` / `.removeEventListener`. `binaryType`
  (`arraybuffer` | `blob`) respected for incoming Binary frames.

### Compatibility with older providers (client fail-fast)

- Provider ≥ 0.6.0 (both `/0` and `/1`) → full WS support.
- Provider 0.5.0 (only `/0`) → `pool_get_or_connect` returns `v1=false`
  after fallback; `ws_open` returns
  `Err("provider does not support WS - upgrade to netfory-provider ≥ 0.6.0")`.
  Polyfill emits `error` + `close(1006, ...)`. dApp handles as usual.
- Provider < 0.5.0 (single-stream) → HTTP-only, no pool benefit,
  WS still returns the explicit upgrade message.

### Rules compliance

- **R1:** everything async - `tokio::sync::Mutex`, timeout on every
  network op, no `.blocking_lock()`, no `std::sync::Mutex` across
  `.await`. Slow-consumer disconnect via bounded mpsc + writer
  break-out.
- **R3:** version bump across `package.json`, `tauri.conf.json`,
  `Cargo.toml`.

## [1.55.28] - 2026-07-11

### Added - ConnectionPool for `api://` (multi-request per QUIC connection)

Paired with **netfory-provider 0.5.0**. Every dApp `fetch('api://...')`
used to run through `endpoint.connect()` → hole-punch → single
bi-stream → close. That meant every request paid the full 300-800ms
hole-punch cost, even for chatty dApps making 10 requests in a row.
Now the client holds a live `iroh::Connection` per provider NodeID and
reuses it for every subsequent request - measured latency drops from
~500ms → ~5-15ms on follow-up requests.

- **`src-tauri/src/p2p.rs` - `CONN_POOL`:**
  `Lazy<tokio::sync::Mutex<HashMap<EndpointId, PooledConn>>>` with:
    - **`pool_get_or_connect(endpoint, node_id)`**: fast-path returns
      a cloned `Connection` if the pool entry is alive
      (`close_reason() == None`); slow-path releases the mutex, does a
      12s-timeout `endpoint.connect(NETFORY_ALPN)`, then inserts. The
      mutex is **never** held during hole-punch - a slow connect for
      one peer cannot block requests to another.
    - **LRU eviction (`POOL_MAX_CONNS = 32`)**: when the pool is full,
      the least-recently-used entry is closed with reason
      `"pool-lru-evict"`.
    - **`pool_drop(node_id)`**: explicit removal when the caller sees an
      `open_bi()` / `read_to_end` failure (peer closed the connection,
      e.g. running provider < 0.5.0). `api_request_impl` retries **once**
      on a fresh pool entry - good UX during rolling upgrades.
- **`src-tauri/src/p2p.rs` - `pool_gc_loop()`:**
  30-second background task started once (idempotent
  `AtomicBool::swap`). Sweeps entries with `close_reason().is_some()`
  and proactively evicts entries idle for > 5 min (300s) - before
  iroh's own idle-timeout, so the pool stays small.
- **`NETFORY_ALPN`** promoted to `pub(super) const` so the pool and
  the request pipeline share the same ALPN literal.
- **`api_request_impl`** - replaced the per-request
  `endpoint.connect(...).close("ok")` dance with
  `pool_get_or_connect` + `open_bi()` on the pooled connection. Kept
  the 14s read timeout. Removed the trailing `conn.close(0u32.into(),
  b"ok")` - that call would have terminated the pool entry after
  every request and killed the whole point of the pool.

### Compatibility with older providers

- Provider ≥ 0.5.0: multi-stream loop → full benefit.
- Provider < 0.5.0 (one-shot): the first pooled request succeeds;
  the second `open_bi()` on the same connection fails because the
  provider already closed it. We detect the failure, drop the dead
  entry from the pool, and retry once against a freshly-established
  connection. Net effect: same latency as before the pool for legacy
  providers - no regression.

### WebSocket support (Step 2) - status: framing landed, tunnel pending

Provider 0.5.0 ships the wire framing (`WsFrame::{Text,Binary,Ping,
Pong,Close}` + LE-length prefix), an `EndpointCfg.local_ws_url` +
`ws_rate_limit_per_peer` config surface, and a fully-implemented
`handle_ws_stream()` with `tokio_tungstenite` upstream + bounded
`mpsc(128)` backpressure + 5s slow-consumer disconnect. It is
**intentionally not wired** into `handle_stream()` dispatch and marked
`#[allow(dead_code)]` because our handshake demarcation is unresolved:
the current stream layout uses `recv.read_to_end()` (waits for EOF),
which is incompatible with a long-lived tunnel that must keep receiving
frames after the handshake JSON. Two clean fixes are on the table for
the next release - length-prefixed handshake on a new
**`netfory/api/1` ALPN** with legacy fallback to `/0`. Client Rust
pump + `WebSocket` polyfill in `dapp.rs` are deferred to the same PR to
keep the wire changes coordinated.

No dApp-visible behavior change in 1.55.28 for WS.

### Rules compliance

- R1: `tokio::sync::Mutex` for pool state, `tokio::time::timeout` on
  every network op, no blocking calls, no `.blocking_lock()`, no
  `std::sync::Mutex` held across `.await`.
- R3: version bump synchronized across `package.json`,
  `tauri.conf.json`, `Cargo.toml`.

## [1.55.27] - 2026-07-11

### Added - IPv6 outbound probe (client & seeder audit)

Audit confirms both `frontend/src-tauri` (client) and `seeder` already
use `Endpoint::builder(presets::N0).…bind()` **without** any
`clear_ip_transports()` call, so iroh's preconfigured dual-stack
(`0.0.0.0:0` + `[::]:0` unspecified) is intact - they were dual-stack
from day one. The 1.55.23 v6-loss was localized to `netfory-provider`
(fixed in provider 0.4.5).

Adds a lightweight diagnostic probe so the operator can see the v6
path status in SystemConsole at startup:

- **`src-tauri/src/p2p.rs` - `probe_ipv6_available()`:**
  Attempts a TCP-connect to three well-known v6 literal endpoints
  (Cloudflare / Google / Quad9 on `[…]:443`) with a **2s hard timeout**
  covering all three in parallel (`tokio::spawn` + `TcpStream::connect`).
  Returns `true` on the first success. Fully async, honors **R1**:
  no blocking, no `.block_on`, no sync DNS (we use raw v6 literals so
  the AAAA-resolver is never involved - it's the *transport* we want
  to test, not name resolution).
- **Result logged once at endpoint start** to `syslog scope=net`:
    - `ipv6 outbound: OK` → clients can bind you directly over v6, no
      CGNAT-hole-punch, no relay overhead.
    - `ipv6 outbound: unavailable` → operating v4-only, which is fine but
      will use relay if the remote peer is v6-only or if CGNAT triggers.

## [1.55.26] - 2026-07-11

### Fixed - netfory-provider companion release (0.4.4) to close the 401 loop

The 1.55.24 client-side header-forwarding patch alone was not enough:
the provider (`/app/netfory-provider`) was dropping the `headers` field
before proxying to the upstream REST API. Landed the matching provider
release so the round-trip actually delivers `Authorization: Bearer …`.

- **`/app/netfory-provider` bumped to 0.4.4** - see
  `netfory-provider/CHANGELOG.md`.
    - `MeshPacket.headers` field added (`#[serde(default)]`, backward-
      compatible with 0.4.3 clients).
    - `proxy_engine::execute()` forwards headers with a hop-by-hop safe
      whitelist (drops `Host`, `Connection`, `Content-Length`,
      `Transfer-Encoding`, `Upgrade`, `Keep-Alive`, `Origin`, `Referer`,
      `User-Agent`); `Content-Type` / `Accept` fallback to
      `application/json` only when the client did not send them.
- Operator note: **restart the netfory-provider node** (`cargo run
  --release`) after pulling 0.4.4 for the fix to take effect. The
  SmartNet client already sends the `headers` field since 1.55.24, so
  no coordinated deploy is needed.

## [1.55.25] - 2026-07-11

### Changed - SystemConsole always opens in its own native window

Consolidating the change from 1.55.23: the toolbar `⌨` button now spawns
`toggle_console_window()` unconditionally (previously it toggled the
in-layout floating panel; a secondary "🗗 TO WINDOW" button on the panel
opened the detached copy). Rationale:

- One code path - one `sys-log` subscriber, no accidental duplicate
  ring buffer subscriptions bloating memory.
- The console is never occluded by a maximised dApp / embedded webview
  in the main window (right-drawer wallet, address bar).
- Consistent with P2P Messenger which is already a detached window.

- **`src/layouts/AppLayout.vue`:** removed floating `<SystemConsole />`
  render, removed `sysConsoleOpen` ref and the `SystemConsole` import.
  Toolbar button now calls `openSysConsoleWindow` → `toggleConsoleWindow`.
- **`src/components/SystemConsole.vue`:** the `🗗 TO WINDOW` button
  disappears because it only rendered for `!props.detached` in the
  floating panel - the floating panel is gone. The component itself
  stays intact so an in-window widget can still be embedded later
  (e.g. dev tab) without a rewrite.

## [1.55.24] - 2026-07-11

### Fixed - `api://` requests dropped Authorization / custom headers (401)

The fetch polyfill and the `api://` uri-scheme handler both forwarded
method + body + path over the mesh packet but **not** the HTTP headers.
This meant a dApp's `fetch(url, { headers: { Authorization: 'Bearer …' } })`
reached the upstream provider header-less → 401 on any authenticated
endpoint (`/auth/me`, `/wallet`, etc.).

- **`src-tauri/src/p2p.rs` - `api_request` / `api_request_impl` now
  accept `headers: BTreeMap<String, String>`:** headers are embedded
  into the mesh packet's `headers` field, with a hop-by-hop whitelist
  (drops `Host`, `Connection`, `Content-Length`, `Transfer-Encoding`,
  `Upgrade`, `Keep-Alive`, `Origin`, `Referer`, `User-Agent`). Providers
  that do not know the field simply ignore it - JSON forward-compatible.
- **`src-tauri/src/lib.rs` - `api://` uri-scheme handler**: reads
  `request.headers()` and forwards them.
- **`src-tauri/src/dapp.rs` - `apiFetch` bridge method**: reads
  `params.headers` (object) and forwards them.
- **`src-tauri/src/dapp.rs` - fetch polyfill (`DAPP_BRIDGE_JS`)**: now
  sends `params.headers = reqHeaders` on every `api://` intercept.
- All callers in `sth_node.rs` updated to `Default::default()` for the
  new arg - internal SmartNet endpoints don't need custom headers.

After this fix, `Authorization: Bearer <jwt>` from `POST /auth/verify`
survives round-trips and `GET /auth/me` returns 200 instead of 401.

## [1.55.23] - 2026-07-11

### Added - Detached System Console window + `api://` fetch polyfill in dApps

**1. System Console → separate native Tauri window.**
The floating console can now be popped out into its own OS-level window
(like the P2P messenger). Mirrors the `?w=chat` pattern already used by
`p2p_chat_main`.

- **`src-tauri/src/lib.rs` - `toggle_console_window`:** New Tauri
  command spawning a `WebviewWindowBuilder` labelled `syscon` at
  1180×460 (min 780×320), URL `index.html?w=syscon`. Subsequent calls
  focus the existing window instead of stacking new ones.
- **`src/App.vue` - `detectConsoleWindow()`:** Same defensive strategy
  as the chat window (query flag + native window label). When true,
  renders **only** `<SystemConsole :detached="true" />` - no auth,
  wallet, updater, discovery - so the detached window doesn't spawn a
  parallel copy of the app and its background timers.
- **`src/components/SystemConsole.vue` - `detached` prop:**
  Fullscreen layout (inset:0, no border/shadow/backdrop), no `✕`
  button, plus a new `🗗 TO WINDOW` button in the floating variant that
  invokes `toggleConsoleWindow()` and closes the floating panel to
  avoid two active `sys-log` subscribers holding duplicate ring buffers.
- **`src-tauri/capabilities/default.json`** - `syscon` label added to
  the window/webview lists so `core:default` permissions apply.
- **`src/lib/bridge.ts`** - `toggleConsoleWindow()` wrapper.

**2. dApp `fetch('api://...')` polyfill (fix for cross-scheme CORS).**
WebkitGTK / WKWebView block cross-scheme fetch (`sth://` → `api://`)
by the same-origin policy **before** our `register_asynchronous_uri_scheme_protocol`
handler ever runs. `Failed to fetch 1ms` was the visible symptom; the
handler `[net] [api] …` line never appeared because the browser
short-circuited.

- **`src-tauri/src/dapp.rs` - `apiFetch` bridge method:**
  New case in `process_dapp_request`. dApp calls arrive via the same
  event pipe as `getAccount` / `signMessage`, which is granted to
  remote origins by `capabilities/dapp.json`. No approval prompt (this
  is a read-only P2P transport). Uses `p2p::api_request`, decodes
  `pdata.body_b64`, returns `{ status, body, contentType, sigVerified,
  provider, signedAt, relayed, target }`.
- **`src-tauri/src/dapp.rs` - fetch polyfill (`DAPP_BRIDGE_JS`):**
  Wraps `window.fetch`. If URL starts with `api://`, routes through
  the bridge (`smartholdem`-style event request) instead of the
  browser's native fetch. Synthesizes a proper `Response` object with
  `Content-Type` sniffed from the raw body and the same
  `X-Sth-Sig-Verified` / `X-Sth-Provider` / `X-Sth-Signed-At` /
  `X-Sth-Relayed` / `X-Sth-Target` headers that the uri-scheme handler
  emits - so dApp code that reads these headers works identically
  whether the scheme registration is honored by the platform or not.
- Every intercepted `api://` fetch is still logged to the SystemConsole
  (`[dapp]` scope) exactly like a normal HTTP fetch - with the correct
  status/timing/URL - so the dev experience is uniform.

## [1.55.21] - 2026-07-11

### Fixed - Right-click "Translate selection" not appearing

The custom "🌐 Translate selection" menu was gated behind
`state.selOn`, which was only set to `true` after the host shell fired
`dapp_translate_config(...)` in response to a `dapp-loaded` event. On
slow first-run of a dApp (or when the host handler was late to attach)
the right-click landed on the WebkitGTK **native** selection UI (the
small floating "…" pill in the screenshot) instead of ours, because our
`contextmenu` capture-phase listener had not been bound yet.

- **`src-tauri/src/dapp.rs` (`DAPP_I18N_JS`) - enable by default:**
  `state.selOn` now starts as `true` with a sane Russian fallback label,
  and `enableSelection()` is called unconditionally on `DOMContentLoaded`.
  The host still owns the language (`target`) and the localized label -
  it just overrides in-place when `config()` lands. No more race with
  `dapp-loaded`.
- **`src-tauri/src/dapp.rs` - `disableSelection()`:** New symmetric
  helper. If the host explicitly passes `selOn: false` to
  `config(target, selOn, label)`, we remove the `contextmenu` listener
  and dismiss any open menu/tooltip - instead of leaving the previously
  installed handler dangling.

## [1.55.20] - 2026-07-11

### Fixed / Hardened - `api://` WebView handler

Reports of `[dapp] ERR GET err:Failed to fetch 0ms api://…` in the System
Console were traced to two independent gaps in the 1.55.18 landing:

1. **OPTIONS preflight was proxied to the P2P provider**, which does not
   implement OPTIONS - some WebKit builds send a preflight on any cross-
   scheme fetch even for "simple" GETs. The provider replied with a
   generic error → the browser blocked the actual request → `Failed to
   fetch` before the real GET was even attempted.
2. `X-Sth-Signed-At` was documented in the 1.55.18 changelog but was
   never actually emitted (the `signed_at` field wasn't threaded through
   `ApiFetchResult`).

- **`src-tauri/src/lib.rs` - short-circuit CORS preflight:** OPTIONS on
  `api://` now returns `204` immediately with `Access-Control-Allow-*`
  headers reflecting `Access-Control-Request-Headers`, `Max-Age: 600`,
  and no P2P round-trip.
- **`src-tauri/src/lib.rs` - visibility:** every `api://` fetch reaching
  the handler is written to syslog as
  `[net] [api] <METHOD> <nodeId>/<provider><path>` so the developer can
  distinguish "handler not reached" (stale binary, scheme not
  registered) from "handler reached, upstream error".
- **`src-tauri/src/p2p.rs` - `ApiFetchResult.signed_at`:** propagated
  through `api_request_impl` so the handler can emit `X-Sth-Signed-At:
  <unix-ts>` as promised.

### ⚠ Operator note

Every change in this file affects the compiled Tauri binary. `yarn dev`
alone will NOT pick up Rust changes - the app must be **rebuilt** with
`yarn tauri dev` (dev) or `yarn tauri build` (release). If you see
`[dapp] ERR … 0ms …` in the console, the current binary was compiled
before 1.55.18 landed and the scheme isn't registered yet. After a fresh
build you will see `[net] [api] GET <nodeId>/<provider>/…` for every
dApp request that reaches the handler.

## [1.55.19] - 2026-07-11

### Added - dApp runtime detection (`window.__SMARTNET__`)

dApps loaded inside SmartNet can now synchronously detect that they run
inside our WebView (as opposed to a plain browser tab) and pick the right
backend (`api://` P2P vs `https://` fallback) without any user-agent
sniffing or feature-probing timeouts.

- **`src-tauri/src/dapp.rs` - `window.__SMARTNET__`:** Frozen object
  installed by the bridge init script (runs *before* any page JS) with
  a stable, minimal schema:
    - `isSmartNet: true` - top-level flag
    - `protocol: "api"` - recommended transport for this environment
    - `appId` - the dApp's own address (host in `sth://<appId>/`)
    - `clientVersion` - real Cargo package version, substituted at
      build time (`env!("CARGO_PKG_VERSION")` in `dapp_bridge_script`)
    - `network: "mainnet"`
    - `features: { wallet, apiScheme, fileScheme, translate, crypto }`
- Installed with `Object.defineProperty` + `Object.freeze` so a dApp
  cannot accidentally overwrite `protocol: 'api'` and downgrade to
  `https`. If a dApp had already set `__SMARTNET__` (e.g. in local
  browser dev), we do not clobber it - makes local-in-browser
  development ergonomic.
- `window.smartholdem.isSmartHoldem === true` remains as the legacy
  probe for wallet-only detection.

Recommended Vite / Vue snippet for dApp authors:

```ts
// src/api/config.ts
const runtime = (window as any).__SMARTNET__
export const BACKEND_URL =
  runtime?.isSmartNet
    ? `api://cf389ca0.../pokersth`         // P2P inside SmartNet
    : (import.meta.env.VITE_BACKEND_URL_HTTPS || 'https://api.example.com')
```

## [1.55.18] - 2026-07-11

### Added - Transparent `api://` fetch inside dApp WebViews

Third-party dApps embedded in SmartNet (e.g. the SmartHoldem Poker frontend
built with `VITE_BACKEND_URL=api://<nodeId>/<provider>`) can now call
`fetch('api://...')` and `XMLHttpRequest` directly against P2P providers,
without any client-side changes and without going through a CORS-bypassing
HTTP proxy. Previously the WebView had no handler for the `api://` scheme
and every request failed instantly with `TypeError: Failed to fetch`.

- **`src-tauri/src/lib.rs` - `register_asynchronous_uri_scheme_protocol("api")`:**
  New URI-scheme handler for the WebView that:
    1. Parses `api://<nodeId>/<provider>/<path>?query` from the WebView
       request (stripping WebView2's synthetic `.api.localhost` suffix on
       Windows/Linux).
    2. Invokes the existing `p2p::api_request` (which speaks iroh/QUIC to
       the netfory provider and verifies the Ed25519 signature).
    3. **Unwraps the envelope:** decodes `pdata.body_b64` and returns the
       *upstream provider's original response body* to the WebView. The
       dApp receives exactly what a plain REST server would have sent
       (`{"nonce":"…", "message":"…", "ttlSec":120}` etc.) - the `pdata`
       wrapper is not leaked into the response body.
    4. **Signature metadata as response headers** (opt-in for dApps that
       want to verify integrity themselves):
        - `X-Sth-Sig-Verified` (`1` / `0`)
        - `X-Sth-Provider` (Ed25519 node id of the signer)
        - `X-Sth-Relayed` (`1` when the response traveled via a relay hop)
        - `X-Sth-Target` (provider name)
    5. Content-Type is sniffed from the first non-whitespace byte
       (`{` / `[` → JSON, `<` → HTML, else `text/plain`) - no assumption
       that upstream is always JSON.
    6. Adds `Access-Control-Allow-Origin: *` +
       `Access-Control-Expose-Headers: *` so a dApp loaded from `sth://`
       can read the response and the signature headers.
    7. Runs on the tokio runtime via `tauri::async_runtime::spawn` - the
       WebView main thread never blocks even under 12s connect / 14s read
       timeouts inherited from `api_request_impl`.
- No new dependencies; reuses `crate::p2p::api_request` and the existing
  `ApiFetchResult` type.

### Notes
- This is **not** an HTTPS/CORS proxy - only the `api://` P2P transport
  is exposed, and every response is cryptographically pinned to the
  provider's NodeID. External clearnet HTTP(S) URLs are untouched.
- The Netfory `api://` address bar in the main window keeps working as
  before (it still uses `invoke('api_fetch', …)` for the pretty-printed
  envelope view with signature diagnostics).

## [1.55.17] - 2026-07-11

### Fixed - Critical: `ipc.localhost` recursion still leaking through JS tap

Even after the previous JS-side filter (`isInternalIpc`), users reported the
SystemConsole flooded with `[dapp] EXT POST 200 http://ipc.localhost/plugin%3Aevent%7Cemit`
lines, freezing the app and pinning the CPU. The prior "Fix 2 (Rust side)"
described in 1.55.16 was **never actually landed** - the listener still
processed every URL. This release lands the true defense-in-depth filter.

- **`src-tauri/src/lib.rs` - hard filter in `dapp-log-event` listener:**
  Any URL whose lowercase form starts with (or contains a path segment for)
  `http://ipc.localhost`, `https://ipc.localhost`, `http://tauri.localhost`,
  `https://tauri.localhost`, `ipc://`, `tauri://` is dropped **before**
  writing to syslog **and** before re-emitting `dapp-log-detail` to the
  frontend. This kills the feedback loop at the Rust level regardless of
  whether the JS-side hook fires.
- **`src-tauri/src/dapp.rs` - removed the "waiting for approval" overlay
  entirely:** `__sthShowSignOverlay`, `__sthHideSignOverlay`,
  `__sthCancelApproval`, `window.__sthOverlayTimer` - all gone. The overlay
  duplicated the WalletDrawer (which auto-opens in the main window), caused
  z-order fights, and - worst of all - a stale overlay could survive after
  auto-approval and block clicks in the main dashboard.
- **`request()` timeout path** in the bridge JS no longer references the
  deleted overlay helper, so the 120s timeout cannot throw a ReferenceError
  and leak a pending promise.

### Memory-leak audit (post-fix)

All bounded/cleanup contracts re-verified after the overlay removal and
Rust filter landing:

| Surface | Bound | Cleanup on unmount |
| --- | --- | --- |
| `syslog::SYS_LOG` (VecDeque) | 500 lines, `pop_front` | n/a (process-scoped) |
| `SystemConsole.vue` `lines` ref | 500 (`splice`) | `unlisten()` |
| `SystemConsole.vue` `details` Map | 500 (`keys().next()`) | `unlistenDetail()` |
| `dapp.rs` `pending` map | per-request `delete` on resolve/reject/120s timeout | inside webview lifetime |
| `dapp.rs` `__sthNetTapped` guard | boolean flag | webview navigation resets |
| `WalletDrawer.vue` `transportTimer` | 5s poll | `clearInterval` in `onBeforeUnmount` |
| `store/wallet.ts` `refreshTimer` / `bridgeTimer` | 30s / 5s poll | `clearInterval` on stop |
| `store/discovery.ts` `pollTimer` | poll | `clearInterval` on stop |
| Tauri `listen_any` listeners | registered once in `setup()` | app lifetime |

No unbounded collections identified. Fix verified by `cargo check --features p2p`.

## [1.55.16] - 2026-07-11

### Fixed - Critical: SystemConsole `dapp` log recursion & memory leak
- **Root cause:** The injected `dapp.rs` network-tap filter (`isInternalIpc`)
  matched Tauri IPC URLs by *prefix* (`http://ipc.localhost/`). When the tap
  itself calls `api.event.emit('dapp-log-event', rec)`, Tauri v2 routes that
  emit through a `fetch()` to `http://ipc.localhost/plugin:event|emit`.
  In some WebView2/WKWebView builds the URL passed to fetch is normalized
  differently (e.g. relative, query-tagged), so the prefix check failed →
  every emit re-triggered the fetch tap → infinite recursion. The console
  flooded with `[dapp] EXT POST 200 …plugin%3Aevent%7Cemit`, the app froze,
  and memory ballooned.
- **Fix 1 (defense in depth, Rust side):** Added a second filter in
  `lib.rs::app.listen_any("dapp-log-event")` that drops any URL containing
  `ipc.localhost` / `tauri.localhost` / `ipc://` / `tauri://` / `asset://` /
  `blob:` / `data:` before writing to syslog **and** before proxying the
  `dapp-log-detail` payload to the frontend. Even a mis-compiled bridge JS
  can no longer poison the console.
- **Fix 2 (bridge JS):** `isInternalIpc` now does substring match (not
  prefix), also catches `/plugin:` and URL-encoded `plugin%3a` variants,
  plus an `__sthEmittingLog` re-entrancy guard so the emit path can never
  recursively trigger itself even if a WebView pumps fetch synchronously.
- **Fix 3 (frontend memory):** `SystemConsole.vue` used to clone the whole
  `Map<id, detail>` on every `dapp-log-detail` event (`new Map(details.value)`).
  Under the recursion storm this created thousands of 500-entry Map clones
  per second → GC pressure → freeze. Switched to `shallowRef` + `triggerRef`
  with in-place `Map.set`/`Map.delete`. Constant memory, O(1) per event.

### Chore
- Bumped `package.json`, `Cargo.toml`, `tauri.conf.json` to `1.55.16`.

## [1.55.15] - 2026-07-11

### UX - Non-blocking "waiting for approval" pill instead of full-page overlay
- `__sthShowSignOverlay()` DAPP_BRIDGE_JS no longer renders a dimming overlay over the entire dApp; instead, it displays a compact floating pill at the top of the page (centered, 24px from the top) featuring an icon and text. The user can still see the dApp content beneath the pill and understands where to switch.
- Removed: dark background `rgba(6,8,12,0.82)`, `backdrop-filter: blur(6px)`, full-screen overlay.
- Added `pointer-events: none` on the container (the pill itself is clickable) - the user can continue interacting with the dApp while waiting for approval in the main window.

### Fixed - Removed forced main-window focus on `getAccount`
- Removed `unminimize + show + set_focus` main window on receiving
  connect-request. This was stealing focus even when the dApp was running in
  a separate OS window, which occasionally broke the z-order (dApp content
  ended up on top of the main window UI - user report from 11.02.2026).
- Notification pill is now the only way to notify the user;
  window z-order remains as set by the OS.


### Fixed - Infinite self-logging spam of `http://ipc.localhost/plugin:event|emit`
- The 1.55.12 dApp network tap wrapped `window.fetch` - but Tauri v2 routes
  its IPC through HTTP fetches to `http://ipc.localhost/plugin:event|emit`
  (WebView2/Linux WebKit) or `https://tauri.localhost/*` (macOS custom
  scheme). Every `api.event.emit('dapp-log-event', ...)` from inside the
  tap itself was therefore a fetch that got logged, which triggered
  another emit → recursive spam that filled the 500-line ring buffer with
  duplicate `EXT POST 200 15ms http://ipc.localhost/plugin%3Aevent%7Cemit`
  lines in under a second.
- **Fix**: added an `isInternalIpc(url)` guard at the entry of the emit
  callback - URLs matching `http[s]://ipc.localhost/`, `http[s]://tauri.localhost/`,
  `tauri://`, `ipc://` are silently skipped (they are Tauri infrastructure,
  never dApp network activity anyway).


### Fixed - ReferenceError `deletingId is not defined` in Installed.vue
- In 1.55.8 I added the `uninstallError` ref alongside `confirmDelete` / `flash`,
  but at the same time lost the declaration of `deletingId` (used in `uninstall()`
  and in the template to show the overlay during deletion). In the prod build the error
  was masked (Tauri release build without source maps), in debug mode
  browser DevTools showed `ReferenceError` on the first click "Delete →
  Confirm". Restored `const deletingId = ref('')`.


### Added - dApp network traffic tap in the System Console
- New `dapp` scope in the floating System Console captures **every** network
  request a dApp makes - pure observation, no interception. The bridge JS
  wraps `window.fetch` and `XMLHttpRequest` in each dApp webview, measures
  duration and forwards `{method, url, status, ms, ok, err}` via the
  `dapp-log-event` Tauri event. WebView still performs the request itself
    - we do NOT proxy, cache, replay, or block anything.
- Rust listens to the event, tags the entry by URL scheme + status and
  appends it to the existing `syslog` ring-buffer (`crate::syslog::log`).
- Colour coding in the console:
    * 🟢 green - our schemes: `sth://`, `api://`, `sn://`, `f://`, `dev://`
    * 🟡 yellow - public `http://` / `https://`
    * 🔴 red - 4xx / 5xx or transport failure (any scheme)
    * ⚪ grey - `data:` / `blob:` / relative (internal)
- New **`EXPORT TXT`** button in the System Console header - dumps the
  currently-visible (i.e. filtered) log lines to a timestamped `.txt` file
  via the standard browser download flow. Useful for pasting logs into
  GitHub issues.
- Ring-buffer capped raised from 300 → **500** lines (per user's request).

### Added - docs/08-dApp-Networking-Guide.md
- Short guide for dApp authors on how to set up backend correctly:
  CORS + Let's Encrypt HTTPS + Private Network Access preflight. Copy-paste
  FastAPI/Express templates + Caddyfile. Explains WHY the client refuses to
  proxy their requests (attack surface) and points at netfory-provider
  gateway as the recommended long-term pattern.


### Reverted - Native HTTP proxy for dApps (1.55.10) fully removed
- Following a security review the entire native `fetch` proxy - added in
  1.55.10 to work around WebView CORS / Private Network Access when talking
  to LAN backends - has been rolled back. Bypassing browser security to
  reach `192.168.x.x` created too broad an attack surface: allow-list or
  not, a compromised or malicious dApp would gain data-exfiltration,
  LAN-reconnaissance, and attack-relay primitives out of the box. Even
  per-dApp allow-lists cannot make this safe as a default - the correct
  posture is *no* network privilege by default.
- Removed:
    * `dapp.rs` - the `"fetch"` bridge method, `perform_dapp_fetch()`,
      `DappAllowConfig`, `set_dapp_allowlist`, `get_dapp_allowlist`,
      `parse_host_port`, `is_dapp_fetch_allowed`.
    * `DAPP_BRIDGE_JS` - `smartholdem.fetch()` and the transparent
      `window.fetch` override.
    * `bridge.ts` - `DappAllowConfig` type, `readDappAllowConfig`,
      `saveDappAllowConfig`, `applyDappAllowConfig`.
    * `App.vue` - `applyDappAllowConfig()` boot call.
    * `Settings.vue` - the `settings-dapp-allowlist` UI section.
- **Recommended pattern for dApp authors**: expose your backend behind a
  public HTTPS URL with a valid certificate (Let's Encrypt) and correct
  CORS headers (`Access-Control-Allow-Origin: *` or `sth://...`). Or run a
  netfory-provider gateway on your infrastructure - either way, plain
  browser `fetch()` will work with zero client-side changes and full
  browser security intact.


### Fixed - dApp `fetch()` to LAN/private-network backends
- Reported symptom: authorization via wallet succeeded on the 2nd client, but
  the subsequent dApp call to `http://192.168.x.x:8002/api/health` failed
  with "Failed to fetch". Root cause was NOT CORS on the backend (browser
  fetch worked fine) - the WebView2/WKWebView stack behind the `sth://`
  custom scheme enforces Chrome's **Private Network Access** and stricter
  cross-origin rules for custom protocols, silently blocking outbound HTTP
  to RFC 1918 address ranges (192.168.x.x, 10.x.x.x, 127.x.x.x).
- **Fix**: added a native HTTP proxy inside the SmartHoldem bridge.
    * **Rust** (`dapp::process_dapp_request`): new `"fetch"` method routes the
      request through `ureq` on the OS TCP stack - no CORS, no PNA. Returns
      a Web Fetch-API-shaped envelope `{ ok, status, statusText, headers,
    body, bodyBase64, url }` with 8 MiB body cap and 15 s default timeout.
    * **DAPP_BRIDGE_JS**: new `window.smartholdem.fetch(url, opts)` returns a
      Response-like object with `.ok / .status / .headers.get / .text() /
    .json() / .arrayBuffer() / .blob() / .clone()`.
    * **Transparent fetch override**: `window.fetch` is patched - absolute
      `http://` and `https://` URLs are routed through the Rust proxy, while
      same-origin / `sth://` / `f://` / `data:` / `blob:` requests keep using
      the native browser fetch. **Existing dApps work without any code
      changes** - the LAN backend just starts responding.


### Fixed - Context-menu "Translate selection" now runs on CPU too
- Prior behaviour: `dapp_translate_batch` set `cache_only = true` for every
  CPU-only build (any dApp translation was skipped and the original text was
  returned). Perfectly reasonable for **auto-translate the whole page** -
  CPU inference of 200 UI strings would freeze the app for minutes. But it
  also silently disabled the **right-click → Translate selection** context
  menu, which was the biggest UX regression for non-GPU users.
- Fix: `DappTrReq` now carries an optional `manual: bool` flag. Set to
  `true` by the injected DAPP_BRIDGE_JS in `translateSelection()`, it tells
  Rust to force the inference even on CPU builds - a single 1–2 sentence
  selection takes 2–5 s on CPU, which is acceptable when the user
  explicitly asked for it.
- Auto page translation on CPU is still cache-only (unchanged) - it relies
  on the P2P `tr:<lang>` topic to receive translations from GPU peers.

### Speed - Uninstall bulk deletes coalesced (from 1.55.8 request)
- `crate::p2p::schedule_reclaim_after_delete()` - new fire-and-forget
  debounced (500 ms) wrapper around `reclaim_after_delete`. When the user
  bulk-uninstalls several dApps in a row, `seed_all()` no longer runs after
  every single click; only ONE reclaim pass fires at the end of the burst.
  UI returns to the caller immediately.


### Fixed - Uninstall app silently failing on "Installed"
- The Installed page "Delete → Confirm" click flow was resetting the
  confirm state (`confirmDelete = ''`) but the app never actually vanished
  from the list - the button just re-rendered as "Delete". Root cause
  was two-fold:
    1. **Rust**: `delete_app_blocking` unwrapped a poisoned `SESSION_SEED`
       mutex, which re-panics inside `spawn_blocking` → `JoinError` → the
       Tauri command rejects the frontend Promise → the store's `.filter()`
       never runs, leaving the app in the list. Now uses `.lock().ok()` and
       falls through gracefully; local purge always runs.
    2. **Rust**: `broadcast_delete` was called synchronously with 5 s HTTP
       timeout per tracker (up to 15 s for the default 3 trackers). If any
       tracker was slow the whole delete_app_blocking stalled. Now the
       tombstone broadcast runs in a fire-and-forget `std::thread::spawn`
       so the local purge (what the user actually cares about) returns
       immediately.
    3. **Frontend** (`Installed.vue::uninstall`): the `try` block had no
       `catch`, so any deleteApp failure was swallowed silently. Added
       error capture with a red banner surfacing the failure message plus
       `console.error` for developer diagnostics.
- Added `installed.removeFailed` i18n string (ru / en / id).


### Fixed - `getAccount: extension did not respond in 45s` on 2nd client
- Real-world dApps (poker, marketplace and others written against the
  Core-Wallet **browser extension** API) apply a 30–45 second internal
  timeout on `getAccount()`. When the SmartNet main window was in the
  background - a common state on machines where the user just launched the
  dApp from the "Applications" catalog - the connect-permission prompt
  appeared invisibly, the user never saw it, dApp's timeout fired, and the
  page rendered its own `getAccount: extension did not respond in 45s`
  error. That is the exact scenario reported in the field.
- **Rust fix** (`dapp::process_dapp_request::getAccount`): before emitting
  `incoming-connect-request`, the shell now `unminimize + show + set_focus`
  on the main window, matching MetaMask/Phantom behaviour on first
  handshake. Guarantees the prompt is visible even if the dApp is running
  in a floating webview above the main window.
- **In-dApp overlay**: the DAPP_BRIDGE_JS overlay (introduced in 1.55.5 for
  signMessage) now also paints on `incoming-connect-request` events, with
  the copy "Waiting for connection approval". Overlay is auto-removed on
  `dapp-response` and on the bridge's 120s timeout - so the dApp never
  looks frozen. Overlay is delayed 500 ms so it doesn't flash for
  auto-approved trusted origins.


### Changed - Auto-sign for trusted dApps (Core-Wallet UX parity)
- After the user grants a dApp `getAccount` access (once, via the "Connect"
  prompt) subsequent `signMessage` requests from the same origin are now
  auto-approved WITHOUT re-prompting - just like the Core-Wallet browser
  extension. Rationale: the connect handshake already established trust, and
  poker/gaming/chat dApps that do per-request heartbeat signatures were
  making the wallet unusable with a prompt for every signature.
- Explicit `SIGN` prompt still shows for **unknown origins** on first
  contact, giving the user full control over the initial trust decision.
- Removing trust is one click: Wallet → Apps → DISCONNECT. Next sign
  request from that origin will surface the AuthorizeMessage prompt again.

### Fixed
- Rust no longer auto-focuses the main window on every `signMessage` - that
  behaviour moved to the frontend and now runs ONLY when we actually need to
  surface a prompt (untrusted origins). Prevents the shell from stealing
  focus during rapid auto-signing sessions.
- In-dApp AuthorizeMessage overlay now waits 500ms before painting itself -
  if the request auto-approves fast (trusted origin case) the overlay never
  even flashes.
- i18n `wallet.apps.desc` (ru / en / id) updated to reflect the auto-sign
  behaviour and communicate that DISCONNECT restores per-signature prompts.


### Fixed - "Failed to fetch" on dApp signMessage (race condition)
- After the 1.55.3 change dApps no longer received a signature synchronously -
  they now wait for the user to press "SIGN" in the main window. Real-world
  dApp windows however float above the main SmartNet window, so the user did
  not see the AuthorizeMessage prompt and dApps timed out into their own
  `fetch()` fallback path → the reported `Failed to fetch`.
- **Backend fix**: `dapp::process_dapp_request::signMessage` now brings the
  main window back to the foreground (`unminimize + show + set_focus`) before
  emitting `incoming-sign-message-request`. The prompt is always visible.
- **In-dApp overlay**: the injected `DAPP_BRIDGE_JS` now listens to
  `incoming-sign-message-request` and paints a full-screen semi-transparent
  overlay inside the dApp webview - "Waiting for signature · open the main
  window". The overlay is auto-removed on `dapp-response` or on request
  timeout, so the dApp never appears frozen.
- Added console.info diagnostics on the frontend when a sign request lands.


### Added - Wallet drawer bottom toolbar (Core-Wallet parity)
- The right-side wallet drawer now has a sticky bottom tab bar exposing three
  quick actions that used to be buried in the main app Settings screen -
  matching the layout of the SmartHoldem Core-Wallet browser extension.
- **Connected Apps** (`ConnectedAppsView.vue`): lists every dApp origin the
  user granted `getAccount` access to, with an inline two-step Disconnect
  action. Uses the existing `approvedOrigins` state.
- **Nodes** (`NodesView.vue`): quick node picker with two tabs (Clear Nodes /
  P2P Nodes), live ping per row, active-node highlight, AUTO (best) and
  RESCAN actions. Reuses `refreshNodePool` / `autoSelectNode` / `selectNode`
  from `lib/bridge.ts`. Web-preview mock so the UI is testable without Tauri.
- **Wallet Settings** (`WalletSettingsView.vue`): "Reveal private key"
  danger-zone flow - 2-step arm checkbox, `nativeRevealSeed()` fetch,
  numbered 12-word grid, 30-second auto-hide countdown, copy-to-clipboard.
- `WalletDrawer.vue`: sticky footer with three tabs (`apps` / `nodes` /
  `settings`), auto-hidden on approval-gated views (verify / connect /
  authorize) so nothing distracts the user from a pending prompt.
- i18n: added `wallet.view.{apps,nodes,settings}`, `wallet.bottom.*`,
  `wallet.apps.*`, `wallet.nodes.*`, `wallet.settings.*` in ru / en / id.


### Added - Approval-gated `signMessage` (Core-Wallet AuthorizeMessage flow)
- The dApp `window.smartholdem.signMessage(text)` bridge no longer auto-signs.
  A new `AuthorizeMessageView.vue` slides in on the right-drawer wallet, shows
  the exact plaintext payload, the dApp origin + trust badge, and the signer
  address. Users must press "SIGN" for the signature to be produced.
- Rust:
    * `dapp::process_dapp_request::signMessage` now emits
      `incoming-sign-message-request` instead of signing immediately.
    * New Tauri commands `resolve_sign_message` / `reject_sign_message` - the
      former invokes `sth_node::sign_message` (bip-schnorr legacy) and resolves
      the dApp Promise with `{ address, publicKey, hash, message, signature }`.
- Frontend:
    * `store/wallet.ts` - new `pendingSignMessage` state, listener, and
      `approveSignMessage()` / `rejectSignMessage()` actions.
    * New `WalletView = 'authorize'` route in `WalletDrawer.vue`.
    * `wallet/ipc.ts` - mock command handlers and
      `window.__sthDevIncomingSignMessage()` dev helper for the web preview.
    * i18n: added `wallet.authorize.*` and `wallet.view.authorize` to ru / en / id.

### Refactored - Left-sidebar nav pulled into typed config
- Added `src/config/nav.config.ts` with a typed `NavItem` schema
  (`{ id, icon, i18nKey, staticLabel, badgeSource, gate }`).
- `AppLayout.vue` now imports `NAV_ITEMS` and resolves badges / gates via
  lookup tables - adding a nav item is a single-line diff instead of touching
  an 1800-line layout file.

### Added - P5 Seeder payout history (MyApplications)
- New "PAYOUT HISTORY" button on the MyApplications header opens
  `PayoutHistoryModal.vue` - a modal showing every on-chain `payout:<appId>`
  credit from the network payout-server, grouped by dApp with per-app filter.
- Rust: new `p2p::payout_history` command scans `fetch_history_cached()` for
  incoming transfers whose memo starts with `payout:`, aggregates total / app
  count, joins app names from the local Sled catalog.
- Frontend: `lib/bridge.ts::payoutHistory()` with web-preview mock (three
  sample payouts) so the flow is demonstrable without Tauri.
- i18n: `payoutHistory.*` block added to ru / en / id.


### Fixed - DEV HUB sidebar icon
- The `DEV HUB` item in the left sidebar had an empty icon, leaving a blank slot when the
  sidebar was collapsed (icon-only mode). Added the `⎇` (branch) glyph, matching the
  unicode-glyph style used by the other nav items and hinting at git/code hosting.

## [1.55.1] - 2026-07-11

### Added - Copy dev:// link on repository page
- The repository dashboard now has a "Copy dev:// link" button (with clipboard fallback)
  that copies `dev://<username>/<repo>` and shows a toast confirmation - makes sharing decentralized
  projects in chat / on the wall one click.

## [1.55.0] - 2026-07-11

### Added - dev:// protocol & developer web interface (Phase 1)
- New decentralized developer protocol `dev://` with end-to-end routing:
    - `dev://<username>` → developer profile hub (repo grid).
    - `dev://<username>/<repo>[/<subsystem>]` → repository dashboard with Code/Issues/Wiki tabs.
    - `sn://dev` hub mapped to the in-app `/dev` dashboard (search + my repos + popular projects).
- Backend `dev_router.rs`: robust URL parser with typed `DevRouterError` and a `Target` enum
  (`UserProfile` / `Repository`), plus mock Tauri commands `parse_dev_url`, `get_user_repositories`,
  `get_repository`, `search_dev_network` returning Phase-2-ready structs (repos/user/search results
  with hash/seeders fields). Includes unit tests for the parser.
- OS-level deep linking via `tauri-plugin-deep-link` (scheme `dev` in tauri.conf.json + capability),
  wired in `lib.rs`: `on_open_url` and single-instance argv both emit a `deep-link` event that the
  frontend intercepts and routes.
- Frontend: Vue Router routes (`/dev`, `/dev/:username`, `/dev/:username/:repo`), a `deep-link`
  event listener in `App.vue` + `devLink.ts` router helper, and three components - `DevDashboard.vue`,
  `DevUserProfile.vue`, `DevRepoView.vue` - plus a DEV:// sidebar entry. Bridge wrappers include web
  fallbacks mirroring the Rust mock so the UI works in browser preview.
- Phase 2 hooks documented in code: replace mocks with `resolve_username()` (Ed25519 registry),
  `dht.get_peers()` seeder discovery, and `iroh.download()` for git objects.

## [1.54.0] - 2026-07-08

### Changed - Unified seeding ranks by uploaded volume (GB)
- Node ranks are now driven by **total uploaded GB** instead of Byte-Hours, with 5 tiers:
  **LEECH** <10 GB · **PEER** ≥10 GB · **SEEDER** ≥500 GB · **SUPERSEED** ≥1000 GB (+100 rep) ·
  **ANCHOR** ≥5000 GB (+500 rep). Backend `profile_stats` rank/`nextRankAt`/progress recomputed on
  uploaded bytes; the progress label now reads in GB. SUPERSEED added to rank metadata (frontend
  Network Map + User Profile).
- The "MY NODE" seeding-awards block was made ~3× more compact: a single-line rank-tier strip
  (LEECH→PEER→SEEDER→SUPERSEED→ANCHOR, reached tiers highlighted) plus two slim single-row award
  progress bars, replacing the tall two-card layout. Restores visibility of the earlier rank
  achievements (Seeder, etc.).

## [1.53.0] - 2026-07-08

### Added - Seeding awards credited to real reputation
- The SUPERSEED (1,000 GB) and ANCHOR (5,000 GB) seeding awards now grant their reputation bonus
  (+100 / +500) to the **real reputation ledger** stored in the user DB (Sled), so it shows up in
  the profile's reputation badge and everywhere `get_reputation` is read.
- Backend: `reputation::grant_once(addr, tag, bonus)` - idempotent one-time credit guarded by a
  Sled flag (`reputation:once:<addr>:<tag>`). The profile-economy tick checks the seeded-bytes
  thresholds every minute and grants each award exactly once.

### Fixed - Slow profile avatar load
- The own-profile avatar is stored locally but was loaded **last** in `onMounted`, behind the whole
  social/wallet/network chain. It now loads **first and non-blocking**, so it appears instantly.

## [1.52.0] - 2026-07-08

### Added - Network Map: torrent seeding awards
- New "Seeding awards" block in the personal node economy card (Network Map). Grants reputation
  bonuses based on total seeded volume:
    - **SUPERSEED** - 1,000 GB seeded → +100 reputation
    - **ANCHOR** - 5,000 GB seeded → +500 reputation
- Each award shows earned/locked state, a progress bar toward the threshold, the current
  seeded-GB total, and the reputation bonus. Total earned reputation is summarized in the header.
  i18n RU/EN.

## [1.51.7] - 2026-07-08

### Fixed - Installed apps: wrong earnings widget + delete indicator
- Fixed a bug where an installed app card with zero personal accrual rendered the **aggregate
  network-earnings list** ("Estimated network accrual") instead of nothing. The per-app vs aggregate
  branch in `SeederEarnings.vue` is now strictly gated by `appId` (per-app widget only shows when
  `estSth > 0`; the aggregate never leaks into a card).
- Delete in the Installed view now shows the **same deletion overlay** ("Deleting…" with a
  spinner) as the MY_APPLICATIONS cards while the app is being removed.

## [1.51.6] - 2026-07-08

### Changed - Sidebar NETFORY wordmark
- Reverted the sidebar header to the left-aligned layout (logo pinned left, collapse arrow right).
- Added the **NET**FORY wordmark to the right of the logo ("NET" white, "FORY" cyan #00F0FF),
  matching the marketing-site header. Hidden when the sidebar is collapsed.

## [1.51.5] - 2026-07-08

### Changed - Sidebar logo alignment
- Centered the sidebar header row (logo + collapse arrow) so the NETFORY logo sits closer to the
  center of the block instead of being pinned to the far-left edge.

## [1.51.4] - 2026-07-08

### Added - NETFORY logo in the left sidebar
- Ported the marketing-site NETFORY logo (rotating cyan/lime squares around a pulsing cyan dot)
  into the sidebar header, placed to the left of the collapse arrow. On hover the two squares
  counter-rotate around the central pulsing dot (outer 45°→135°, inner 45°→-45°). Colors match the
  brand palette (primary #00F0FF, secondary #B4FF39). Hidden when the sidebar is collapsed.

## [1.51.3] - 2026-07-08

### Changed - Sidebar collapse button
- Replaced the "‹ COLLAPSE" text button with a larger, noticeable icon-only chevron button
  (rounded, accent-tinted, hover fill + glow). The chevron rotates 180° to indicate expand/collapse
  state; centered when the sidebar is collapsed.

## [1.51.2] - 2026-07-08

### Fixed - Music page layout / album card buttons wrapping
- Widened the Music page central content (max-width 1100 → 1440px) and album grid cell
  (minmax 340 → 420px) so cards have more room.
- Album card action buttons no longer wrap onto two lines: added `white-space: nowrap` to buttons
  and `flex-wrap: wrap` to the action row so labels stay single-line and overflow to a new row.

## [1.51.1] - 2026-07-08

### Added - Discover live results counter
- Sticky search bar now shows a **live "Found: N" counter** (right-aligned) reflecting the number of
  dApps matching the current query + tag filter, updated as you type. i18n RU/EN/ID.

## [1.51.0] - 2026-07-08

### Changed - Home banner + Discover page UX
- Home banner headline renamed **"WELCOME TO SMARTHOLDEM" → "WELCOME TO NETFORY"** (i18n RU/EN/ID).
- Discover page ("Find apps"): the search bar is now **sticky** (Google-style, always visible while
  scrolling the results); tag filter bar is **capped at 40 tags** with a "+N" overflow indicator to
  keep the bar compact when thousands of dApps expose many tags.

### Changed - Seeder earnings estimate: forecast + clarity (per user feedback)
- Aggregate "Estimated network accrual" widget (Earn tab) now shows a **forecast** for the
  currently-running (not-yet-closed) day at current seeding weights, shown **only when the actual
  accrual is still 0.0000** (i.e. the window's first day hasn't closed). Forecast is styled amber
  with a "forecast" tag so it is never confused with a settled amount.
- Added an explanatory note + **live countdown** ("First estimate in Xh Ym Zs") that explains the
  bank is split over 5 days and each day's share only becomes final after that day closes - fixes
  the confusing "everything shows ≈ 0.0000 STH" question for freshly-funded dApps.
- Per-dApp compact widget ("Your accrual for this dApp") in the Installed grid is now **hidden
  entirely when the accrual is ≈ 0.0000 STH** (no forecast/empty state shown there).
- Backend (`p2p.rs`): added `forecast_current_day()` and new `DappEarning` fields
  `forecastSth` / `nextPayoutSecs`; mirrors the payout model's per-day, 12h-offline rules.

### Changed - Dashboard trackerless indicator: info popover instead of Settings redirect
- The Dashboard trackerless-DHT pill no longer navigates to Settings. It now reveals an **info
  popover on hover** (pinnable by click) showing live diagnostics: seeders found via DHT, dApps
  with DHT providers, dApps announced to DHT, nodes in the Mainline DHT table, and the UDP
  identity-beacon port. i18n RU/EN.

### Added - P1: Trackerless DHT diagnostics panel in Settings
- New diagnostics card under the DHT-providers toggle (Network tab) showing the full
  `dht_providers_summary` metrics (seeders / provider dApps / announced dApps / DHT routing nodes /
  beacon UDP port) with a manual refresh button. Off-state hint when the toggle is disabled.
- Extended the `DhtProvidersSummary` bridge type with `announcedApps`, `beaconPort`, `dhtNodes`;
  implemented the previously-missing `p2p_impl::beacon_diag()` (fixes a compile break).



### Added - Dashboard trackerless indicator (clickable → P2P settings)
- Slim status pill on the Dashboard, placed between the eyebrow line and the WELCOME title:
  shows **"TRACKERLESS MODE ACTIVE · N seeders via DHT"** (green, pulsing dot) when DHT-discovered
  seeders exist, or "TRACKERLESS MODE ON · discovering via DHT" when enabled but none found yet;
  hidden when the toggle is off. **Clickable** → routes to Settings `?section=p2p`, which
  smooth-scrolls to and flashes the DHT-providers toggle row. Gives users in censored networks a
  visible signal + one-click path to the privacy settings. i18n RU/EN.
- New command `dht_providers_summary` counts unique DHT-discovered EndpointIds across the
  `dht:providers:*` Sled cache; polled on the Dashboard's 12 s refresh loop.
- Note: BEP5 `implied_port` intentionally NOT used - our identity-beacon runs on a SEPARATE UDP port
  from the DHT socket, so we must announce the explicit beacon port (implied_port would announce the
  wrong one).


- **Seeder side (`p2p.rs` `seed_all`)**: for every seeded dApp, announces `app_infohash` on the
  BitTorrent Mainline DHT via librqbit (`get_peers(id, Some(beacon_port))` drives BEP5
  `announce_peer`), and updates the identity-beacon's served set.
- **Beacon lifecycle**: `ensure()` starts the identity-beacon once (`data/node.key` signer → same
  EndpointId as the iroh endpoint), port stored in `DAPP_BEACON_PORT`.
- **Client side (`gather_providers` in `install()`)**: when enabled, runs `get_peers(app_infohash)`
  → parallel beacon `probe` of discovered IP:ports → verified iroh EndpointIds merged into the swarm
  provider list. Results cached in Sled `dht:providers:<app_id>` (instant restart + DHT-flakiness
  resilience; fallback to cache when DHT is empty).
- **librqbit DHT wrappers (`torrent.rs`)**: `dht_announce_dapp` / `dht_discover_dapp` over the live
  librqbit session DHT (driven on librqbit's runtime, results returned via oneshot).
- **Toggle** `cfg:dht_providers_enabled` (default ON) + commands `get/set_dht_providers_enabled`;
  Settings → P2P shows `dht-providers-toggle` (i18n RU/EN). Fully additive to trackers/mDNS with
  automatic fallback - no regression when the DHT is unreachable.
- Verified: `cargo check --features p2p` GREEN (no errors/new warnings); `yarn build` GREEN.


- New module `src-tauri/src/dht_dapp.rs` (behind `p2p`) - the IP:port→EndpointId bridge for
  trackerless dApp seeder discovery over Mainline DHT. Separate dedicated `tokio` UDP socket
  (does NOT touch iroh's QUIC socket, which quinn owns exclusively).
- Protocol: `HELLO` (24 B: `STH\x01` + 20-byte app_infohash) → `BEACON_REPLY` (108 B: `STH\x02`
    + 32-byte iroh EndpointId + 8-byte BE timestamp + 64-byte ed25519 signature). The seeder signs
      `[app_infohash | ts | endpoint_id]` with its iroh secret key; the client verifies with the
      EndpointId from the packet → DHT poisoning / fake providers rejected with 100% certainty.
- Hardening: replay/freshness window (±300 s), per-IP rate limit (anti-DDoS/amplification),
  beacon stays SILENT for apps it doesn't seed (no identity leak). `app_infohash = sha1("smartnet:dapp:"+id)`.
- Added `sha1 = "0.10"` (RustCrypto) for BEP5-compatible infohash.
- Verified: `cargo check --features p2p` GREEN; protocol logic + crypto validated by 9/9 isolated
  unit tests (roundtrip, UDP end-to-end resolve, tampered id/sig/app rejected, replay rejected,
  wrong-magic rejected, silent-for-unseeded) - run without the heavy iroh build.
- Next: Phase 1 - wire librqbit `announce_peer`/`get_peers` under app_infohash + integrate probe
  into `p2p::gather_providers`, behind a `cfg:dht_providers_enabled` toggle.


### Added - hover tooltip on Network Map globe nodes
- `NodePlanet.vue` now raycasts the cursor against node markers and shows an overlay tooltip; text is
  supplied by the parent via a new `tooltips` prop. `NetworkMap.vue` feeds per-node text: label +
  personal bandit speed (% of your fastest) + ✓successes/✗errors, or "no data". See a node's metric
  right on the point without checking the table. Frontend-only (`yarn build` GREEN).


- `NetworkMap.vue` globe gains a colour-mode toggle (`globe-color-mode`): **BY SOURCE** (existing
  transport/source colours) or **BY MY SPEED** - nodes coloured by the local swarm-bandit speed
  (bright green = fast, cyan = medium, muted blue = slow, orange = had errors, grey = no personal
  data), with a matching legend. Visually highlights the best routes for YOU. i18n RU/EN.

### Decision - HashSeq per-file collection migration CANCELLED
- The per-file iroh-collection (HashSeq) migration (former backlog "option b") is dropped as too
  risky (touches deterministic hashing, `verify_app`, dedup, desktop+headless seeders, tracker) for
  marginal gain - iroh already chunks a single blob and the built-in torrent handles chunked
  transfers. The blob-level bandit-failover + torrent cover the real needs.


- `NetworkMap.vue` leaderboard now shows a **"MY SPEED"** column next to Byte-Hours (`score`):
  per-node speed from the local swarm bandit (`swarm_top_seeders`, matched by `endpointId`) as a
  compact bar (green→ok, orange/red when that peer had errors) + ✓successes, or `-` for nodes you
  never downloaded from. Ties the global seeder leaderboard to YOUR real measured throughput.
- i18n RU/EN `network.mySpeed` / `network.mySpeedHint`. Frontend-only (`yarn build` GREEN).


- **`swarm_top_seeders` command** (`p2p.rs`) returns the bandit's peers ranked by speed
  (`SwarmSeeder{peer,speed,errors,attempts,successes}`); empty without the `p2p` feature.
- **Settings → P2P mini-widget** (`swarm-top-seeders`): ranked list with a speed bar (green→ok,
  orange/red when the peer has errors) + ✓successes / ✗errors, shown when swarm is ON, with a
  refresh button. i18n RU/EN `peers.swarmTop*`.
- **Non-spam System Console log**: after a weight-changing download `save_swarm_weights()` logs the
  top-3 seeders to `syslog` (scope `swarm`) AT MOST once per 60 s and only once real stats exist -
  `🏆 top-seeders: 1. ab…cd 1234B/s · …`.
- Verified: `cargo check --features p2p` GREEN (0 swarm warnings); `yarn build` GREEN.


- **Runtime toggle** `cfg:swarm_enabled` (default ON) + Tauri commands `get_swarm_enabled` /
  `set_swarm_enabled`; Settings → P2P shows a `swarm-toggle` switch (i18n RU/EN `peers.swarm*`).
  When OFF, `install()`/`prefetch()` run the exact previous single-peer path (no behavior change).
- **`install()` + `prefetch()` (`p2p.rs`)**: when swarm is ON they first try `download_blob_bandit`
    - an ε-greedy (ε=0.15) whole-blob failover across ALL known providers (announced peer + headless/
      tracker seeders from the Network Map). It picks the historically fastest/most-reliable peer, gates
      on `BlobStatus::Complete` within an 8 s per-try timeout, rewards/penalizes the peer, and instantly
      switches on a stall (6–16 tries). On ANY failure it safely falls back to the original download path,
      so installs never break. Kills the previous ~10–30 s stalls on a dead/slow announced peer.
- **Persistent bandit** `SWARM_BANDIT` (process-global `Arc<SmartSwarmBandit>`) loads its weights from
  Sled `swarm:weights` on first use and saves after every install/prefetch → the client remembers good
  seeders across sessions and tries them first. Decisions/rewards logged via `syslog` (scope `swarm`).
- Rationale: current wire format is one tar-blob per dApp (no per-file HashSeq), so this is a
  reliability + best-peer failover layer at the blob level (torrent handles chunked transfers
  separately). True per-file simultaneous multi-peer download remains the future HashSeq migration.
- Verified: `cargo check --features p2p` GREEN (no new warnings); `yarn build` GREEN. The bandit
  algorithm itself is covered by the 4 swarm unit tests (isolated crate). Real download behavior is
  Tauri-runtime - user validates on the local `yarn tauri:build`.


### Added - async fault-tolerant Swarm download engine (Contour 2: client bandit)
- **New module `src-tauri/src/swarm/`** (gated behind the `p2p` feature) implementing per-file,
  multi-peer PARALLEL dApp download - the client previously fetched a dApp blob from a SINGLE
  announced peer with a sequential headless-seeder fallback (no real swarm).
    - `mod.rs` - public API: `BlockSource` trait (network abstraction, boxed `Send + 'static`
      futures), `SwarmError` enum (NoPeers / BlocksFailed / Runtime - no stringly-typed errors),
      `SwarmConfig` (concurrency limit, per-block timeout, ε, max retries).
    - `bandit.rs` - `SmartSwarmBandit`, an ε-greedy multi-armed bandit. Lock-free hot path:
      `std::sync::RwLock` guards the peer map (write only on peer registration), each peer's weight
      is atomics (`AtomicU64` speed-as-bits + `AtomicU32` error/attempt/success counters). Speed is
      an EWMA (α=0.2) so a single lag never zeroes a good seeder but sustained degradation quickly
      demotes it. `select_peer` picks max-speed / zero-error with prob (1-ε), random peer with prob ε
      (channel-capacity exploration). Sled persistence via `to_json`/`from_json`.
    - `engine.rs` - `download_dapp_swarm(...)`: `tokio::task::JoinSet` scheduler with a concurrency
      limit and a pending/retry queue. Each block is wrapped in `tokio::time::timeout`; on timeout the
      block is instantly re-queued for another bandit-selected peer and the stalling peer is penalized.
    - `source_iroh.rs` - real `IrohBlockSource` over iroh-blobs 0.103 (download-to-`Complete` gate +
      export bytes). Ready to wire into `install()` next; NOT yet called from install (staged).
    - `tests.rs` - simulation unit tests. Key test `swarm_reroutes_around_degrading_peer`: 50 blocks,
      3 peers, one peer hard-degrades (stalls past the timeout) mid-run. Result: 50/50 blocks fetched,
      the swarm rebuilds the graph onto live nodes (measured: healthy_a=47, healthy_b=3, slow=0).
- **Logging** via the existing `syslog` bus (scope `"swarm"`): bandit EXPLORE/EXPLOIT decisions,
  per-peer rewards/penalties (speed change), progress + fastest-peer readout. Streams to the in-app
  System Console.
- Verified: `cargo check --features p2p` GREEN (zero warnings from the new module); the 4 swarm
  unit tests pass 4/4 in an isolated crate (validated without the heavy iroh full build/link, which
  is disk-constrained in this container).
- NOTE (honest scope): raw throughput only improves with ≥2 healthy seeders on larger dApps and on
  relay-bound transfers; the guaranteed win is reliability - no more multi-second stalls on a dead/
  slow peer. Contour 1 (tracker genetic algorithm) intentionally NOT implemented in this iteration.


### Fixed - profile playlist: first-click playback + scrollable list
- **First click on a track played nothing (UserProfile → Playlist)**: the global `<audio>`
  element lived inside `v-if="player.hasTrack || player.queueOpen"`, but the `loadTick`/`playing`
  watchers fire pre-flush - BEFORE the element mounts on the very first play - so `loadCurrent()`
  bailed early (`audioEl` was still `null`) and nothing loaded. The bottom bar showed the pause
  icon (playing=true) at 0:00/0:00 while no audio ever started; the 2nd click worked only because
  the element was mounted by then. Fix (`BottomAudioPlayer.vue`): the root `.bap` container now
  always mounts (idle → height 0 via `.bap-idle`) so `<audio>` exists from first paint.
- **Autoplay unlock**: added a one-time `pointerdown`/`keydown` listener that creates + `resume()`s
  the `AudioContext` on the first real user gesture, so the suspended-context autoplay policy no
  longer silences the first track (play() previously ran after an `await`, losing user activation).
- **Playlist height + internal scroll (`WallPlaylist.vue`)**: `.pl-list` now caps at `max-height: 460px`
  with `overflow-y: auto` (thin neon scrollbar, `overscroll-behavior: contain`) so long playlists
  scroll internally instead of stretching the profile column.


## [1.50.2] - 2026-07-08

### Fixed - one large/failing app blocked seeding & version announce for ALL apps
- Diagnosis (live trackers): the publisher (@catsatoshi) advertised 6 of 7 apps but NOT the large
  XBTS DEX (5111 files) - so it stayed at v1 while the owner's client showed v7. Root cause: in
  `seed_all` the per-app tar build (`??`) and iroh import (`?`) propagated errors, so a single failing
  or oversized app **aborted the whole `seed_all`** - the tracker snapshot never rebuilt and the new
  version was never announced (the 6 announced apps were leftovers from an older successful run).
- Fix: `seed_all` is now resilient - a failure in tar build, path resolution, iroh import, or a
  not-`Complete` blob status for ONE app is logged (`eprintln`) and **skipped**, so the remaining apps
  still (re)seed and announce. If the oversized app now imports, it is announced; if it genuinely
  can't (disk/size), the log line pinpoints which app and why instead of silently blocking everything.

## [1.50.1] - 2026-07-08

### Fixed - UI froze ~10s when the earnings widget loaded
- The new `network_earnings` command was a **synchronous** Tauri command doing blocking `ureq` HTTP
  calls to every tracker (`/apps` + `/app/{id}/seeders` per active app, 6s timeout each) - on the
  main thread, freezing the window (title showed "Not responding"). Made the command `async` and moved
  all blocking tracker I/O into `spawn_blocking`. Also added an in-flight guard in the `earnings` store
  so multiple widgets mounting at once trigger a single tracker sweep, not N concurrent ones.

## [1.50.0] - 2026-07-08
Release marker rolling up the recent work: Phase G per-app 5-day payout model
(tracker 1.2.0 + payments-server 1.1.0 + seeder 1.3.0 auto-delete), the per-dApp
earnings widget, the dashboard redesign, DApp translation Phase 3 / fixes, and the
publish/update re-announce fix. No functional change in this entry - version bump only.

### Fixed - published app updates (new versions) never reached trackers/seeders (client 1.49.1)
- Symptom: publishing v5 (paid) of a dApp left every tracker showing v1 with a stale name/hash; the
  new version never propagated while the app kept running.
- Root cause: `finalize()` / `update()` / `editMetadata()` / `updateFromPath()` updated the local
  manifest but did **not** re-run `start_discovery` (seed_all + tracker snapshot rebuild). The tracker
  heartbeat kept re-announcing the OLD snapshot (previous version/hash), so no provider ever advertised
  the new version - and seeders, which upgrade only when they see a higher version in the directory,
  never learned about it. It self-healed only on the next app restart (startup runs discovery).
- Fix: all four publish/update/edit store actions now call `startDiscovery()` on success, so the new
  blob is immediately re-seeded and the new version/hash/name is re-announced to trackers + DHT within
  the next heartbeat - no restart required.

### Added - Phase G follow-ups: headless auto-delete, per-dApp earnings widget (client 1.49.0, seeder 1.3.0)

**Headless seeder (`seeder/src/main.rs`, v1.3.0) - cargo check GREEN**
- **Auto-delete of expired-paid apps under disk pressure.** When the allocated disk budget is
  >90% full (`SEEDER_EVICT_FILL_RATIO`), the headless seeder now frees space by dropping apps whose
  PAID seeding window has ended (tracker reports `paid && secondsRemaining==0`), lowest-priority /
  largest first, until back under the threshold. Apps still in an active paid window (earning) and
  apps whose status no trackers currently confirm are never touched. Home/client nodes never run this
  binary, so a user's chosen apps are never auto-removed.

**Client (`frontend`, v1.49.0) - cargo check --features p2p + yarn build GREEN**
- **`network_earnings` Tauri command** reproduces the payout server's per-app 5-day math from the
  trackers (`/apps` + `/app/{id}/seeders`): each funding bank / 5 per day, split by online-seconds,
  12h-offline eviction. Returns per-app + total estimated accrual for the wallet's reward address.
- **"Your accrual for this dApp" mini-widget** on each Installed app card (`SeederEarnings.vue`):
  estimated STH, bank, your uptime, live seeder count, seeding time left - a motivational nudge to
  keep seeding. Backed by a throttled `earnings` store (1 tracker poll/min, cached).
- **Earn tab** now shows a real **"Estimated network accrual"** panel (total + per-app breakdown)
  from the new per-app model, alongside the existing K_up forecast. en/ru strings added.

> The `network_earnings` estimate is display-only; the authoritative payout is settled by the
> reward server's owed ledger. Runtime UI verification is on the desktop build (native-only feature).

### Added - Phase G: per-app, 5-day seeder payout model (tracker 1.2.0 + payments-server 1.1.0)
Replaces the old "split the whole escrow bank by gbSeeded×K_up every 7 days" with a
fair, per-application distribution driven by on-chain funding events.

**Tracker (`tracker/src/main.rs`, v1.2.0)**
- **Funding events per app.** Every escrow payment `sth:<id>` (publish/paid update /
  reseeding) and `sth_boost:<id>` (boost) becomes its own `FundingEvent {ts, amount, txid}`
    - an independent BANK distributed over 5 days from when it landed. Deduped by txid,
      retained 30d. Paid updates naturally start a fresh 5-day reseeding window; overlapping
      windows simply sum (each still capped at its own bank).
- **Per-app-per-seeder stats.** From signed heartbeats (`active_app_ids`) the tracker keeps
  per (app_id, reward_address) daily online-seconds buckets (`seedstat:<id>:<addr>`, ~8 days).
  Uptime is signed by the address key → can't be forged. New endpoint `GET /app/{id}/seeders`.
- **`/apps` extended** with `paidTs`, `bankSth`, `dailyPoolSth`, `secondsRemaining`, and active
  `funding[]` (id, ts, amountSth, amountSmarthoshi, endsAt) for the payout server.
- **Status UI** now shows per-app **PAID** badge, **Bank** (STH to distribute) and **Seed left**
  (countdown of the paid seeding window).

**Payments-server (Node, v1.1.0)**
- **New accrual engine (`accrual.js`).** For each funding event, `bank/5` per day is split among
  that app's ELIGIBLE seeders **proportionally to online-seconds served that day** (more seeders →
  less each). A seeder is dropped from a day (and later days) once cumulative offline since it
  STARTED seeding that app exceeds 12h - so mid-period joiners are paid from their own start.
- **Ledger + batched payout.** `accrueOnce()` (hourly) settles each completed (event, day) exactly
  once (`settled:<txid>:<day>`) into an `owed:<address>` ledger; `payoutBatch()` broadcasts the
  ledger as ONE multi-payment when the period elapses (fewer tx fees), guarded so payout+fees never
  exceed the live escrow balance. New endpoints `/accrue/run`, `/reports/owed`; `/preview` now shows
  accrual preview + pending owed.
- Verified: pure accrual unit test + full mock-tracker integration (no overpay, idempotent,
  proportional split, mid-period join). Tracker `cargo check` GREEN.

> Follow-up (next): headless-seeder auto-delete of apps whose paid period ended when disk >90%
> full (client/home nodes keep everything - user-controlled). Frontend "Earn STH" estimate.

## [1.48.0] - 2026-07-01

### Changed - Dashboard redesign: header search + tidier layout
- **App search moved to the banner.** A rounded search bar now sits directly under the welcome
  heading. Submitting (Enter or the → button) navigates to **// Find Apps** (Discover) with the query
  pre-applied via a `?q=` route param - Discover reads it on mount and reacts to further changes, so
  the search runs immediately. No tags / results are shown on the dashboard itself (kept minimal).
- **NameService and Stealth-mode blocks relocated** from the stat row into the banner header (a
  responsive 2-column row under the heading), where they fit naturally.
- **Removed the "YOUR IDENTITY" card** (the active wallet address is already shown top-right) and the
  **"APPS SEEDING" card** - the whole stat-card row is gone.
- **DApp count badges in the left sidebar.** Menu items now carry small pill badges: MY_APPLICATIONS
  (my published apps), Installed (apps installed from other devs) and Find Apps (pending updates
  available). Badges hide at 0 and cap at `99+`; in the collapsed sidebar they float on the icon.
  Frontend-only; `yarn build` GREEN, web-preview screenshot verified.

## [1.47.2] - 2026-07-01

### Fixed - DApp translation ran forever on animated pages (tickers / live maps)
- On pages with constantly-mutating DOM (price tickers, the XBTS animated world-map with live
  numbers) the translator never settled - the badge climbed to `823s` and it kept re-translating.
  Root cause: the `MutationObserver` fired on every animation frame and ran a **full-body rescan**
  (`querySelectorAll('*')`), re-queuing the ever-changing labels endlessly.
- The observer now processes **only the added nodes' subtrees** (not a full rescan), with a longer
  700ms debounce and two budgets: a global `MAX_TOTAL` (4000) string cap and an `MO_MAX` (600) cap on
  strings added from dynamic mutations - after which the observer disconnects and logs
  `dynamic-content translation paused (animated page)`. The initial viewport + on-scroll pass is
  unchanged, so static content still fully translates; animation no longer keeps the model busy.
  Re-toggling resets the budgets.

## [1.47.1] - 2026-07-01

### Fixed - DApp translation stripped spaces between styled words
- Headings split into several inline `<span>`s (e.g. XBTS "Top **decentralized** Exchange", each word a
  differently-coloured span) came out glued together ("Topdecentralized…"). Root cause: the content
  script sent each text node trimmed and then wrote the translation back with `nodeValue = tr`, dropping
  the node's original leading/trailing whitespace - so the space that lived on a neighbouring inline
  node was lost. `applyItem` now re-attaches the original node's leading/trailing whitespace around the
  translated core (`leadTrail()`), so inter-word spacing is preserved.

## [1.47.0] - 2026-07-01

### Added - Translation Phase 3: P2P sharing of the translation cache (GPU → CPU)
- **Per-language gossip topics `tr:<lang>`.** New `translator::tr_net` module (feature-gated on
  `p2p`, with a no-op stub for the non-p2p build) that rides the same iroh endpoint/gossip handle as
  the wall and chat layers (wired in `p2p.rs` right after `social::net::install`). Topic id is
  `blake3("smartnet-tr:v1:" ‖ lang)`; packets reuse the existing stealth anti-DPI padding frame and
  are capped at 32 KB.
- **GPU builds publish, CPU builds consume.** When a **CUDA/Metal** build translates a DApp string
  with the local Qwen model, it now broadcasts `(blake3(text), translated, lang, orig_len)` to
  `tr:<lang>` (originals where translation == source are skipped to save traffic). **CPU builds run in
  a new "cache-only" mode**: `dapp_translate_batch` never invokes the model on a miss - it subscribes
  to `tr:<target>` and serves whatever GPU peers have shared. Both build types subscribe to the target
  language so a GPU node can also benefit from other GPU nodes.
- **Inbound validation.** A received translation is written to the local Sled DApp cache
  (`social:translation:dapp:<hash>:<lang>`) only if it passes basic checks: 64-char hex hash,
  non-empty, ≤ 8192 chars, and length ≤ 4× the original + 40 (`orig_len` travels in the packet).
  Existing cache entries are never overwritten. On a new entry the backend emits
  `translator-cache-updated`.
- **DApp translate button re-enabled on CPU (cache-only).** The address-bar 🌐 toggle is no longer
  greyed out on a CPU build; it works in cache-only mode (purple accent + `apps.translateCacheOnly`
  tooltip, en/ru). `AppLayout` listens for `translator-cache-updated` and, if translation is active on
  the current DApp, re-runs the pass to pick up freshly shared strings live. Auto-translate and the
  right-click "Translate selection" config now run on every build (respecting cache-only on CPU).
- Bumped to v1.47.0.

> NOTE: same environment caveat - the `candle`/CUDA/Metal backends and the full desktop build are
> compiled locally by the user (`yarn tauri:build` / `:cuda` / `:metal`). Verified here:
> `cargo check --features p2p` GREEN and frontend `yarn build` GREEN.


## [1.46.0] 2026-06-30

### Fixed / Changed - DApp translation quality, safety & GPU gating
- **GPU-only.** DApp page/selection translation is now enabled only in **CUDA/Metal builds** (too slow
  on CPU). New `translator_build_accel` command exposes the compile-time accelerator; on a CPU build
  the 🌐 address-bar icon is greyed out and disabled with a "GPU builds only" tooltip.
- **Never translate identifiers.** The content script now skips STH base58 addresses, hex hashes,
  truncated `abc…def` tokens, alphanumeric IDs, tickers and long base64-like tokens - so wallet
  addresses and transaction hashes are left intact.
- **Layout-safety guard.** The backend discards model output that looks like an *explanation* rather
  than a translation (contains "translates"/"has no meaning" etc.) or that balloons in
  length (> 4× source + 40 chars), keeping the original instead - this fixes garbled UI where a single
  word like "Assets" was replaced by a long sentence and broke the page layout.

### Added - Translate selection via right-click (short strings)
- Selecting text in a DApp and right-clicking now shows a **"Translate to my language"** item that
  translates just the selection and shows the result in a small popup near the cursor (great for short
  labels the page-wide pass skips). Wired via the new `dapp_translate_config` command; enabled on
  `dapp-loaded` for GPU builds only. Same Sled hash-cache and event bridge.

## [1.45.0] 2026-06-30

### Added - DApp page translation (Phase 2): attributes, progress, memory, terminal logs
- **Attribute translation.** Beyond text nodes, the content script now also translates `title`,
  `placeholder`, `alt` and `aria-label` attributes (tooltips, form hints, image captions). The
  collection is shallow per element and driven by the same viewport/lazy IntersectionObserver, so
  offscreen attributes translate on scroll.
- **Pulsing icon + progress badge.** While a page is being translated the 🌐 address-bar icon pulses
  (cyan glow). A small badge shows live `N%` progress while running and, once finished, `<cachePct>% ·
  <seconds>s` (how much came from the local Sled cache and how long it took). The content script emits
  a debounced `dapp-translate-progress` event; `dapp_translate_batch` returns a parallel `from_cache`
  flag array so the badge can report the cache-hit ratio.
- **Per-DApp memory.** The translate choice is remembered per DApp address in `localStorage`; when a
  remembered DApp finishes loading (`dapp-loaded`), translation auto-enables - no re-click needed.
- **Terminal-console logs.** The DApp terminal now logs `[sth://i18n] translation started → <LANG>`
  and `[sth://i18n] translation done · N strings · M% cache · Xs` (and `translation disabled`),
  mirroring the existing `[sth://system]` / `[sth://crypto]` lines.

## [1.44.0] 2026-06-30

### Added - DApp page translation (Phase 1): in-webview DOM translation
- DApps (`sth://` sites) can now be translated to the UI language with the local Qwen translator,
  using a browser-style **DOM translation** approach (like Immersive Translate) - NOT server-side HTML
  parsing, which would break JS apps and dynamic content.
- **Address-bar toggle.** A 🌐 icon sits in the top address bar (left of the copy button), visible only
  when a DApp is open. Click = translate the page; it stays highlighted (cyan glow) while active; click
  again = restore originals. State is tracked per open DApp address.
- **Content script** (`dapp_i18n_script`, injected as an init script into every DApp webview alongside
  the existing web3 bridge): walks visible text nodes, translates the **viewport first and the rest
  lazily on scroll** (IntersectionObserver, `rootMargin` 150px), handles dynamic SPA content
  (debounced MutationObserver), stores originals for one-click restore, and skips `script/style/code/
  pre/textarea`, editable fields, numbers/symbols and non-letter strings. Batches ≤50 strings per
  request; request IDs are salted per webview session to avoid cross-window collisions.
- **Backend** `dapp_translate_batch` (event bridge `dapp-translate-request` → `dapp-translate-response`,
  mirroring the web3 `dapp-request` pattern): dedupes identical strings, **caches by content hash** in
  Sled (`social:translation:dapp:<blake3(text)>:<lang>`, capped at 5000 newest) so re-enabling the
  toggle or revisiting pages never re-runs the model, and - since the key is the content hash - an
  updated DApp automatically produces a fresh translation. **Skips strings whose detected language
  already equals the target** (e.g. English page + English UI → no translation), via whatlang.
- New Tauri command `dapp_translate_toggle(address, on, target)` evals the controller into the
  `embed-<addr>` / `app-<addr>` webview. Bumped to v1.44.0.

## [1.43.0] -29

### Added - Optional GPU acceleration for the local AI translator
- The local neural translator (candle / Qwen2.5-1.5B GGUF) can now run on a GPU instead of the CPU.
  New `translator::select_device()` picks the accelerator at engine init with a safe fallback to CPU
  if the GPU is unavailable at runtime. Two new **opt-in** Cargo features (OFF by default so the
  client still builds anywhere without extra toolchains):
    - `cuda` → external NVIDIA GPU (requires the CUDA Toolkit): `yarn tauri:build:cuda`
    - `metal` → Apple Silicon / macOS (works out of the box): `yarn tauri:build:metal`
      (npm scripts `tauri:build:cuda` / `tauri:build:metal` added; they map to
      `tauri build --features p2p,cuda|metal`.) `TranslatorStatus` gained an `accelerator` field
      (`"cpu"|"cuda"|"metal"`); `WallFeed` shows a green `⚡ CUDA/METAL` badge next to the translate
      toggle when a GPU is active.
- **Adaptive GPU profile.** When a GPU is active the translator drops the `min(4, cpus/2)` CPU thread
  cap (kept only on CPU so P2P seeding never freezes) and raises the per-translation token budget
  (`MAX_NEW_TOKENS` 640 → 2048) so long posts translate fully and fast. On CPU the sequential 1.5B
  token generation under the thread cap is the speed bottleneck - a GPU build removes it.

### Changed - Auto-translate UX on the Wall
- When **auto-translate is ON**, foreign posts AND comments are now translated automatically (comments
  translate as soon as they are expanded). The manual **Translate** button is hidden in auto mode and
  only appears when auto-translate is OFF (manual mode), matching the intended behaviour.
- **Live new posts are auto-translated too.** A `watch` on the post-id list re-runs auto-translation
  whenever the feed changes (a post arriving live over gossip, a refresh, pagination), so a newly
  received foreign post is translated in the background on the current author's page without needing a
  manual click - previously only posts present at load/refresh time were auto-translated.
- **Auto-translate toggle always visible on other walls.** The toggle no longer requires the language
  pack to be installed (`trStatus.installed`) to render - it previously vanished on any client where
  the pack wasn't downloaded yet. It now always shows on another user's wall; turning it on (or
  clicking a per-post Translate button) with no pack starts the background model download and shows the
  progress bar.

### Added - Smart Sled translation cache with staleness invalidation + per-author cap
- The translation cache moved to a composite key **`social:translation:<author>:<post_id>:<target_lang>`**
  with a JSON value `{ src_text_hash, translated_text, timestamp }`.
- **Staleness protection:** on each translate request the backend recomputes the blake3 hash of the
  CURRENT source text and compares it with the stored `src_text_hash`. If they differ (the post was
  edited / extended) the stale entry is deleted, Qwen re-runs inference, and the cache is overwritten
  with fresh data - so an edited post never serves an outdated translation.
- **Memory cap per author:** `prune_author` keeps at most **200** entries under each author's prefix
  (the newest by `timestamp`), evicting the oldest - bounding the cache size for each wall (last ~200
  translated posts + their comments). Applied on every write.
- `translate_text` gained an `author` param (wall owner); `translate_and_cache` (prewarm) and the
  frontend `translateText(author, …)` binding updated accordingly.
- **Settings → "Clear translation cache".** New commands `translation_cache_stats` (count / total
  bytes / average entry size across `social:translation:*`) and `clear_translation_cache` (wipes the
  whole subtree, returns the removed count). Settings → Storage shows the cache size, record count and
  average size with a Clear button. i18n `storage.trCache*` (en/ru).

### Added - System Console logging for the AI model auto-check
- The background language-pack check that fires ~120 s after launch now logs to the floating System
  Console (`sys` scope): "checking for the model on disk", "already installed", "not found - starting
  background download (torrent, HTTP fallback)", the torrent→HTTP switch on stall, and the final
  installed/error line - so the auto-download is visible instead of silent.

### Added - Background iroh download of the translator language pack (previously unlogged)
- **iroh-only download.** The language pack (tokenizer.json + Qwen2.5-1.5B GGUF) is fetched over **iroh
  blobs** - our own transport - directly into `data/models`. `iroh_fetch` pulls each blob via
  `crate::p2p::fetch_file(IROH_NODE, <cid>, <dest>)` with a live progress pump (reads
  `crate::p2p::blob_progress` → emits `translator-progress`). The old torrent + HTTP-fallback paths
  were removed from the auto-bootstrap for simplicity.
- **No wait-timeout.** Since iroh's `download_complete_any` only retries 5 times before giving up,
  `fetch_blob_with_progress` now loops indefinitely (retry every 15 s, logged to the System Console)
  until the file is fully on disk - the pack download is never abandoned on a transient network drop.
- `translator_bootstrap` is triggered in the background ~120 s after app launch. (A separate manual
  `translator_download` HTTPS command remains available.)
- **Global download status bar.** New `TranslatorBar.vue` (mounted in `App.vue`, bottom of the window,
  purple/cyan) listens for `translator-progress` globally and shows "Downloading language model · N/M
  MB · %" on any page - previously the progress was only surfaced inside `WallFeed` (and only when
  viewing another user's wall), so the background download was invisible.
- **HTTP fallback.** If the model torrent gets no live seeders (no bytes for ~90 s), the bootstrap now
  aborts the torrent and falls back to a direct HTTPS download from HuggingFace, so the pack reliably
  completes even when the community torrent is cold. Same `translator-progress` events drive the bar.
- **Persistent, navigation-safe download progress.** The model magnet is added into the shared librqbit
  session (same one the Torrents tab lists), so the download persists and resumes across page
  navigation - it is never reset by leaving the wall. New `translator_progress` command returns the
  last known `DownloadProgress` (kept in a `LAST_PROGRESS` static, updated on every emit); both the
  global `TranslatorBar` and the wall's language-pack block now query it on mount, so returning to a
  page immediately re-shows an in-progress download instead of waiting for the next live event.

> NOTE: same environment caveat as v1.41.0 - `candle` (and now the optional CUDA/Metal backends) can't
> compile on the ~99%-full container disk, so the Rust side is verified locally via `yarn tauri:build`.
> Frontend `yarn build` is GREEN.

## [1.42.1] -26

### Fixed - Translator build error on candle < 0.11 (`clear_kv_cache` not found)
- The local `yarn tauri:build` failed with `no method named clear_kv_cache found for
  quantized_qwen2::ModelWeights` because that method only exists in candle 0.11+, while the resolved
  version was older. Removed the call: candle's `forward` already resets the per-layer KV cache when
  `index_pos == 0` (verified against the quantized_qwen2 source), and every translation's first
  forward runs at `index_pos = 0` - so contexts never bleed between translations. This compiles on
  all candle 0.9–0.11 versions. (Remaining `tauri-plugin-shell` deprecation and `p2p.rs`
  unused-assignment warnings are pre-existing and non-fatal.)

## [1.42.0] -26

### Added - Background pre-translation of subscription posts
- **The feed opens already translated.** New Rust command `translate_prewarm(target_lang)` spawns a
  background task (guarded by a `PREWARMING` flag) that walks the user's followed walls
  (`social::wall_follow_list`), reads their recent posts (`social::wall_post_texts`, up to 30/wall),
  skips anything already cached, detects foreign-language posts (`whatlang`), and translates + caches
  them in Sled (`social:translation:<post_id>:<lang>`) via `translate_and_cache`. Runs in
  `spawn_blocking` under the same CPU-thread cap (UI/P2P never freeze), caps at 80 translations per
  run, and emits `translator-prewarm` progress. `WallFeed` triggers a prewarm when the pack is
  installed and auto-translate is on (on mount, on toggle-on, and after a fresh download completes).
  Added `social::wall_post_texts` / `social::wall_follow_list` helpers and the `translatePrewarm`
  bridge binding.

> NOTE: same environment caveat as v1.41.0 - `candle` can't compile on the ~99%-full container disk,
> so the Rust side is verified locally via `yarn tauri:build`. Frontend `yarn build` is GREEN.

## [1.41.0] -26

### Added - LOCAL neural translation layer for the Wall (AI Translation Layer)
- **Fully offline, private post/comment translation** - no external APIs. New Rust module
  `translator.rs` runs CPU inference via `candle-core` + `candle-transformers`
  (`quantized_qwen2::ModelWeights`) on the quantized **Qwen2.5-1.5B-Instruct GGUF (Q8_0, ~1.89 GB)**.
    - **Thread cap:** `min(4, max(1, num_cpus/2))` via `RAYON_NUM_THREADS`/`CANDLE_NUM_THREADS` so P2P
      seeding and the UI never freeze. All inference/downloads run in `spawn_blocking`.
    - **Lazy init:** weights + tokenizer load once into a global `Mutex<Option<Engine>>`; KV cache is
      cleared before each translation. Strict Instruct system prompt to avoid hallucinated commentary.
    - **Sled cache:** `social:translation:<post_id>:<target_lang>` - cached translations return instantly.
    - **Downloader:** `translator_download` streams the GGUF (+ `tokenizer.json` from the base repo)
      with `ureq` in `spawn_blocking`, emitting `translator-progress` (%/MB) to the UI. Pre-checks ≥1.5 GB
      free disk (hard) and warns (soft) if available RAM < 1 GB.
    - **Language detection:** `detect_lang` via `whatlang` (ISO-639-1) to decide "foreign" posts.
    - No `unwrap()` - all errors returned as `Result::Err(String)`.
    - New commands: `translator_status`, `translator_download`, `detect_lang`, `translate_text`.
      Deps added: `candle-core`, `candle-transformers`, `tokenizers`, `num_cpus`, `whatlang`
      (+ sysinfo `system` feature). Model path: `data/models/Qwen2.5-1.5B-Instruct-q8_0.gguf`.
- **Vue UI (`WallFeed`).** Header toggle "Translate to my language" (persisted in localStorage). Auto
  mode: foreign posts are detected and swapped to the translated text with a fade transition; manual
  mode: a 🌐 Translate button per post/comment with a neon spinner, plus a "Show original" toggle. When
  the pack is missing, a cyberpunk block "Language pack required (~1.8 GB)" with a live progress bar is
  shown. Target language follows the UI locale (RU/EN/ID). i18n RU/EN.

> NOTE: `candle` is very heavy to compile and the container disk is ~99% full, so `cargo check
> --features p2p` could NOT be run in this environment - the Rust code is written to the candle 0.9 /
> quantized_qwen2 API and must be verified locally via `yarn tauri:build` (per agreement with the user).
> Frontend `yarn build` is GREEN.

## [1.40.0] -26

### Fixed - Playlist track that wouldn't play until played on the wall first
- Playing a freshly-added playlist track sometimes did nothing until the same track was played on the
  wall (which downloaded + cached the blob locally). Root cause: cold-start P2P fetch - the first
  `social_file_bytes` fetch from the source node could fail/time out before the connection was warm,
  and the error was swallowed silently. `BottomAudioPlayer.loadCurrent` now uses `fetchAudioBytes`
  which retries once after 800ms, then falls back to provider discovery with no fixed node (handles a
  stale/unreachable source node while other peers still seed the content).

### Added - Shuffle & repeat for the playlist
- Added **shuffle** (🔀) and **repeat** (🔁 all / 🔂 one / off) toggles as icon indicators next to the
  playlist title (`WallPlaylist`). Backed by new player-store state (`shuffle`, `repeat`) and honored
  by `next()` and the track-ended handler: repeat-one replays, shuffle picks a random next track,
  repeat-all loops back to the start. i18n RU/EN (`wall.shuffle/repeatOff/repeatAll/repeatOne`).
  Verified: `yarn build` GREEN.

## [1.39.0] -26

### Added - Drag-and-drop playlist reordering
- **Custom playlist order, persisted in the DB.** The personal playlist (`WallPlaylist`) can now be
  reordered by drag-and-drop on the owner's own profile. Chosen storage: Sled (consistent with the
  rest of the app; a JSON file would be a separate persistence path). A new key
  `social:pl_order:<owner>` holds a JSON array of track CIDs; `playlist_list` sorts by it (ordered
  tracks first in saved order, any not-yet-ordered tracks appended newest-first). New command
  `playlist_reorder(order)`. Rows show a ⠿ grip, become `draggable` for the owner, and dropping
  persists the new order via the bridge. i18n RU/EN (`wall.dragReorder`).
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

## [1.38.0] -26

### Added - Reputation system + paid dislike
- **On-node reputation ledger (`reputation.rs`).** Stored in Sled at `reputation:<address>` as i64.
  Forward-only accrual from this release: **+1000** one-time on name registration (hooked in
  `naming::commit_name`, set only if absent), **+10** per new subscriber and **+1** per following
  (hooked in `subs::subscribe`), **+100** per album purchase (`social::album_buy`), **+250** per paid
  dApp publish (`apps::finalize_publish`). Already-registered addresses with no record are lazily
  backfilled to **2000**. New command `get_reputation(address)`.
- **Paid dislike (`social::dislike_user`).** Costs 1 STH split as two `reputacion-off` transfers
  (fee 0.1): **0.9 STH → burn address** `smartHoLdemBurnAddrHereXXXmUW7f` and **0.1 STH → the
  disliked author**; then decrements the target's reputation by −1. Command `dislike_user(target)`.
- **Comment gating.** `social::add_comment` now rejects commenting on *other users'* walls when the
  commenter's reputation is < 1 (own wall always allowed). The error surfaces in `WallFeed` via the
  existing flash toast.
- **Profile UI.** Reputation badge (⚡ Reputation: N) shown under the username; a 👎 dislike pill added
  after the ❤/👍 counters (other profiles only) that opens a `TxConfirmModal` (1 STH, `reputacion-off`)
  before broadcasting. i18n RU/EN (`profile.reputation/reputationHint/dislike*`).
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

## [1.37.0] -26

### Added - Torrent tab: seeding stats + My Node card
- **Profile → Torrents tab now shows live stats (own profile only).** Replaced the "coming soon"
  placeholder with three counters - **My seeds** (torrents at 100% / total active), **Total
  downloaded** and **Total uploaded** (from `get_profile_stats`, with live ↓/↑ speed sub-labels
  aggregated from `get_active_torrents`) - plus a **My Node** card mirrored from the Network Map
  (Byte-Hours value, rank badge Leech→Peer→Seeder→Anchor with rank color, total uploaded, and a
  progress bar to the next rank). Auto-refreshes every 4s while the tab is open; interval cleared on
  unmount. Visitors see a "stats are local to your own node" hint. i18n RU/EN
  (`profile.mySeeds/activeTorrents/totalDownloaded/totalUploaded/torrentLocalOnly`). Frontend only.
  Verified: `yarn build` GREEN.

## [1.36.0] -26

### Fixed - Clickable album banner for legacy announcement posts
- **Old album announcements are now clickable for subscribers.** Album posts published before the
  `AlbumRef` banner existed (e.g. the 528Hz album) showed only plain text on a follower's wall.
  `WallFeed` now fetches the wall author's locally-known albums (propagated via the wall snapshot,
  `Snapshot.albums`) and, for any announcement post missing an `AlbumRef`, synthesizes one by
  matching the post's cover CID (or the album title inside the post text) - so the clickable album
  banner (mini cover, title, price/FREE, CTA) renders and opens the album page. Newly published
  albums already carried the banner; this back-fills existing posts without a re-publish. Frontend
  only. Verified: `yarn build` GREEN.

## [1.35.0] -26

### Added - Album sales sparkline (demand dynamics)
- **Per-album sales chart.** New Rust command `album_sales_series(id)` returns a sorted time series
  of `{ts, amount}` points from the owner's confirmed incoming `albumbuy:<id>:` payments. New
  `AlbumSalesSparkline.vue` (author-only, shown on the paid album page under revenue) buckets the
  points by **day** or **week** (toggle), fills gaps with zeros over the range, and renders a neon
  bar chart with per-bucket tooltip (date · count · STH) plus Total / Peak footers. i18n RU/EN/ID
  (`music.salesDynamics/byDay/byWeek/noSalesYet/chartTotal/chartPeak`).
  Verified: `yarn build` GREEN, `cargo check --features p2p` GREEN.

## [1.34.0] -26

### Added - Per-album author revenue
- **Album revenue for the author.** New Rust command `album_revenue(id)` sums the STH amount of all
  confirmed incoming `albumbuy:<id>:` payments in the owner's on-chain history. The album page now
  shows a "💰 Revenue: N STH" line (author only) below the "Sold: N" counter. i18n `music.revenue`.

### Fixed - Milkdrop visualizer
- **Visualizer stayed black after close → reopen.** Closing the visualizer removes its canvas from
  the DOM (v-if), but the butterchurn instance still held the detached canvas, so reopening rendered
  to nowhere. Close now cancels the render loop and drops the visualizer reference so it is recreated
  against the fresh canvas on the next open. The render loop is (re)started once per open (no stacking).
- **Full drag-and-drop.** The whole visualizer window is now draggable anywhere on screen (was only
  the small toolbar). A movement threshold disambiguates drag from click, so a plain click still
  toggles standard ↔ fullscreen while a drag repositions the window (grab/grabbing cursor + ⠿ handle).
  i18n `player.drag`.
  Verified: `yarn build` GREEN, `cargo check --features p2p` GREEN.

## [1.33.0] -26

### Added - Album edit/delete + music UX polish
- **Edit & delete albums (author only).** New Rust commands `album_update` (edit title, artist,
  year, note, price and optionally a new cover - metadata only, tracks unchanged; re-signs and
  re-seeds the manifest with a fresh CID, then rebuilds the wall snapshot) and `album_delete`
  (removes the album manifest from the author's local DB so it disappears from the author's
  profile/feed and future snapshots). Delete does NOT propagate a purge or remove track blobs, so
  buyers/downloaders who already have the album keep their copy and access. `AlbumCard` shows
  ✎ Edit / 🗑 Delete (with an inline confirm) for the owner; the Studio modal now has an edit mode
  (prefilled fields, track list locked).
- **"Albums by other authors" block on my Music tab.** The profile Music tab now shows a second
  section (own profile only) listing all locally-known albums acquired from other authors
  (`albumFeed()` filtered to `owner !== me`) - bought or added-to-me albums surface there.

### Fixed / Changed - player & album page UX
- **Queue fly-out pause.** Clicking the currently-playing track in the bottom-player queue now
  toggles pause/resume instead of restarting it.
- **Album page now-playing indicator.** On the album detail page the currently-playing track shows a
  ❚❚ pause icon (instead of ▶) and is highlighted with a neon row style; clicking it toggles pause.
- **Milkdrop visualizer is now a floating window.** Removed the forced fullscreen overlay; the
  visualizer opens as a standard-size, draggable panel (bottom-right). Clicking the canvas (or the
  ⛶/🗗 toolbar button) toggles between standard and fullscreen size. It stays in the main webview so
  it keeps direct access to the player's `AudioContext` (a separate child webview cannot share it).
- **Wider microblog column.** The profile wall's center column max-width raised 600 → 900 px to use
  the empty space on the right.
- i18n RU/EN/ID (`music.edit/save/delete/…`, `music.otherAuthorsAlbums`, `player.toggleSize`).
  Verified: `yarn build` GREEN, `cargo check --features p2p` GREEN.

## [1.30.0] -26

### Added - Album year + author note in the Studio
- The Musician's Studio now has a **Year** field and an **Author note** textarea (≤2000 chars).
  Both are stored on the signed album manifest (`Album.year`, `Album.note`, included in the
  canonical signature) and shown on the album page - year next to the track count, and the note as
  a description block under the actions.
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

## [1.29.0] -26

### Fixed / Added - Paid album is now reachable & purchasable from the wall
- **Clickable album banner on the wall.** Album announcement posts now carry a structured
  `AlbumRef` (`Post.album` / `PostView.album`): id, manifest CID, node, title, cover, price,
  free, track count. `WallFeed` renders a clickable album banner (mini cover, title, price/FREE,
  CTA "Get for N STH" / "Play all") that opens the album page - fixing the bug where a paid album
  showed on the author's wall but couldn't be opened or bought. `create_post` gained an optional
  `album` param (existing callers pass `null`).
- **`sn://album/…` wired into the global link resolver.** The address/link resolver in `AppLayout`
  now handles `sn://album/<id>~<cid>~<node>` (and `u://album/…`): it fetches + verifies the manifest
  via `album_open_link` and routes to the album page, so album links are clickable from the URL bar,
  chat, and anywhere links are resolved (consistent with `sn://files`, `sn://file/<cid>`).
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

### Note
- The album banner appears on **newly published** albums. The already-published 528Hz album has an
  older announcement post without the banner; re-publishing (or opening via the profile "Music"
  tab once the wall snapshot with `Snapshot.albums` has synced) will show it. Snapshot-based album
  propagation (v1.28.0) makes albums appear in a visitor's profile "Music" tab after a snapshot fetch.

## [1.28.1] -26

### Added
- **Quick "Share" button on album cards.** Album cards (profile "Music" tab + global page) now have
  a small 🔗 button that generates the `sn://album/…` link and shows the QR/copy modal inline, without
  opening the album page - faster link-sharing for musicians.

## [1.28.0] -26

### Added - Album sales counter, network sharing (sn:// + QR), magnet export
- **Sales counter.** `album_sales(id)` counts unique buyers from confirmed on-chain `albumbuy:<id>:`
  payments in the owner's history; shown on the paid album page ("Sold: N").
- **In-network sharing.** On publish, the signed album manifest is now seeded as its own iroh blob
  and its CID stored on the album. `album_share_link(id)` returns **`sn://album/<id>~<cid>~<node>`**
  (service scheme - `sth://` is reserved for dApps/names). `album_open_link(link)` fetches the
  manifest blob by CID, **verifies the owner's signature**, stores it locally and returns the id.
  The Music page gained a "paste link → open" input, and the album page a **Share** button showing
  the link + QR (reusing `FileQrModal`).
- **Passive discovery via wall snapshot.** Album manifests are now embedded in the artist's wall
  snapshot (`Snapshot.albums`) and verified + stored on ingest, so followers/visitors automatically
  get album cards + working album pages - not just the wall post.
- **Magnet export (external world).** For free albums, `album_export_torrent(id)` gathers the tracks
  into a temp folder and creates a torrent via librqbit, returning a **magnet link** shown as a QR -
  for sharing/seeding in the wider BitTorrent world (qBittorrent, etc.).
- i18n `music.*` (RU/EN/ID). New commands registered in `lib.rs`.
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

## [1.27.0] -26

### Added - Paid albums (direct-to-creator STH payment, like paid files)
- **Album pricing.** `album_create` now takes a `price`; `price > 0` marks the album paid
  (`Album.free=false`, `Album.price`). The signed manifest canonical now includes the price.
- **Payment mirrors the paid-file flow.** New `album_buy(id)` sends an STH transfer **directly to
  the album owner** with memo `albumbuy:<id>:<buyer>` (same mechanism as `filebuy:`), and
  `album_access(id)` grants access to free albums / the owner / anyone with a confirmed matching
  on-chain payment in the owner's history. New `album_get(id)` fetches a single manifest.
- **Dedicated album page (`AlbumView.vue`, `/album/:id`).** Cover, title, artist, price badge and
  full tracklist. Free/owner/purchased → "Play all" + playable tracks (routed to the global player).
  Paid & not purchased → **"Get for N STH"** button → pay → poll access (10s, +8s) → unlock,
  mirroring the file download page.
- **Studio.** Album publishing gained a "Paid album" toggle + STH price field.
- **Album cards.** Show a price badge; for a paid album a non-owner hasn't bought, the card's action
  and cover open the album page instead of playing. Paid album wall announcements omit the track
  files (cover + text only) so paid content isn't handed out for free.
- i18n `music.*` payment keys (RU/EN/ID). New commands registered in `lib.rs`.
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

## [1.26.1] -26

### Fixed
- **Milkdrop visualizer rendered a black screen.** butterchurn was drawing into the canvas's
  default 300×150 backing store while its WebGL viewport used the full overlay size, so nothing was
  visible. The canvas drawing buffer (`canvas.width/height`) is now set explicitly to the overlay
  size × devicePixelRatio before `createVisualizer` and on every resize, and `render()` is wrapped
  so a single bad frame can't kill the loop. Overlay init now waits for a double `requestAnimationFrame`
  so the canvas has real layout dimensions.

### Added
- **Empty player auto-loads the user's playlist.** Opening the player (sidebar "Player") while the
  queue is empty now loads the user's personal playlist into the queue (paused, ready to play) via
  the new `setQueue` store action, so there's always something to press play on.

## [1.26.0] -26

### Added - Musician's Studio + free albums (iroh collection, slice 1)
- **Free album architecture (iroh-native).** An album is a **signed manifest** listing
  per-track content-addressed iroh blobs + a cover blob - no giant tar, so a single track can be
  streamed without pulling the whole album, and shared art dedups to one blob. Chosen over a
  whole-folder torrent because it reuses SmartNet's censorship-hardened stack (QUIC/TLS relays,
  Stealth Mode, headless seeder, tracker, byte-hours) and re-seeds reliably after announce.
- **Backend (`social.rs`).** New `Album` / `AlbumTrack` models (`social:album:<owner>:*` in Sled),
  `album_canonical` + secp256k1 signature. New commands: `pick_audio_files` (multi-select),
  `album_create` (processes cover → `wall/images`, adds each track via `files::add_file`, extracts
  ID3 title/artist + per-track APIC cover, signs the manifest, seeds all blobs, and **also publishes
  a wall post** with the tracks so followers see & play it immediately), `album_list(owner)`,
  `album_feed()`.
- **Musician's Studio UI (`MusicStudioModal.vue`).** Pick cover, add MP3s (multi-file), edit
  per-track titles, set album title/artist, publish. Progress + toasts.
- **Album display (`AlbumCard.vue`).** Cover, title/artist, track count, "Play all" and an
  expandable tracklist - every play routes into the global bottom player (continuous queue).
- **Two entry points.** New global **"MUSIC"** sidebar page/route (`Music.vue`) with the album
  feed + Studio button, and a new **"Music" tab** in the user profile (own albums + Studio for the
  owner). i18n `nav.music` + `music.*` (RU/EN/ID).
- Magnet/QR export and paid albums are deferred to the next slices.
  Verified: `yarn build` GREEN, `cargo check --features p2p` GREEN.

### Note
- MVP discovery: album manifests are stored locally; visitors currently see a published album as a
  normal wall post (fully playable). The dedicated album cards on the profile "Music" tab / global
  page show albums whose manifests are held locally (own + received). Network-wide album manifest
  propagation is a later slice.

## [1.25.1] -26

### Fixed
- **Spectrum visualizer was inverted.** During playback the analyser can return silence in the
  desktop webview, so the bars looked "off" while playing and the idle animation only ran when
  paused. The bottom-player spectrum now animates **while playing** (real analyser data, or a
  synthetic live animation as a fallback when the analyser is silent) and shows a **calm flat line
  when paused**.
- **Left-sidebar "Player" did nothing when no track was active.** The player root only rendered
  with an active track, so clicking "Player" with an empty queue showed nothing. The queue panel
  now opens on click even without a track (anchored to the screen bottom) and shows an
  "empty queue" hint; the 70px bar still appears only once a track is playing. i18n `player.emptyQueue`.

## [1.25.0] -26

### Added - Album cover art (ID3) + Milkdrop visualizer in the global player
- **Embedded cover art (ID3 APIC via `lofty`).** On attaching an audio file
  (`social_attach_file`), the embedded front-cover picture is extracted (`lofty` pictures,
  preferring `PictureType::CoverFront`), decoded and re-encoded to a PNG ≤512×512, stored
  **content-addressed** in `wall/images/<blake3>.png` and seeded over iroh-blobs. Only the
  small cover **CID** (`PostFile.cover` / `PlaylistTrack.cover`) travels in the signed gossip
  envelope, and identical album art dedups to a single blob (one cover for many tracks).
- **Cover thumbnails in the player.** The bottom bar's now-playing box and each fly-out queue
  row show the album cover (fetched over P2P via `fetch_wall_media`, cached per cover CID),
  falling back to the neon disc placeholder - a Bandcamp-style look.
- **Milkdrop visualizer (butterchurn).** A new visualizer button (◉) on the player bar opens an
  immersive full-area WebGL overlay driven by `butterchurn` + `butterchurn-presets`, connected to
  the shared `AudioContext` source node. A random preset loads on open; ⤳ cycles to another random
  preset (name shown), ✕ closes. `butterchurn` and its preset pack are **dynamically imported**
  (code-split into lazy chunks, ~198 KB + ~646 KB) so they never bloat the initial bundle.
- i18n `player.visualizer` / `player.nextPreset` (RU/EN/ID). New deps: `butterchurn` 2.6.7,
  `butterchurn-presets` 2.4.7.
  Verified: `yarn build` GREEN (butterchurn code-split), `cargo check --features p2p` GREEN.

## [1.24.0] -26

### Added - Global cyberpunk bottom audio player (Web3 music, Iteration 1: UI shell)
- **Persistent global player (`store/player.ts` + `components/BottomAudioPlayer.vue`).**
  A fixed 70px glassmorphism bar (neon cyan/purple accents, `backdrop-filter: blur(14px)`)
  mounted once at the `AppLayout` root, so **playback survives route navigation** (state lives
  in a Pinia store; the single `<audio>` element is never remounted). The bar only appears
  while a track is active and can be dismissed (✕) to clear the queue.
- **Canvas spectrum analyzer.** A `<canvas>` visualizer driven by `AudioContext.createAnalyser()`
  (`fftSize: 128` → 64 bins) rendered with `requestAnimationFrame` (no SVG/`v-for` per bar, for
  performance). A single `MediaElementAudioSourceNode` is created lazily on first play and reused;
  blob object-URLs (same-origin) avoid any CORS tainting of the graph.
- **Fly-out playlist queue.** A slide-up (`<Transition>`) panel lists the queue with the current
  track highlighted; click to jump, ✕ to remove. Toggled by a queue button on the bar and by a new
  **"Player"** left-sidebar entry (music icon) docked just above the "MESSENGER" block.
- **Controls.** Play/pause, prev/next (prev restarts the track if >3s in), a seek slider bound to
  the audio's `currentTime`, a volume slider, elapsed/total time, and the now-playing title/artist.
- **Sources wired.** The profile **playlist** (`WallPlaylist.vue`) now loads the whole playlist into
  the global player (continuous auto-advance) instead of an inline `<audio>`; each Wall post audio
  attachment gets a **▶ "Play in player"** button (`WallFeed.vue`) that queues the post's audio files
  starting from the clicked track. i18n `player.*` (RU/EN/ID).
  Verified: `yarn build` GREEN (frontend-only; no Rust changes).

### Notes
- Free-vs-Paid track logic (torrent streaming for free tracks, in-memory AES decryption for paid
  tracks gated by an on-chain STH transaction) is the next iteration (Task 2) and is intentionally
  NOT part of this UI-shell iteration.

## [1.23.0] -26

### Added
- **Personal music playlist.** Users can add any post's audio track to their own playlist via a "＋"
  button next to the player. The playlist is stored per-user in `Sled` (`social:pl:<owner>:*`) and
  shown on the profile's right sidebar, below the subscriptions block, with a play button per track
  (sequential auto-advance) and a remove button for the owner. **Other users can see and play**
  someone's playlist when visiting their profile. New commands: `playlist_add`, `playlist_remove`,
  `playlist_list`.
- **ID3 tags for audio.** Attached audio files are parsed with `lofty`; when present, the post and
  playlist show "Artist - Title" instead of the raw file name (falls back to the file name).

### Changed
- On wide screens (≥1600px) the **subscribers and subscriptions blocks sit side-by-side** (2-column
  grid) with the playlist spanning full width below; they stack again on narrower widths.
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

### Added
- **Audio playlist for posts.** When a post has multiple audio attachments they now play
  consecutively - when one track ends the next one auto-starts (loading it on demand if needed).
  A "🎧 Playlist · N" header appears when a post has 2+ audio files.
  Verified: `yarn build` GREEN (backend unchanged).

### Notes
- **Content de-duplication confirmed working** for both images and files (mp3/mp4/…): everything is
  content-addressed by BLAKE3 CID. Identical bytes yield the same CID and are stored once on disk
  (`wall/images/<hash>.png` written only if absent; `filesdata/<XX>/<cid>` keeps the temp file only
  when the target is missing - `hash_and_store_stream`), the manifest is keyed by CID, and iroh-blob
  seeding by hash is idempotent - so the same media referenced by many posts uses a single copy/blob.

### Added
- **Live Markdown preview** in the composer - "Write / Preview" tabs let the
  author see the rendered post (same `marked` + `DOMPurify` pipeline) before publishing.
- **Multiple file attachments per post (up to 10).** You can attach several audio (or other) files to
  one post. This is network-friendly: only tiny metadata (cid/name/size/mime ≈ 1 KB each) travels in
  the signed gossip envelope; the blobs are seeded by the author and downloaded by subscribers only on
  demand (on play/download). A 10-file cap is enforced client-side.
  Verified: `yarn build` GREEN (backend unchanged).

### Added
- **Markdown formatting toolbar** in the post composer (Bold, Italic, Heading, List, Link, Code) that
  wraps/prefixes the current selection - makes the new Markdown support usable without knowing syntax.
- **Double-click a post image** to open it full-size in a separate window (single click no longer
  triggers it, matching the requested behaviour).
  Verified: `yarn build` GREEN (backend unchanged).

### Added
- **Markdown in posts.** Wall posts now render Markdown (headings, bold/italic, lists, links, code,
  blockquotes) via `marked` + `DOMPurify` sanitisation - the same engine used for app READMEs.
- **"Show all" social-graph modal.** Each right-sidebar block (Followers / Following) has a
  "Show all" button opening a scrollable modal list of all entries; clicking a person navigates
  to their profile.

### Changed
- **Removed the profile page header** ("PROFILE: @name" + subtitle); the columns now sit at the top.
  Verified: `yarn build` GREEN (backend unchanged, `cargo check --features p2p` GREEN).

### Changed - VK-style wall layout
- The Wall tab now uses a VK-like three-column layout: the existing profile card on the left, a
  **narrower centered post column** (max-width 600px) in the middle, and a **social-graph sidebar**
  on the right split into two stacked blocks - **Followers** on top and **Following**
  (following) below (replaces the previous tabbed subscribers/following panel). Collapses to a single
  column under 1100px.
- Post images now render **full-width inside the post** (stacked, click → original in a new window)
  instead of a left-aligned thumbnail.

### Fixed
- **Deleting a post now also removes its images on subscribers.** `DeletePost` handling (and the
  owner's own delete) purge attached images from disk (`wall/images/` + the shared `social_cache/`)
  in addition to file attachments - nothing is left behind on followers' machines.
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

### Added - Wall file attachments, audio player, comment avatars, unread badge
- **File attachments on posts (mp3/mp4/png/jpg/…).** A new paperclip button in the composer opens the
  native picker and stores the file through the existing File-Exchange pipeline (`files::add_file`,
  public mode): hashed → saved to `data/filesdata/<XX>/<CID>` → manifest written to `Sled` → seeded
  over iroh-blobs. The private/paid (E2EE) options are intentionally hidden in the microblog context.
  Attachments ride inside the signed post envelope (`Post.files`, included in the signature), so
  subscribers see them but **do not auto-download** - bytes are fetched on demand over P2P.
- **Compact HTML5 audio player** for mp3 attachments (title + size + download button + `<audio controls>`);
  mp4 shown via a lazy `<video>`; image files as a framed thumbnail (click → original in a new window);
  other types as a file row (extension badge + name + size + download). Files show name + human size like
  the "My Files" page. Downloads use a native "Save As" dialog (`social_save_file`).
- **Full file lifecycle in the DB.** Attachments are tracked so they can be found and removed: deleting a
  post purges its attached files from disk + `Sled` (owner via `files::purge_file`), and subscribers drop
  their downloaded copies when they receive the signed `DeletePost` event.
- **Comment author avatars.** Comments now carry the author's avatar CID; the comment row shows it
  (falling back to the identicon).
- **"Unread posts" badge** on the profile Wall tab - sums new posts across followed walls
  (`wall_unread_counts`); opening a wall marks it read (`mark_wall_read`).
- New commands: `social_attach_file`, `social_file_bytes`, `social_save_file`, `mark_wall_read`,
  `wall_unread_counts`. `Post.files: Vec<PostFile>`, `Comment.avatar`. Verified: `cargo check --features p2p`
  GREEN, `yarn build` GREEN.

### Changed - Social commands made fully async (rule R1)
- All social wall Tauri commands (`list_wall`, `list_comments`, `post_reactions`, `react_to_post`,
  `delete_post`, `hide_comment`, `ban_commenter`, `list_blocked`, `follow_wall`, `get_profile_avatar`,
  `my_avatar_hash`, etc.) are now `async`; heavy `Sled` scans and secp256k1 signing run inside
  `spawn_blocking` so the UI thread never stalls (fixes the freeze on "follow wall").

### Fixed
- **Deleted posts no longer resurrect.** Added a per-post deletion tombstone
  (`social:del:<author>:<id>`); inbound `Post` events and snapshot merges skip tombstoned ids, so a
  stale snapshot fetched after a delete can no longer re-insert the removed post.
- **Author avatar visible to visitors.** The wall now records each author's current avatar CID+node
  (`social:avcid:<author>`, learned from posts/snapshots); the left profile column of a visited
  profile fetches and shows that avatar over P2P instead of the generic identicon.

### Added
- **Usernames in the feed.** Posts, comments and reaction lists show `@username` (resolved via the
  naming layer) when the address has a registered name, falling back to the short address.
- **"Who reacted" hover popover.** Hovering a reaction chip shows the list of reactors (by name).
- **Avatar next to the LOCK button.** The topbar avatar shows the uploaded avatar (synced live via an
  `avatar-updated` window event) instead of the identicon.
- **Post media redesign.** Attached images now render as a framed thumbnail to the left of the post
  text; clicking a thumbnail opens the original image in a new window, capped to the screen size.
- **Comments are indented** under their post (left offset + accent rule) for clearer threading.
- **Snapshot delivery hardened.** On a gossip `NeighborUp`, a subscriber (re-)sends `ReqSnapshot` so
  history requests reach a live neighbor; the owner also rebuilds/seeds its snapshot on startup.
- New commands: `author_avatar`; `ReactionSummary` now carries `reactors` (emoji → addresses).
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.

### Added - Social wall: emoji reactions (likes)
- **Signed reaction events over gossip.** Posts can now be reacted to with a single emoji per
  user (a "like"). Each reaction is a secp256k1-signed `Envelope::React` broadcast on the wall
  owner's `iroh-gossip` topic; peers verify the signature (`pubkey → reactor address`) and that
  the reaction targets a post on the topic owner's wall before storing it. Cheap on the network
  (tiny payload, reuses the existing wall topic - no extra subscription).
- **One reaction per user per post, toggleable.** Re-sending the same emoji removes the reaction;
  a different emoji replaces it. Stored in `Sled` (`social:react:<author>:<post>:<reactor>`, last
  write wins by timestamp). Aggregated counts + the caller's current emoji are returned by
  `post_reactions`; `react_to_post` performs the toggle and returns the fresh summary.
- **Included in wall snapshots** so reactions survive as offline history for new subscribers
  (`Snapshot.reactions`, verified + timestamp-merged on fetch; backward-compatible via serde default).
- **UI (`WallFeed.vue`).** Each post shows reaction chips (`emoji count`, the caller's own reaction
  highlighted) and a `☺+` button opening a quick-emoji row (👍 ❤️ 😂 🔥 🎉) plus the full
  `vue3-emoji-picker`. Live updates via the `social-event` (`react`) Tauri event. i18n RU/EN/ID.
- New commands: `post_reactions`, `react_to_post`. Verified: `cargo check --features p2p` GREEN,
  `yarn build` GREEN. (Runtime P2P flow on the user's local `yarn tauri:build`.)

### Added - Social Layer: decentralized wall (Phases 2 & 3)
- **Per-author gossip walls.** Each user's wall is an `iroh-gossip` topic derived from
  `blake3("smartnet-wall:v1" ‖ address)`. The author broadcasts **signed** events
  (post / comment / delete / moderation / snapshot) onto their own topic; subscribers
  receive them live. Backend engine in `social.rs` (`net` submodule, `#[cfg(feature="p2p")]`
  with a no-op stub for the non-p2p build). Bootstrap uses the author's on-chain-published
  iroh EndpointId (new `chat::published_endpoint`). All packets are wrapped in the existing
  stealth padding frame (anti-DPI) and capped at 60 KB.
- **secp256k1-signed content.** Every post/comment/moderation event carries the author's
  compressed pubkey + ECDSA signature; inbound events are verified (`pubkey → address`
  must match the claimed author, posts must originate from the topic owner) before being
  stored, so a peer cannot forge another user's wall.
- **Offline history via snapshot blobs.** The wall owner periodically rebuilds a
  `wall/snapshot.json` (last 100 posts + their comments + avatar CID), seeds it as an
  iroh-blob and announces the CID over gossip. New subscribers send a `ReqSnapshot`, fetch
  the latest snapshot blob and merge the verified history into local `Sled` - so a wall is
  readable even when the author is offline (as long as any peer seeds the blob).
- **Sled storage.** Posts (`social:post:<author>:<ts>:<id>`), comments
  (`social:cmt:<author>:<post>:<ts>:<id>`), wall follows (`social:follow:*`), owner bans
  (`social:block:<owner>:<addr>`), hidden comments (`social:hidden:*`), snapshot CIDs and a
  `social:seen:*` dedup set.
- **Microblog UI (`WallFeed.vue`)** in the profile's **Wall** tab: composer (text + photo
  attach, own wall only), reverse-chronological feed with lazily-fetched avatars/images
  (`fetch_wall_media` → data-URL over P2P, cached), collapsible comments with inline reply,
  follow/unfollow another user's wall, and live refresh via the `social-event` Tauri event.
- **Owner moderation.** The wall owner can **hide** a comment or **ban** a commenter;
  both are owner-signed moderation events propagated to subscribers, who honor them.
  Banned authors' inbound comments are rejected on the owner's node.
- New Tauri commands: `create_post`, `list_wall`, `delete_post`, `add_comment`,
  `list_comments`, `follow_wall` / `unfollow_wall` / `list_wall_follows` /
  `is_following_wall` / `open_wall`, `hide_comment` / `ban_commenter` /
  `unban_commenter` / `list_blocked`, `fetch_wall_media`. i18n RU/EN/ID (`wall.*`).
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN. (Runtime P2P flow on
  the user's local `yarn tauri:build`.)

### Added - User Profile: right-column tabs (Wall / DAPPs / Torrent)
- The profile's right column is now tabbed:
    - **Wall** - the social graph (subscribers / following) on top, plus a **Microblog**
      placeholder (detailed in the next phase).
    - **DAPPs** - the published-dApps showcase on top with the contribution stats below.
    - **Torrent** - placeholder for seeding / network-contribution stats (next phase).
- Frontend-only (`UserProfile.vue` + i18n RU/EN). `yarn build` GREEN.

## [1.12.1] -25

### Fixed
- Downloaded **private files showed no size** in "My Files" (inbox manifests were
  created with `size: 0`). After a download the real plaintext size is now recovered
  cheaply from the SFP2 header (blob size − header − GCM overhead, no decryption) and
  written to the manifest. `cargo check --features p2p` GREEN.

## [1.12.0] -25

### Added - Exchange endpoint: env-configurable + P2P fallback
- Exchange gateway is no longer hardcoded. It resolves at call time (cached ~30s):
  clearnet **`EXCHANGE_API`** (default `https://exchange.smartholdem.io`) first, then
  **`EXCHANGE_API_P2P`** (comma-separated `api://` nodes) as fallback.
- Availability is probed via `/status`: clearnet returns `true`/`false`; P2P returns the
  provider's signed `pdata` envelope whose `body_b64` base64-decodes to `true`.
- All 4 exchange calls (spot rate, swap quote, buy deposit address, sell gate address)
  now route through `exchange_get`, which auto-switches transport on failure and reuses
  the existing iroh `api_request` P2P transport. Backend-only (`sth_node.rs`);
  `cargo check --features p2p` GREEN.

## [1.11.0] -25

### Added - Settings: P2P protocol node tabs + full-width layout
- **"SMARTHOLDEM NODES" now has two tabs**: **Clear Nodes** (the clearnet mainnet pool)
  and **P2P Nodes** (our protocol `api://` nodes from the new build-time env
  `SMARTHOLDEM_NODES_P2P`). Each tab keeps the AUTO (pick fastest) / RESCAN (ping all)
  actions. Selecting a P2P node routes the wallet through it (reuses `select_node`,
  which already accepts `api://`); clearnet stays as fallback; the choice is persisted
  and P2P nodes are pinged on startup.
- Exposed `SMARTHOLDEM_NODES_P2P` to the frontend via Vite `envPrefix`.

### Changed
- **Settings page is full-width** (removed the 1400px max-width) - no more empty space
  on the right.
- **"P2P UPDATES" moved into the left network column** (above P2P Discovery) as a
  section instead of a separate full-width block.
- `yarn build` GREEN (env var confirmed bundled). Frontend-only; no Rust changes.

## [1.10.0] -25

### Added - Private files: real filename + notify confirmation + full-width layout
- **Real filename shown** for private files (owner and recipient). Transit blob is now
  **SFP2** (`magic | keys | meta_enc | content_enc`) - the name/mime live in a separate
  small AES-GCM section so a recipient recovers the display name cheaply right after a
  download (owner gets it at creation). New `FileManifest.displayName` (local-only,
  never transmitted); UI shows `🔒 <real name>`.
- **"Notify" now opens a confirmation modal** showing the cost (recipients × fee) with
  Confirm/Cancel. Notification fee raised to **1 STH per tx (anti-spam)**.
- **My Files page is full-width** (removed the 1100px max-width) - table uses the whole
  working area.
- NOTE: blob format changed SFP1 → SFP2; private files created on a prior build must be
  re-created. `cargo check --features p2p` GREEN, `yarn build` GREEN.

### Added - Downloaded files: quick "Save as…"
- **Folder icon** before the Link button on downloaded files → opens the native save
  dialog (`save_file_as`). **Right-click** any file row → context menu with
  **"Save as…"**. Private files are decrypted on the backend before the dialog and
  written out in plaintext to the chosen folder (public files copy as-is).
  Frontend-only (reuses existing `save_file_as`); `yarn build` GREEN.

### Added - E2EE Private Files (Phase B: on-chain notifications + inbox)
- **Notify recipients on-chain**: `notify_private_recipients` sends a memo transfer
  to every whitelisted address carrying a compact `sf:<cid>~<node>` link in the
  vendorField (≤255 B, standard fee). "✉ Notify" button on owned private files.
- **Inbox scanner**: `spawn_private_scanner` polls the user's own inbound txs (45 s),
  detects the `sf:` prefix, registers a remote private manifest and raises a
  `private-file-received` event → toast + native OS notification (App.vue), added to
  a new **"🔒 Private"** tab (`list_private_inbox`). Deduped by tx id in Sled.
- **Self-serving key delivery**: private transit blob is now `SFP1 ‖ u32 keys_len ‖
  wrapped_keys_json ‖ sealed_container`. Since each wrapped key is individually ECIES-
  sealed, shipping them in the (public) blob is safe and lets a recipient decrypt
  after a P2P pull without any extra key exchange - the `sf:` notification only
  references the blob. `decrypt_stored` parses SFP1 and reads the caller's key.
- **Recipients list in "My Files"**: owned private files show an expandable list of
  the recipient addresses (from `FileManifest.wrappedKeys`/`recipients`). Access
  counter intentionally omitted (P2P/iroh architecture).
- **Private inbox UX**: click → background streaming download (`start_file_download`)
  → Open (decrypts to a temp file) / Save (decrypts to a chosen path). i18n RU/EN.
  Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN. (Runtime on the
  user's local `yarn tauri:build`.)

### Added - E2EE Private Files (Phase A: encryption, key wrapping, metadata)
- Files can be marked **🔒 Private** on upload with a whitelist of SmartHoldem
  recipient addresses. The file is sealed with **AES-256-GCM**; the random file key
  is **ECIES-wrapped** (ephemeral **secp256k1** ECDH → SHA-256 shared secret → AES-GCM)
  for the owner and every recipient. Wrapped keys are stored per address in the
  `FileManifest` (`wrappedKeys`), so only an authorized wallet can unwrap and decrypt.
- The **real filename/mime is hidden**: it lives only inside the encrypted container
  (`[u32 meta_len][meta json][content]`); the public manifest shows a generic
  "🔒 Private file" name and `application/octet-stream`. Seeders/network never see
  plaintext or the original name.
- Recipient public keys are resolved **in the backend** via the existing SmartHoldem
  node pool (`sth_node::fetch_wallet_pubkey`, reads `SMARTHOLDEM_NODES`), avoiding
  webview CSP/CORS issues. An address with no on-chain public key (never broadcast a
  tx) **blocks the upload** with a clear error.
- Decryption happens on access - `save_file_as`, `open_file_native` and the `f://`
  scheme decrypt and restore the real name for authorized addresses only.
- Rust crypto helpers added in `crypto.rs`: `gen_file_key`, `aes_gcm_encrypt/decrypt`,
  `ecies_wrap_key`, `ecies_unwrap_key`. `add_file` now takes an optional `recipients`
  list; `AddFileModal.vue` gains a Private toggle + recipients textarea (with address
  validation). Verified: `cargo check --features p2p` GREEN, `yarn build` GREEN.
  (Runtime E2EE flow verified by the user on the local `yarn tauri:build`.)

### Fixed
- **File-download progress bar stuck at 0%** until completion. `fetch_file` now spawns
  a concurrent local **`observe`** task that streams the blob bitfield and updates a
  live byte counter (`DL_BYTES`); `blob_progress` reports transferred bytes in real
  time instead of only the lagging store `status()` size.
- **Large-file hashing froze the File Exchange.** `add_file` now hashes and copies
  public files in a **single streaming pass** (4 MiB chunks, never loads the whole
  file into RAM) via a temp file + atomic rename, and **emits `file-add-progress`**
  events. `AddFileModal.vue` shows a live hashing/encryption progress bar for large
  files.
- **Compile blocker:** `torrent::open_torrent_file` was registered in the handler but
  missing its `#[tauri::command]` attribute (added but never compiled last session) -
  attribute added so the crate builds again.

## [client 1.9.3]  (Torrents: open file on double-click)

### Added
- **Double-click a torrent row to open its payload with the system default app.**
  New `torrent::open_torrent_file` command: for a single-file torrent it opens the
  file itself via the OS opener (`shell().open`), for a multi-file torrent it falls
  back to opening the folder. The directory walk runs in `spawn_blocking` (never
  blocks the main thread).

## [client 1.9.2]  (no-block UI: async commands)

### Fixed - UI freeze after download / on offline nodes
- Several Tauri commands were **synchronous** and therefore ran on the main thread,
  freezing the UI (notably over RDP) while doing filesystem walks and sysinfo disk
  enumeration - made worse right after a download grew the local store. Converted the
  UI-polled heavy commands to `async` and offloaded their blocking work with
  `tokio::task::spawn_blocking`:
    - `files::list_my_files`, `files::list_exchange_files`, `files::file_disk_stats`
    - `apps::storage_usage`, `apps::disk_space`

## [client 1.9.1]  (File Exchange: background downloads)

### Added - "Are downloading" section + background P2P downloads
- Clicking download on an `sn://file/...` link now calls a new `start_file_download`
  command that records `downloading: true` on the manifest in Sled and spawns the
  iroh fetch in the BACKGROUND (`tauri::async_runtime::spawn`), so the transfer keeps
  going no matter which page the user navigates to. Returns immediately; payment /
  missing-seeder validation still happens up front. A session dedup guard
  (`ACTIVE_DL`) prevents double-starting the same CID.
- New **"Are downloading"** tab in "My Files" showing in-progress downloads with a live
  progress bar (polls `file_download_progress`). Downloading files are excluded from
  the "Downloaded" tab and included in `list_my_files` even before the blob lands on
  disk. On completion the manifest flips to `downloading: false` + `seeding: true`.
- `FileDownload.vue` now uses the background flow and resolves the page via progress
  polling (`complete` flag) instead of blocking on the full transfer.

### Changed - DHT provider fallback for downloads (fixes "provider returned no data")
- `p2p::fetch_file` no longer depends on the single fixed `seeder_node` from the share
  link. It now aggregates ALL known providers - the link node, every seeder recorded
  for the CID (`file:seeds:<cid>`), and tracker/DHT-known headless seeders from the
  network map - into one `Vec<EndpointId>` and races them via the new
  `download_complete_any` (iroh resolves each provider's live address through its
  DHT/pkarr discovery). An offline `seeder_node` no longer blocks the transfer as long
  as any provider is reachable. Verified working (7.5 MB file transferred between
  clients).

### Note
- Version bumped to 1.9.1 across `package.json`, `Cargo.toml`, `tauri.conf.json`
  (previously stuck at 1.8.21 through the 1.8.22 Tor-rollback and 1.9.0 Stealth work).

## [client 1.9.0]  (Stealth Mode / anti-DPI)

### Added - Stealth Mode (hardcore TSPU/DPI censorship bypass)
New module `src-tauri/src/stealth.rs` + `StealthMode.vue` component (toggle on the
Dashboard, in place of the former block). Three layers:
- **Network relay isolation.** On node startup (after app restart), when stealth is
  enabled and n1 relays are configured, the iroh endpoint is built via
  `Endpoint::empty_builder(RelayMode::Custom(RelayMap))` strictly on our n1 relays
  (`smartnet-relay`), with no default n0 relays and no mDNS → the node advertises no
  direct UDP addresses; EndpointId resolution goes through the serverless Mainline
  DHT (pkarr), peer data travels as QUIC-over-relay (TLS/443). Safe fallback to the
  standard network if relays fail to bind. `apply_smartnet_relays` is skipped under
  stealth (relays are already in the custom map).
- **Application-layer padding.** Every outgoing chat frame is wrapped as
  `[MAGIC | payload_len(u32 BE) | payload | random tail 16..=512 bytes]`; the
  receiver strips the tail by `payload_len`. Legacy-compatible (MAGIC uses bytes
  0x00/0x01 that never occur in a base64/hex ciphertext - old frames read as-is).
- **Chaff (dummy traffic).** A background tokio task (`chat::gossip::start_chaff`)
  broadcasts empty (payload_len=0) padded frames over gossip topics at a random
  1.5–4.5 s interval while idle → a monotone network pattern through the relays. The
  task self-terminates when stealth is turned off.
- **Tauri State + commands.** `NetworkManager` (Arc<Mutex<..>>, short critical
  sections with no .await held) in Tauri State; commands `toggle_stealth_mode(enable,
  custom_relays)` and `get_stealth_status()`. Application layers (padding + chaff)
  activate instantly; relay isolation applies on the next startup (no deadlock risk,
  no dropping of active chats).

## [client 1.8.22]  (Tor rollback)

### Removed - full rollback of the Tor Connect integration
- Completely removed the experimental "Tor Connect" layer (arti-client): the
  `tor.rs` module, `TorConnect.vue` component, the `tor` cargo feature and its deps
  (`arti-client`, `tor-rtcompat`, `thiserror`, `futures`), the `tauri:dev:tor` /
  `tauri:build:tor` npm scripts, the `TOR_ENABLED` / `TOR_SOCKS` globals in `lib.rs`,
  librqbit peer SOCKS5 routing in `torrent.rs`, the bridge bindings, and the
  `docs/TOR_CONNECT_SETUP.md` doc. The codebase is back to its pre-Tor state (build
  with `--features p2p`, no changes to any other functionality).

## [client 1.8.18] - 2026-07-02

### Fixed
- **Published dApp version not propagating (author sees v3, peers see v1).** After
  `update_app_at` / publish / delete, the frontend updated the local manifest but
  never rebuilt the discovery announce, so the node kept advertising the **stale
  version** over mDNS and via the cached `TRACKER_SNAPSHOT` heartbeat - peers only
  saw the new version if/when the author happened to reopen "Find apps". The app
  store now re-runs `startDiscovery()` (rebuilds seeded set + tracker snapshot +
  re-registers the mDNS TXT with the current version/hash/metadata) right after any
  publish, version update, metadata edit or removal, so the new version reaches the
  LAN and trackers immediately.

## [client 1.8.17] - 2026-07-02

### Added
- **Live update progress in the status bar + system console.** The update bar no
  longer sits at a frozen "0%". While a magnet resolves it shows an **indeterminate
  animated bar** and rotating status text - "Connecting to the torrent network…", "Searching
  for source in DHT…", "Waiting for live seeds…", "Requesting metadata…" - with live DHT
  node / peer counts, then switches to "Downloading · seeds N / peers M · ↓ speed" and
  a real percentage once metadata arrives. Backend mirrors every phase to the
  system console under a new **"upd"** scope (check → found/none → download start →
  metadata received → progress every 15 s → 100 % / install), so it's clear why one
  node finds the release instantly and another waits (no live seeders yet).

### Changed
- **Refactor: `chat.rs` → `chat/` module folder.** Split the ~2 k-line file toward a
  future standalone Android messenger build:
    - `chat/mod.rs` (command surface, rooms/groups/policy logic),
    - `chat/crypto.rs` (E2EE key/codec helpers, was `chat_crypto.rs`),
    - `chat/bans.rs` (moderation ban table, was `chat_bans.rs`),
    - `chat/gossip.rs` (Iroh Gossipsub free-message transit, extracted from the inline
      `mod gossip`).
      Compiles clean both with and without `--features p2p`.

## [client 1.8.16] - 2026-07-02

### Added
- **"UPDATE" badge on the release torrent row.** Update-package torrents
  (seeded from `data/updates/v<version>`) are now flagged with `isUpdate` on
  `TorrentStatus` and show a green badge in the Torrents list, so it's obvious the
  client is re-seeding the new SmartNet release to other nodes. Tracked via a new
  `MGR.updates` set, populated on the update download path and on resume when the
  torrent folder is under `/updates/v`.
- **Telegram-style messenger skin.** The P2P messenger now uses a familiar
  Telegram "Night" palette regardless of the app's global theme: dark blue-grey
  panels (`#17212b`), darker chat canvas (`#0e1621`), blue accent (`#64b5ef`),
  solid blue own-message bubbles (`#2b5278`), sans-serif text, rounded input,
  circular blue send button and blue active-chat / unread badges.
- **Emoji picker (Telegram-like).** Added `vue3-emoji-picker`; a smiley button in
  the composer opens a searchable native-emoji panel above the input and inserts
  the chosen emoji at the caret position.

## [client 1.8.15] - 2026-07-02

### Added
- **"Total downloads" stat on the Torrents page.** A new cumulative *total downloaded*
  card sits just before *Total given* (total uploaded). Backed by a new
  `downloaded_bytes` counter in the `profile_stats` Sled tree, accumulated once a
  minute from librqbit's per-session `snapshot.fetched_bytes` using the same
  monotonic-delta scheme as the upload counter (survives restarts, never
  double-counts). Exposed via `get_profile_stats` → `downloadedBytes`.

### Fixed
- **Invisible "Delete torrent" dialog.** The confirmation modal markup existed in
  `Torrents.vue` but had no styles, so triggering a delete did nothing visible.
  Added the full `<style scoped>` block for `.del-overlay` / `.del-card` /
  `.del-check` / `.del-confirm` (dark SmartNet theme, centered overlay, red accent
  when "also delete files" is checked). Deleting torrents works again.
- **Downloaded update didn't reappear / kept seeding after the app restarted into
  the new version.** Other torrents resumed fine, but the update package torrent
  (added from a magnet) was gone after the post-install relaunch, so seeding of the
  new release could not be verified. Two changes make it resume like any normal
  torrent, seeding straight from `data/updates/v<version>`:
    - `resume_persisted` now **prefers the cached `.torrent`** (instant local re-seed
      with the correct info_hash, no DHT metadata round-trip) and only falls back to
      the magnet if no `.torrent` is cached yet.
    - `download_magnet_to` (the updater's download path) now **caches the update's
      `.torrent` as soon as metadata loads** via a background waiter, so a quick
      "Update now" → relaunch (before the periodic 10 s persist tick) still leaves a
      cached `.torrent` for resume.

## [client 1.8.14] - 2026-07-01

### Fixed
- **Torrents list empty after restart (seeding didn't show).** The UI reads from
  `MGR.handles`, which was only populated by re-adding our own Sled records. librqbit
  restores its torrents from its own persistence, so those handles were missing from
  `MGR.handles` and the list looked empty even while seeding continued underneath.
  Added `sync_handles_from_session()` (called in `ensure_session` after
  `resume_persisted`) which enumerates `session.with_torrents(...)` and populates
  `MGR.handles`/`MGR.names` from librqbit's actually-restored torrents. Now all
  resumed downloads/seeds (including the last downloaded update) appear and keep
  seeding after a restart.

## [client 1.8.13] - 2026-07-01

### Fixed
- **Windows installer launch failed ("Failed to open installation package" / "\\").**
  The auto-restart command chained `msiexec` and `start` inside a single
  `cmd /C "…"` string; Rust's argument quoting wrapped the whole string, producing
  broken nested quotes that corrupted the MSI path. Now a temporary `.cmd` script is
  written (paths on separate, individually-quoted lines) and executed - no nested
  quotes. The script waits, silently installs (`/qb`) and relaunches the client.

## [client 1.8.12] - 2026-07-01

### Changed
- **Single-file torrents no longer create a redundant subfolder.** Previously a
  one-file torrent was saved as `downloads/<name>/<file>`. Now single-file torrents
  are written directly into the downloads root (`downloads/<file>`). Multi-file
  torrents still get their own subfolder. File count is captured during
  `parse_magnet` / `parse_torrent_bytes` (`MGR.file_counts`).

## [client 1.8.11] - 2026-07-01

### Fixed
- **Update hangs on the release's own seeder.** When a client already seeds the
  update torrent (e.g. it created the release from a local folder), the updater no
  longer tries to re-download "from itself". `start_update_download` now short-circuits
  via `torrent::is_managed()` and returns the existing info_hash, so the flow
  completes instead of hanging at "Downloading… 0%".

### Added
- **Auto-restart after update (Windows/Linux).** `run_p2p_installer` now launches a
  detached script that waits for the app to close, installs silently, then relaunches
  the client:
    - Windows: `cmd /C timeout & msiexec /i <msi> /qb & start "" <exe>` (CREATE_NO_WINDOW).
    - Linux: `sh -c 'pkexec dpkg -i|snap install …; (<exe> &)'`.
    - macOS `.dmg`: mounts via `open` (manual drag-install; no auto-restart).

## [client 1.8.8] - 2026-07-01

### Added
- **Fully decentralized P2P update system (blockchain-anchored + torrent transport).**
  No central servers / GitHub. Source of truth for versions is the SmartHoldem (STH)
  blockchain; file transport is BitTorrent (librqbit).
    - **Backend (`updater.rs`):** scans STH transactions to `SaQUdQbxZGJdjzZpLPyDYAqtE47EupDaTe`,
      accepts only sender `SeZLuyhhYf2qxs4ArPJ71oEu3x8EsVw51C`, parses vendorField
      `smartnet:<version>:<btih_hash>`, semver-compares against `CARGO_PKG_VERSION`.
      Commands: `check_for_update`, `start_update_download` (magnet built from btih →
      downloads into `data/updates/v<ver>/`, keeps seeding after 100%),
      `read_update_manifest` (recursive `manifest.json` lookup), `resolve_installer_path`
      (recursive filename lookup), `cleanup_update_dirs`, `run_p2p_installer`,
      `get_update_period_hours`, `get_client_version`, `current_os`.
      Startup cleanup keeps only the current running version folder.
    - **Installer launch:** Windows `msiexec /i <path> /qb`; Linux `pkexec dpkg -i`
      (.deb) / `pkexec snap install --dangerous` (.snap); macOS `open` (.dmg);
      Android JNI-Intent stub. `std::process::exit(0)` after launch.
    - **Frontend:** `updater.ts` service (first check 5 min after start, then every
      `CHECK_UPDATE_PERIOD` hours - default 48), `UpdateBar.vue` (bottom status line:
      "Downloading… N%" + persistent "Update" button), `UpdateModal.vue`
      ("New version downloaded! [Update now] [Later]"). Torrent helper
      `torrent::download_magnet_update` downloads a magnet into an exact folder.
    - `.env`: `CHECK_UPDATE_PERIOD=48`.
    - `manifest.notes` may be a filename (e.g. `CHANGELOG.md`) - its file content is
      loaded (≤40 KB) and shown as the release notes text in the update modal.
    - **Settings → Network → "P2P-UPDATES"**: manual "Check for updates now"
      button (calls `updater.checkNow()`) + current client version display, for
      instant testing without waiting for the 5-min timer.
    - Robust installer resolution (`resolve_installer`): resolves the OS installer
      by manifest key first, then falls back to extension match (.msi/.deb/.snap/.dmg/.apk).
      On failure it returns a diagnostic error listing all files in the package, shown
      in the update modal.

## [client 1.8.7] - 2026-07-01

### Added
- **"Private torrent" checkbox** in the create-torrent dialog. When enabled,
  `info.private = true` is set on the generated torrent (not shared over DHT).
  Since `private` lives inside the info-dict, the info-hash is recomputed from the
  final serialized bytes via `librqbit::torrent_from_bytes` (no extra deps).

### Fixed
- **Build error `E0433: cannot find crate urlencoding`** in `torrent.rs` (`magnet_of`
  helper). Replaced the `urlencoding::encode` call with a dependency-free inline
  percent-encoder (`pct_encode`) for the magnet display-name.

## [client 1.8.6] - 2026-07-01

### Added
- **Create a new torrent from a file or folder.** A new "Create Torrent" button
  on the Torrents page (left of the settings gear) opens a qBittorrent-style
  creation dialog: pick a file or folder, choose the piece size (Auto / 64 KiB –
  16 MiB), edit the tracker list (pre-filled from our cached public trackers
  synced from GitHub `ngosang/trackerslist`) and add an optional comment.
  On "Create Torrent" the `.torrent` is generated via `librqbit::create_torrent`,
  saved into our torrents cache (`data/torrents/xx/<hash>.torrent`) and the source
  content is immediately put up for seeding in the Torrents list (no save dialog).
    - Backend: `create_torrent_from_path` and `get_bt_trackers` commands in
      `torrent.rs` (helpers `create_and_seed` / `public_trackers`), registered in
      `lib.rs`. Frontend: `TorrentCreateModal.vue` + IPC bindings in `bridge.ts`.

### Fixed
- **Non-p2p build break.** `install_app` used `tokio::fs::read` without the `p2p`
  feature gate (tokio is only linked under `p2p`), breaking `cargo check` on the
  default feature set. Switched to blocking `std::fs::read`.

## [client 1.8.5] - 2026-07-01

### Added
- **Right-click context menu on torrent rows.** Right-clicking any torrent in the
  Torrents list opens a floating menu with two actions: **"Copy magnet link"**
  (copies the magnet URI to the clipboard) and **"Export to .torrent"**
  (opens the native "Save As" dialog to export the `.torrent` file).
  Backend commands `get_torrent_magnet` and `get_torrent_file_bytes` (thin wrappers
  over the existing `magnet_of` / `torrent_file_b64` helpers) were added to
  `torrent.rs` and registered in `lib.rs`; frontend IPC bindings added to `bridge.ts`.

## [client 1.8.4] - 2026-07-01

### Added
- **Peak upload speed + leecher count on each torrent card.** Each torrent now
  shows its session peak upload speed ("peak ▲ …", tracked client-side) and the
  peers column is relabeled **Seeders/Leechers** - the connected-peer count
  (`peer_stats.live`) represents the leechers actually downloading from us when
  seeding, making "who we're uploading to" visible at a glance.

### Changed
- **Network Map globe: fits the window, wheel-zoom, 3:2 frame.** The planet is now
  auto-fit to the render block (with padding) regardless of window size via a
  `fitCamera()` that computes camera distance from the frame aspect, so it never
  clips. The frame is a 3:2 landscape rectangle (wider than tall). Added
  **mouse-wheel zoom** over the globe (0.6×–3× around the auto-fit distance).

## [client 1.8.3] - 2026-07-01

### Fixed
- **Torrents page showed "0 B uploaded" although seeding actually works.** The
  Torrents "total uploaded" stat summed each torrent's **per-session**
  `uploaded_bytes`, which librqbit resets to 0 when a torrent is re-added on
  restart - so after a restart it read 0 even though the node had really uploaded
  data (the Network Map "My Node" card correctly showed e.g. 12.2 MB from the
  cumulative `profile_stats`). The Torrents stat now uses the cumulative uploaded
  from `profile_stats` (matches the Network Map) and falls back to the session sum,
  and is relabeled "Total given". Seeding was never broken - 0 live peers on a
  heavily-seeded popular torrent just means few leechers currently need our data.
- **Network Map globe: planet now fills the block width (no empty side areas).**
  Moved the camera closer (z 285 → 232) so the planet fills the render frame, and
  the globe host spans the full block width (capped at 440px) instead of a small
  centered square that left gaps on the sides.

## [client 1.8.2] - 2026-07-01

### Fixed
- **UI froze (and process became unkillable) when pausing a large torrent / after
  restart while downloading.** The `pause`/`resume`/`remove` commands are
  synchronous Tauri commands (run on the **main UI thread**) and each did
  `MGR.rt.block_on(session.pause/unpause/delete)` - for a multi-GB torrent that
  disk-state flush blocked the whole UI (Windows even reported the process as
  not-responding / access-denied to kill). Also `get_active_torrents` (polled)
  called `ensure_session`, which `block_on`-ed session creation + network on the
  first poll after restart, freezing the UI. Now: pause/resume/remove **spawn the
  work on the torrent runtime and return instantly** (state persisted to Sled
  immediately), and `get_active_torrents` never inits the session (a background
  `resume_on_startup` thread does). No torrent operation blocks the main thread.

### Added
- **Exact paused-state restore (1:1 like qBittorrent).** The Sled record now
  stores `paused`; resume re-adds torrents with `AddTorrentOptions.paused`, so a
  paused torrent stays paused after restart instead of resuming.
- **Missing-folder detection → Error status.** On startup/resume, each torrent's
  download folder is checked on disk; if it was deleted/moved externally, the
  torrent is not re-seeded and is shown with the **Error** status (Error tab)
  instead of silently failing.
- **Single-instance guard.** Added `tauri-plugin-single-instance` - launching a
  second copy now focuses the existing window instead of starting a duplicate
  process.

### Note
- Partial-file seeding ("seeding until full download") already works - librqbit
  uploads the pieces you already have to peers while still downloading (confirmed
  live: a 100% torrent seeding + an in-progress download at 16 MB/s).

## [client 1.8.1] - 2026-07-01

### Fixed
- **Torrents disappeared after restart (persistence/resume).** The librqbit
  session was created lazily (only on the first torrent action), so
  `resume_persisted` never ran at launch - persisted torrents only reappeared
  after the user added another torrent. Now the session is initialised and
  persisted torrents (downloading / paused / completed, with their save path)
  are resumed on app startup via `resume_on_startup` (background thread), and
  `get_active_torrents` also ensures the session as a safety net.

### Added
- **Per-torrent connection-type indicator (TCP / µTP).** The torrent list now
  shows live TCP and µTP connection counts per torrent (`peer_stats.live_tcp` /
  `live_utp` from librqbit 9), visually confirming µTP is actually working.
- **Torrent settings gear + tabbed settings modal.** Added a settings gear button
  next to the DHT-nodes counter on the Torrents page. The settings modal is now
  organised into tabs - **Connection** (peer protocol TCP/µTP, incoming port +
  random-port toggle & reroll, UPnP, max peers/torrent), **Speed** (down/up
  rate limits), **BitTorrent** (DHT on/off, LSD on/off, RuTracker key). New
  `dht`/`lsd` settings persist to Sled and apply on restart (`SessionOptions.dht`
  / `disable_local_service_discovery`).

### Changed
- **Network Map globe made shorter.** The 3D globe block is now capped/centered
  (max 320px square) instead of stretching to the full leaderboard height.

## [client 1.8.0] - 2026-07-01

### Changed (BREAKING: requires local rebuild)
- **Upgraded librqbit 8.1.1 → 9.0.0-rc.0 and enabled µTP - the real fix for
  "downloads don't start / 0 peers".** librqbit 8.1.1 was **TCP-only** (no µTP at
  all), so peers behind CGNAT/NAT - the majority of RU residential seeders -
  were unreachable, while qBittorrent (TCP + µTP) downloaded the same torrents
  fine on the same machine. The BitTorrent session now uses librqbit 9's
  `ListenerOptions { mode: TcpAndUtp }`, so the client connects over both TCP and
  µTP like qBittorrent. Also enabled `fastresume`. API migration in `torrent.rs`:
  `SessionOptions.listen/connect/peer_limit`, `dht.with_routing_tables()`
  (replaces removed `clone_routing_table`), DHT node count now sums IPv4+IPv6
  routing tables. `Cargo.lock` re-pinned to the librqbit 9 crate set.

### Added
- **Torrent connection settings** (in the torrent-client settings modal):
  incoming port with a "Random port" toggle + reroll button, µTP on/off,
  UPnP on/off, and per-torrent peer limit. Persisted to Sled (`__net`) and applied
  on next app start. New Tauri commands `get_torrent_net_settings` /
  `set_torrent_net_settings`.

### Note
- This is a **beta/rc** librqbit release. Build locally with
  `cargo build --features p2p` (Tauri GUI can't run in the CI pod). The librqbit
  9.0 API usage was validated by compiling an isolated probe against the exact
  crate version.

## [client 1.7.9] - 2026-07-01

### Fixed
- **Torrents stuck at 0 %, no peers (the real root cause).** The librqbit session
  was created without `listen_port_range`, so librqbit **never opened a TCP
  listener** (`tcp_listen_port = None`). Consequences: (1) no incoming peer
  connections, (2) tracker announces were sent **without a port**, so private,
  tracker-only torrents (e.g. RuTracker, which forbid DHT/PEX) got **0 peers**,
  and (3) UPnP port-forwarding was never activated. qBittorrent always listens on
  a port - which is exactly why it worked on the same machine/network while our
  client hung. Fixed by configuring `listen_port_range: 6881..6889` and
  `enable_upnp_port_forwarding: true` in `SessionOptions`. (Supersedes the earlier
  incorrect "ISP/DPI blocking" diagnosis - it was a client-side config bug.)

## [client 1.7.8] - 2026-07-01

### Fixed
- **Torrent add modal: "Start" button lost with long file lists.** With torrents
  containing many files (e.g. 53), the file list grew unbounded and pushed the
  settings column (save path + Start button) out of the clipped modal. The body
  grid now uses `grid-template-rows: minmax(0, 1fr)` and `flex: 1 1 auto`, so the
  file list scrolls internally while the settings column (and Start button) stay
  fully visible; the settings column can also scroll on very short screens.
- **Torrent modal no longer closes on backdrop click.** Removed the
  `@click.self` outside-click handler - the modal now closes only via the ✕
  button (top-right), preventing accidental dismissal.

## [client 1.7.7 · tracker 1.1.1] - 2026-07-01

### Added
- **Byte-Hours reported to the tracker → ranked on the public /map leaderboard.**
  The client's signed 5-minute heartbeat now carries an unsigned `byteHours`
  field (accumulated torrent Byte-Hours in GiB·h, from `profile_stats`). The
  tracker reads it (`Heartbeat.byteHours`) and uses it as the home node's
  leaderboard **score**, so your node now competes directly with headless
  seeders (whose score = GB seeded × uptime hours) in the "GB·h" column of the
  public `GET /map` leaderboard and `GET /seeders`. When no Byte-Hours are
  reported yet, the previous tiny uptime-based score is kept so home nodes never
  outrank GB-backed servers with zero contribution. `byteHours` is kept out of
  the signed digest (same self-report trust model as `seededGb`/`storageLimitGb`),
  so older trackers simply ignore it.

### Note
- The tracker is a separately deployed binary - rebuild/redeploy `smartnet-tracker`
  (1.1.1) for the leaderboard side; the client change is backward-compatible with
  older trackers.

## [client 1.7.6] - 2026-07-01

### Added
- **Byte-Hours economy + node rank on the Network Map.** A background worker
  (torrent runtime, once per minute) integrates seeded volume over time:
  `byte_hours += Σ(total_bytes of Seeding torrents) × (1 min / 60)`, persisted in
  a dedicated Sled tree `profile_stats` (key `current`) together with cumulative
  uploaded bytes. New Tauri command `get_profile_stats` returns Byte-Hours
  (GiB·h), node rank, uploaded bytes and progress to the next rank. Ranks by
  GiB·h: **Leech** (<1) → **Peer** (1–100) → **Seeder** (100–1000) →
  **Anchor** (>1000). The Network Map now shows a "My Node" card with the
  Byte-Hours counter, a coloured rank badge and a progress bar to the next rank
  (ru/en i18n). This ties torrent seeding to gamification, nudging users to seed
  longer.

## [client 1.7.5] - 2026-07-01

### Fixed
- **Torrent live stats (progress / speeds / peers) not displaying.** `status_from`
  navigated the serialized `TorrentStats` JSON and read `live.download_speed.mbps`
  through a `u64` accessor - truncating the fractional part (any speed < 1 MiB/s
  became 0) and then multiplied by `125_000` (Mbit→byte conversion instead of
  MiB→byte). Rewrote extraction to use the **typed** `librqbit::TorrentStats`
  fields directly: `state` maps via `TorrentStatsState`, download/upload speeds
  come from `live.download_speed.mbps` / `live.upload_speed.mbps`
  (MiB/s → bytes/s = `× 1_048_576`, fraction preserved), and connected/seen peers
  from `live.snapshot.peer_stats.{live,seen}`. This was most visible with
  `.torrent` files (metadata known up front) that hung at 0 % with no speed.

### Changed
- **Per-torrent download sub-folder.** librqbit only auto-creates the `<name>/`
  sub-folder when `output_folder` is `None`; the client always passed an explicit
  folder, so every torrent's files were dumped straight into `data/downloads/`.
  Now each torrent downloads into its own `<base>/<sanitized-torrent-name>/`
  folder (works for single- and multi-file torrents; base = user-chosen path or
  `data/downloads`). The folder name is sanitized for path traversal / illegal
  Windows characters.
- **Readable torrent folder persisted in Sled.** The `torrents` tree record now
  stores the full torrent folder path in both `save_path` and a dedicated
  `folder` field for UI/debugging; "Open folder" resolves it directly.

## [client 1.7.4] - 2026-07-01

### Changed
- **Resilient tracker-list update.** The public tracker refresh now tries several
  mirrors (GitHub raw → jsdelivr CDN → GitHub Pages). If all sources are
  unavailable/blocked, the timestamp is left untouched (retried next launch) and
  the client keeps using the trackers already stored in DB (`cfg:bt_trackers`),
  falling back to the built-in list only when the DB is empty.

## [client 1.7.3] - 2026-07-01

### Fixed
- **Empty torrent info-hash (critical).** `Id20` serializes via `serialize_bytes`,
  so `serde_json::to_value(info_hash)` produced a byte array, not a hex string -
  the parsed info-hash came back **empty**. This broke handle/DB keys and saved
  the cached torrent as a bare `.torrent` file. Now uses `Id20::as_string()`
  (hex, upper-cased). Torrents are correctly cached at
  `data/torrents/{first-2-hash-chars}/{HASH}.torrent`
  (e.g. `data/torrents/69/69A9C93074A08B3F16B62245DF5157A1EE28EE9C.torrent`).
  The real info-hash is always available from parsing, so no topic-number
  fallback is needed.

## [client 1.7.2] - 2026-07-01

### Added
- **`RUTRACKER_BT` env var.** The RuTracker keeper key can now be provided via
  `frontend/.env` (`RUTRACKER_BT=...`, exposed through Vite `envPrefix`). On
  startup `main.ts` pushes it to the backend (`set_rutracker_key`), so RuTracker
  announces (`http://bt{,2,3,4}.rutracker.cx|t-ru.org/ann?kk=<key>`) are attached
  to torrents automatically. Still overridable in the settings modal.

## [client 1.7.1] - 2026-07-01

### Fixed
- **RuTracker keeper key format** corrected from the real keeper `.torrent`
  source: the passkey parameter is `kk` (not `bt`) and announce hosts include the
  `rutracker.cx` mirrors, e.g. `http://bt4.rutracker.cx/ann?kk=<key>`.

## [client 1.7.0] - 2026-07-01

### Added
- **Public tracker list (peer/metadata discovery speed-up).** Fixes magnets that
  hung for minutes when relying on DHT only (e.g. a single private tracker).
    - Built-in fallback list of ~15 popular public UDP/HTTP trackers.
    - Auto-refresh from `ngosang/trackerslist` (`trackers_best.txt`) at most once
      every 5 days, cached in DB (`cfg:bt_trackers` + `cfg:bt_trackers_ts`), fetched
      in the background via `ureq`.
    - Trackers injected into every `AddTorrentOptions` (magnet list-only, torrent
      file, download start, resume) so metadata resolves fast like qBittorrent.
- **RuTracker keeper key (kk passkey).** Configurable in the torrent settings
  modal ("Torrent Client Settings"). When set, private RuTracker announce URLs
  (`http://bt{,2,3,4}.t-ru.org/ann?kk=<key>` and the `rutracker.cx` mirrors) are
  added to the tracker set for peer discovery on private releases. Commands
  `get_rutracker_key` / `set_rutracker_key`; stored in DB (`cfg:rutracker_bt`).

## [client 1.6.0] - 2026-07-01

### Added
- **Reverse DHT synergy (torrent → SmartNet).** A background task (every ~10s)
  harvests librqbit's live DHT routing-table nodes
  (`Session::get_dht().clone_routing_table()`) and merges them into our
  anti-censorship bootstrap cache (`p2p::merge_dht_hints`), strengthening the
  SmartNet Mainline DHT from the torrent client's peers.
- **Drag & drop .torrent files.** A full-window drop overlay ("Drop to
  add .torrent") appears while dragging (Tauri `tauri://drag-*` events);
  dropping a `.torrent` parses it and opens the same metadata modal as magnets.
  New backend commands `parse_torrent_file` (bytes) and `parse_torrent_path`
  (path). A drop hint is also shown next to the "+ Add" button.
- **`.torrent` file cache.** Torrents are cached at
  `data/torrents/{first-2-hash-chars}/{hash}.torrent` - saved for dropped files
  immediately and for magnets once metadata resolves (speeds up resume).
- **Native librqbit session persistence** (`SessionPersistenceConfig::Json`)
  under `data/session` for fast-resume across restarts.

### Fixed
- **Removing a torrent now deletes its files.** The remove action asks for
  confirmation and calls `remove_torrent(delete_files=true)` (previously partial
  downloads stayed on disk).
- **"Open folder" opened Documents instead of the torrent folder.** `folder_of`
  now returns the effective `<output_folder>/<name>` (absolute), and the OS-open
  helper normalizes path separators (forward slashes broke `explorer.exe`) and
  ensures the directory exists. Downloads now default to a concrete stored path.
- **Torrents page width.** Removed the `max-width: 1200px` cap so the page fills
  the content area (no right-side gap).

## [client 1.5.1] - 2026-07-01

### Added
- **Native torrent DHT node counter.** New `get_torrent_dht_nodes` command
  returns librqbit's own BitTorrent DHT routing-table size
  (`Session::get_dht().stats().routing_table_size`) - the exact "DHT: N nodes"
  metric qBittorrent shows. The Torrents page header badge now reads this
  torrent-client DHT (polled every 5s) instead of the SmartNet Mainline DHT.

### Notes
- Verified librqbit 8.1.1 does **not** expose custom DHT bootstrap-node
  injection via `SessionOptions` (only `disable_dht` / `disable_dht_persistence`
  / persistence `dht_config`; `DhtConfig.bootstrap_addrs` is unreachable from the
  Session builder, and `Dht` has no public `add_node`/`bootstrap`). Pushing our
  SmartNet bootstrap nodes into librqbit's DHT is therefore not possible without
  patching the crate. The reverse synergy (harvesting librqbit's live DHT nodes
  into our anti-censorship cache via `Session::get_dht().clone_routing_table()`)
  remains feasible as a future option.

## [client 1.5.0] - 2026-07-01

### Added
- **Torrents page - speed limits & filters.**
    - Global speed-limit modal (`components/TorrentLimitsModal.vue`, qBittorrent-style
      sliders + numeric inputs, 0 = unlimited) opened by clicking the "Download" /
      "Upload" stat cards. Backend `get_torrent_limits` / `set_torrent_limits`
      persist KiB/s in the `torrents` Sled tree and apply live via
      `Session.ratelimits.set_download_bps/set_upload_bps` (also re-applied on
      session start).
    - "DHT Nodes: N" live counter in the page header (reuses the existing Mainline
      DHT stats, polled every 5s).
    - Record filters above the table (radio group): All / Downloading / Done /
      Paused / Error, each with a live count.

## [client 1.4.0] - 2026-07-01

### Added
- **Embedded BitTorrent client - Phase A (Rust core).** New module
  `src-tauri/src/torrent.rs` built on top of `librqbit` 8.1.1 (pure-Rust async
  stack on Tokio, built-in DHT, disk IO for terabyte-scale files). Gated behind
  the `p2p` feature (desktop build).
    - Global thread-safe `TorrentManager`: dedicated multi-thread Tokio runtime +
      `librqbit::Session` + handle/name/magnet maps.
    - Commands: `parse_magnet_link` (list-only metadata fetch: name, size, file
      tree), `start_torrent_download` (background download, default path
      `data/downloads`, `only_files` for partial/episode selection),
      `get_active_torrents` (status snapshot), `pause_torrent`, `resume_torrent`,
      `remove_torrent`, `open_torrent_folder`.
    - `torrent-progress` Tauri event - pushes `Vec<TorrentStatus>` once per second.
    - Metadata persisted in a **dedicated Sled tree** `torrents` (resume on
      restart). Seeding continues after completion; sequential flag stored per
      record for streaming.
    - `librqbit` compiled with `default-features=false, features=["rust-tls"]`
      (no OpenSSL - cross-platform build).
- **Embedded BitTorrent client - Phase B (Vue 3 UI).**
    - Sidebar entry "TORRENTS" + `/torrents` route (`views/Torrents.vue`).
    - Page: aggregate stats (download / upload / uploaded-this-session), compact
      custom table (name, size, progress bar, status, seeders/peers, speed,
      pause/folder/remove buttons), "+ Add magnet" input.
    - Magnet capture modal (`components/TorrentMagnetModal.vue`): network metadata
      load via `parse_magnet_link`, file tree with checkboxes (episode selection →
      `only_files`), save-path field (default `data/downloads`), "Sequential
      download (for streaming)" checkbox, "Start" button.
    - Address-bar `magnet:?` interception (on Enter and auto-on-paste) → opens the
      Torrents page with the modal; reactive `route.query.magnet` watch so it works
      even when the page is already open.
    - `bridge.ts` wrappers: `parseMagnetLink`, `startTorrentDownload` (+`onlyFiles`),
      `getActiveTorrents`, `pause/resume/removeTorrent`, `openTorrentFolder`, and
      `onTorrentProgress` subscription (progress bars refresh reactively 1×/sec).

### Changed
- **`DEFAULT_SEEDERS`** now ships all three canonical headless seeders
  (`x.x.x.x:6885`, `x.x.x.x:6881`, `y.y.y.y:6881`) → the
  Network Map "SEEDER" badge reflects the full default set.

### Fixed
- **3D globe rendering**: data spikes shortened by 15% (`alt × 0.85`), the globe
  container is now square (`aspect-ratio: 1/1`, no more "egg" distortion), and
  the planet is scaled to 90% inside the render square for better framing.
- **Network Map client visibility**: client (non-headless) nodes are now colored
  distinctly in blue (`#60a5fa`) with a dedicated "CLIENT" legend entry, so they
  are no longer indistinguishable from WAN servers.

### Build
- **Committed `Cargo.lock`** (previously untracked) pinning `time` to 0.3.49 for
  reproducible builds - fixes the transitive `cookie 0.18.1` vs `time ≥ 0.3.51`
  `Parsable::parse` arity break on fresh checkouts.

## [client 1.3.0] 2026-06-30

### Added
- **Configurable DHT seeders** (multiple supported, no longer a single hardcoded
  host). Resolution: Network Settings list (`cfg:seeders`) → auto-discovered
  (`cfg:seeders_auto`) → `SMARTNET_SEEDERS` env → built-in default; the DHT build
  paths use the deduped union. New `get_seeders`/`set_seeders`/`get_seeders_auto`
  commands and a Settings UI section (ru/en/id) mirroring WAN trackers.
- **Zero-config seeder auto-discovery (`GET /seeders-dht`).** SmartNet seeders
  advertise their public DHT bootstrap addr (`seederDht`) in `/announce`; the
  tracker pools the fresh ones (≤30 min) and serves them, the client harvests them
  every tracker tick into `cfg:seeders_auto` - so users never need to add seeders
  manually. Network Settings shows a read-only **"+N auto-discovered"** badge.
- **Anti-censorship DHT bootstrap exchange (tracker + gossip).** Nodes with a
  healthy DHT share ≤8 live node addrs; censored nodes (blocked from public
  routers/seeder, DHT=0, but reachable via tracker/mesh) harvest them and
  **re-bootstrap on the fly**. Tracker channel: `/announce` `dht[]` → pooled (cap
  256, 1h TTL, validated) → `GET /dht`. Gossip fallback: well-known
  `SMARTNET-DHT-HINTS-BOOTSTRAP-v01` topic. Hardened: `merge_dht_hints` is
  rate-limited (10s), socket-addr-validated, capped (≤8/batch, ≤64 cache), deduped.
- **Operator-signed seeder list - TOFU + backward-compatible (P2 anti-poisoning).**
  Tracker publishes its operator pubkey at `GET /operator-pubkey` and in `/stats`
  (`pubkey`) only when `OPERATOR_SECRET` is set; signs `/seeders-dht` (secp256k1
  ECDSA). Clients **auto-pin** the key on first connect (TOFU) and verify
  signatures, rejecting forged lists; `SMARTNET_OPERATOR_PUBKEY` hard-overrides.
  Trackers without a key omit the field ⇒ clients stay in unsigned mode (old
  trackers keep working unchanged).
- **Floating System Console** (topbar terminal icon, Tauri only): live `sys-log`
  Tauri event stream with a 300-line ring buffer (`syslog.rs`), backlog via
  `sys_log_history`. Scope-filter chips (DHT/P2P/NET/SYS - click toggle,
  double-click "only", "ALL"), **Copy-all**, grep text filter; ~1080px wide.
  Log points: boot, endpoint bind, DHT bootstrap/table changes, tx rejections,
  seeder/key pinning.
- **DHT node-count mini-sparkline** next to the counter on Dashboard + Network Map
  (`Sparkline.vue`), backed by a rolling localStorage history (`dhtHistory.ts`).
- **"SEEDER" badge on the Network Map** - bootstrap-source seeder nodes are shown
  in violet on the globe with a legend entry and live count, distinct from
  ephemeral amber DHT peers.
- **Client version in the window title bar** (set at runtime from `CARGO_PKG_VERSION`).
- **`frontend/.env.example`** documenting every supported frontend param plus the
  backend `SMARTNET_SEEDERS` / `SMARTNET_DHT_BOOTSTRAP` / operator-key runtime envs.

### Fixed
- **Crash on "Publish key" in the Messenger** (`assertion failed: is_char_boundary`).
  `parse_sth` truncated the fee/amount fractional part with `String::truncate(8)`,
  panicking on a multi-byte UTF-8 char. The parser now keeps **only ASCII digits** -
  char-boundary-safe and robust against any malformed input.
- **"Transaction rejected by node" when publishing the messenger key.** The fee used
  was the generic `/node/fees` min (~0.001 STH), but a key publish carries a
  ~100-byte `xkey:` vendorField, so the node's size-based dynamic minimum (~0.011
  STH) was undershot → `ERR_LOW_FEE`. Publishes (key, msg-cost, endpoint re-publish)
  now compute a **size-aware dynamic transfer fee**, the banner previews the real
  fee, and the node's rejection reason is surfaced + logged to the console.
- **Network Map UI freeze (~10s) on open.** `NodePlanet` built the hex-polygon globe
  synchronously at `hexPolygonResolution(3)` over an 838 KB GeoJSON. Now resolution
  **2** + the heavy `init()` is deferred two animation frames so the view paints first.
- **Network Map central content** now spans **100% width** (removed the 1400px cap).
- **DHT node count inconsistent / flashing 0 between Dashboard and Network Map.**
  The monitor stored `routing_table_size` unconditionally and never persisted it, so
  Mainline's "breathing" table caused transient 0s and every cold start began at 0.
  Now: restores the last-known count (`cfg:dht_nodes`) + peer snapshot at startup;
  **last-known-good** (only stores/persists when `> 0`); both views share a
  `sth_dht_stats` localStorage cache and refuse to display a transient 0.
- **Outdated P2P-discovery hint** ("Blob transit (iroh) is the next step") removed -
  dApp/blob transfer over iroh-blobs already works (ru/en/id).

## [tracker 1.1.0] 2026-06-30

### Added
- `GET /dht` - pooled Mainline DHT bootstrap nodes (fed by `/announce` `dht[]`,
  capped 256, 1h TTL, socket-addr-validated) for anti-censorship bootstrap.
- `GET /seeders-dht` - DHT bootstrap addrs of fresh SmartNet seeders (≤30 min) for
  zero-config client auto-discovery; ECDSA-signed when `OPERATOR_SECRET` is set.
- `GET /operator-pubkey` + `pubkey` in `/stats` - operator signing pubkey for client
  TOFU pinning (empty/omitted when no key configured ⇒ unsigned mode).
- `/announce` accepts `dht[]` (peer DHT hints) and `seederDht` (a seeder's own DHT
  bootstrap addr).

## [seeder 1.2.0] 2026-06-30

### Added
- Advertises its public DHT bootstrap addr (`seederDht` = metrics host + `dht_port`)
  in `/announce`, so trackers can auto-distribute it via `GET /seeders-dht`.

## [client 1.2.4] 2026-06-30

### Fixed
- **REGRESSION: "DHT nodes = 0" on the client.** The DHT monitor had switched to
  `DhtBuilder::bootstrap(&pre_resolved_ips)`, which **replaces** the default public
  Mainline routers. The pre-resolved list often collapsed to just the (not-yet-
  deployed / unreachable) SmartNet seeder IP, leaving the DHT with no reachable
  bootstrap targets → it never joined the network → `routing_table_size` = 0.
  Fix: the monitor now uses `extra_bootstrap` (KEEPS the reliable default public
  routers and ADDS the SmartNet seed + cached nodes on top), and runs the blocking
  `build()` inside `tokio::task::spawn_blocking` so the synchronous DNS never stalls
  the UI thread. Verified `seeder/examples/dht_probe` reproduction: routing table 107–131 nodes (was 0).

## [seeder 1.1.2] 2026-06-30

### Fixed
- **Seeder DHT bootstrap node "Address already in use" (os error 98).** The
  seeder created two DHT instances in one process: the implicit `DhtAddressLookup`
  node (which n0-mainline binds to the default Mainline port 6881) and the new
  explicit server-mode bootstrap node also on `dht_port` (6881) → EADDRINUSE, so
  the bootstrap node never started. Fix: run a **single** DHT node - the explicit
  server-mode bootstrap node on `dht_port` (also backs `/metrics` telemetry); the
  redundant `DhtAddressLookup` node was removed.

## [client 1.2.3 / seeder 1.1.1] 2026-06-30

### Added
- **Seeder `/metrics` DHT telemetry**: the seeder now reports `dhtBootstrapServer`
  (bool), `dhtPort`, and `dhtNodes` (routing-table size, polled every 12s) in the
  `/metrics` JSON and on the HTML status page (`dht bootstrap: ON :6881 · N nodes`).
  A dedicated server-mode `Dht` handle backs the telemetry; the address-lookup
  node (pkarr publish) runs separately in client mode.
- **Network Map bootstrap-source indicator**: the DHT stat now shows whether the
  DHT came up **via SmartNet** (green - the SmartNet seeder IP is present in the
  local routing table) or via **public DHT** (grey). Backed by a new
  `via_smartnet` flag on `dht_stats`.

## [client 1.2.2 / provider 0.4.3 / seeder 1.1.0] 2026-06-30

### Added
- **SmartNet public DHT bootstrap node** baked into the headless **seeder**
  (`smartnet-seeder` 1.1.0). The seeder now runs its pkarr/Mainline address-lookup
  node in **server mode** on a fixed UDP port (`dht_port`, default `6881`), making
  it a reachable bootstrap node so clients/providers behind NAT or with closed UDP
  egress can join the DHT through SmartNet infrastructure - independent of n0/n1
  and public BitTorrent routers. New seeder config: `dht_enabled`, `dht_port`.
  The address `x.x.x.x:6881` is baked into the **client** and **provider**
  DHT bootstrap defaults; the client also honours env `SMARTNET_DHT_BOOTSTRAP`.
- **DHT nodes on the Network Map globe**: live DHT routing-table nodes (IP:port)
  are now plotted on the globe (amber **DHT** points) alongside peers, with a new
  legend entry. New client command `dht_nodes_list` + `DHT_PEERS` snapshot updated
  every 12s from `Dht::to_bootstrap()`. `NodePlanet` gained a `maxPoints` prop
  (raised to 72 on the map).

### Fixed
- **Blocking-call audit (no-freeze hardening)**: converted blocking `std::fs`
  calls on the P2P startup/discovery hot path to async - `ensure()` now uses
  `tokio::fs` for `create_dir_all` and the `node.key` read/write, and `install_app`
  reads `content.json` via `tokio::fs`. Confirms the main thread is never blocked
  during discovery (the earlier sync-DNS freeze fix remains in place).

## [client 1.2.1 / provider 0.4.2] 2026-06-30

### Fixed
- **CRITICAL: UI freeze ("Not responding") during peer/app discovery.** Root cause:
  `n0-mainline` resolves the default DHT bootstrap hostnames **synchronously**
  (`std` `to_socket_addrs`) inside `DhtBuilder::build()`. When DNS to those hosts
  is slow/blocked, the call blocked the thread for the full DNS timeout (~10-12s),
  freezing the app. Fix: bootstrap hostnames are now resolved **asynchronously up
  front** (`tokio::net::lookup_host`, concurrent, 1.5s/host timeout) and passed to
  the DHT as **pre-resolved IP literals** via `DhtBuilder::bootstrap(..)`, so
  `build()` performs no blocking DNS. Applied to both the client (endpoint
  address-lookup + monitor) and `netfory-provider`. Discovery is now fully
  non-blocking - the main thread never stalls even when no nodes are reachable.
- **Installed apps grid**: switched `auto-fill` → `auto-fit` so dApp cards stretch
  across the full page width (no empty trailing columns on the right).

## [provider 0.4.1] 2026-06-30

### Added
- **Configurable DHT bootstrap** (`network.dht_bootstrap`, host:port list) for
  `netfory-provider`, merged with the cached `dht_boot.dat` and passed via
  `DhtBuilder::extra_bootstrap`. Lets operators point the Mainline DHT at
  reachable nodes when the default public routers are blocked.

### Fixed / Changed
- **Quieter DHT logs**: the repeating `n0_mainline::core` "Could not bootstrap
  the routing table" ERROR spam is silenced in the default `EnvFilter`
  (`n0_mainline::core=off`, `n0_mainline::actor=warn`). A single clear WARN is
  emitted once if the DHT fails to bootstrap, explaining that outbound UDP must
  be open (or `dht_bootstrap` set).
- **Privacy note in relay-only**: when both relay-only and DHT are enabled the
  provider now warns that participating in the Mainline DHT exposes the server
  IP to other DHT nodes (clients still only get the relay URL).

## [client 1.2.0 / provider 0.4.0] 2026-06-30

### Added
- **Network Map redesign**: the Network Map leaderboard is now a 65% table +
  35% interactive globe (reused `NodePlanet`). All known peers are plotted on
  the globe and coloured by transport source. A new **DHT nodes** stat card and
  a per-peer **source badge** (DIRECT / RELAY / mDNS / WAN) are shown. New
  `peer_sources` command derives transport from `Endpoint::remote_info`
  (direct/relay) + local mDNS set; truthfully, iroh 1.0 does not expose per-peer
  DHT/n0-DNS provenance, so DHT autonomy is surfaced as the network-wide node
  counter rather than a per-peer label.
- **Mainline DHT (pkarr) discovery - serverless node discovery.** Both the
  Tauri client and `netfory-provider` now publish/resolve their NodeID over the
  BitTorrent **Mainline DHT** via pkarr (`iroh-mainline-address-lookup` 0.4 +
  `n0-mainline` 0.5), in addition to the n0 DNS path. This removes the hard
  dependency on n0/n1 servers for finding peers - relays remain only as the
  transport fallback behind NAT (the hybrid model). DHT publishing uses the
  default `relay_only` address filter, so the real IP is never leaked to the DHT.
    - **Client**: `Endpoint::builder` gains `DhtAddressLookup` when enabled; a
      serverless toggle **"Mainline DHT"** sits at the top of Settings → Network
      (default ON, applies on next discovery start). A lightweight monitoring DHT
      node feeds a live **DHT node counter** into the Dashboard mesh-meter.
      New commands: `get_dht_enabled`, `set_dht_enabled`, `dht_stats`.
    - **Provider**: new `network.dht_enabled` config key (default `true`);
      publishes its NodeID to the DHT so clients resolve it without n0/n1. The
      `/status` JSON and HTML dashboard show a new **"DHT Nodes (Mainline)"** card.
- **DHT bootstrap cache (extra_bootstrap).** Both sides periodically snapshot
  live DHT routing-table nodes via `Dht::to_bootstrap()` and persist them
  (provider → `dht_boot.dat`; client → sled key `cfg:dht_boot`, max 64 nodes,
  saved every ~2 min). On the next start these cached nodes are fed back through
  `DhtBuilder::extra_bootstrap`, so the node rejoins the DHT in seconds even when
  the public bootstrap servers are unreachable - strengthening network autonomy.
- **Active client connections counter** in `netfory-provider`: new
  `active_connections` (AtomicU64) in `Stats` tracks how many clients are
  currently being served (distinct from `active_peers`, which counts other
  providers in the mesh). Incremented via a RAII `ConnGuard` at the start of
  `ApiProtocol::accept` and reliably decremented on drop (covers panics/cancels).
  Exposed in `/status` JSON and shown as the "Active clients" card on the
  HTML dashboard.

### Notes
- Client bumped **1.1.5 → 1.2.0**; provider bumped **0.3.0 → 0.4.0**.
- DHT is a native (Tauri) feature - runtime verified by compilation only in the
  cloud preview; full WAN/firewall verification requires the desktop build.
- **Unified node access layer (P2P/HTTPS)** in `sth_node.rs`: wallet & blockchain
  REST calls now route transparently over the iroh P2P mesh when an `api://`
  node is selected, or over HTTPS otherwise. New `node_get`/`node_post` helpers
  branch on the active node; P2P calls are dispatched onto the Tauri/iroh async
  runtime via a channel (`run_async`) so the blocking REST client can await them
  without a nested-runtime panic. `p2p::api_request` generalizes `api_fetch` to
  support GET/POST + request bodies (broadcast over P2P). Indexers (dns_manager,
  naming) also migrated to `node_get`.
- **Custom nodes in Settings → Network**: users can add their own SmartHoldem
  endpoints - clearnet (`node6.smartholdem.io`) or P2P (`api://<nodeId>/<provider>`).
  New backend commands `add_custom_node` / `remove_custom_node` / `list_custom_nodes`
  (persisted in sled `cfg:custom_nodes`); `select_node`/`set_active_node` accept
  custom + `api://` addresses; the background node selector no longer overrides a
  manually-pinned custom node. New `nodes.*` i18n keys (ru/en/id) + UI list with
  select/remove and a P2P badge.
- **Indonesian (id) localization**: full `src/i18n/id.ts` locale file; `id` registered in
  the i18n instance, `LocaleId` type, and both language switchers (sidebar + Settings).
  Users can now switch the entire UI to Bahasa Indonesia.
- **Messenger fully internationalized**: `ChatWidget.vue` previously had hardcoded Russian
  strings only; now wired to `useI18n` with a complete `messenger.*` namespace (ru/en/id) -
  titlebar, rooms, conversation, channel/group settings, moderation, create-group, settings
  modal and all toasts/errors.
- **File Exchange internationalized**: `FileExchange.vue`, `MyFiles.vue`, `FileDownload.vue`
  and `FileQrModal.vue` moved off hardcoded Russian to a shared `fx.*` namespace (ru/en/id).
- **User Profile "My Domains" internationalized**: domain section labels + subscribe error
  toast moved to `profile.*` keys (ru/en/id).

### Changed (earlier)
- **Node Pool moved into Settings → Network**: the "SmartHoldem Nodes" list is
  no longer a standalone sidebar item; it now sits beside P2P Discovery in a
  50/50 split inside Settings → Network. The `nodes` sidebar entry was removed
  (route `/nodes` + `NodePool.vue` kept as a fallback). New i18n `nodes.*` keys.

### Added
- **Search bar in My Applications**: a name filter at the top of My Applications
  filters both Published and Drafts lists live (`searchQuery` + computed
  `filteredPublished`/`filteredDrafts`), with a contextual "nothing found" empty
  state. New i18n keys `apps.searchPlaceholder` / `apps.searchEmpty`.

### Added (earlier)
- **App deletion progress overlay**: clicking confirm-delete in My Applications
  now shows an "Deleting…" spinner overlay on the app card while the (multi
  -second) blob-reclaim runs, instead of the card silently freezing then vanishing.
- **"My Domains" on the profile**: a profile section lists the identity's live
  `.sth` domains. Visible to the owner always; to visitors only when the owner
  ticks **"Show to everyone"** (persisted as `showDomains` in the profile manifest).
  Each domain is a clickable link that opens its bound dApp in a new app tab
  (`tabs.open(appId)`). New backend command `domains_by_owner(address)`.
- **Connection-quality dot on the globe icon**: a small coloured dot
  (green/amber/orange/grey by latency) mirrors the active node's ping, plus the
  node host + latency in the globe tooltip - so users see link quality without
  opening the Node Pool. Backed by a cached `node_status` command (no extra
  network call; updated by the pool pinger).


### Added (SmartHoldem node pool + failover - resilience)
- **Multi-node pool** instead of a single hard-coded node. Default list (7 mainnet
  nodes) is overridable via `SMARTHOLDEM_NODES` (comma-separated) in
  `frontend/.env`. `STH_API` const replaced by a dynamic `api_base()` → the
  currently selected node.
- **Startup ping + best-node selection**: on launch the pool is pinged in
  parallel and the client connects to the lowest-latency online node; re-evaluated
  every 10 min so an overloaded node is replaced automatically.
- **One-retry failover** for HOT reads (balance / subscribers / history): `http_get`
  tries the active node then ONE backup with a short 4 s timeout - never hammers
  the whole pool (public nodes IP-ban over-polling; block time is ~8 s).
- **Node Pool UI** (`/nodes`, new "STH NODES" sidebar item): live list with
  ping/height/online status, "↻ Rescan", "⚡ Auto (best)", and click-to-pin a
  node. Commands: `refresh_node_pool`, `auto_select_node`, `select_node`,
  `current_node`.

### Fixed (P0)
- **My Files row layout was crooked**: cramped columns wrapped text (size
  "121.1 KB" split, "Stop Seeding" on two lines, "· N seeders" dropped below). Fixed
  with `white-space: nowrap` on table cells + `.mini` buttons, `flex-wrap: nowrap`
  on the actions group, ellipsis-truncated CID, and the name column made the
  single flexible one (`width: 99%`). ⚠️ Tauri-gated table - validated by code
  review (the file table renders only under the desktop runtime, not web preview).
- **DomainsMarket auto-refresh**: a 20 s poll re-reads the (local) market index so
  a seller's listing disappears as soon as the chain indexer marks it sold - no
  manual refresh. Pauses when the tab is hidden / busy.
- **"Replace files" update modal overlap**: the update-fee modal is now
  `<Teleport>`-ed to `<body>`, so it renders above the sidebar (a transformed
  ancestor was scoping its `position: fixed`).


### Fixed (UI freeze on poor network - CRITICAL)
- **Root cause:** several `#[tauri::command]`s were *synchronous* and performed
  blocking 12 s `ureq` HTTP calls. Sync commands run on the **main/UI thread**, so
  on a slow network the whole window froze ("Not responding") for ~12 s, then
  recovered. The worst offender was `refresh_subscribers`, auto-called **every
  30 s** by the wallet store → constant freezing on a weak link.
- `subs::refresh_subscribers` and `subs::follow_stats` are now `async` and run
  their blocking chain scans via `tauri::async_runtime::spawn_blocking`, so the
  UI thread never blocks. (`following_addresses` uses a sync inner helper.)
- Audited every command: all other network-touching commands (wallet balance,
  donations, exchange, swaps, transfers, chat BAN-tx, `subscribe`) were already
  `async` + `spawn_blocking`; background watchers run on their own threads.

### Changed (globe / node-telemetry trigger)
- Bigger borderless planet icon (17→26 px, no square button frame).
- Metallic **shine sweep** on hover + subtle drop-shadow glow and rotate/scale.
- **Live stats tooltip** (dApp seeds + online peers + hint) on the icon.
- **Neon flicker (~3.5 s)** when the active dApp gains a new storage peer
  (`activeSeeds` increase watcher).


### Changed
- **File Exchange identifier → native Blake3.** Files are now hashed with Blake3
  (matching the app/merkle hashing used across SmartNet and the iroh-blobs
  transit hash) instead of IPFS CIDv0. The IPFS CIDv0 is still computed and kept
  in a new `ipfsCid` manifest field, reserved ONLY for proof-of-ownership /
  future IPFS-network interop - not for P2P transit or as the local identifier.
- **"To the market" restricted to paid files.** Free files no longer show the
  on-chain marketplace publish button (a commercial listing is meaningless for a
  free file). Paid files keep "To the market" (Type 0 memo publish) until published.

### Added
- **Self-contained share links for free files (Private/Anonymous P2P mode).**
  New "🔗 Link" button in "My Files" copies `sn://file/<cid>~<blob>~<node>`.
  Any peer can open it and fetch the file directly P2P with NO on-chain
  publication. Paid files copy the plain `sn://file/<cid>` (resolved via the
  chain registry after publishing).
- `register_shared_file(cid, blob, node, name, size)` Tauri command + bridge
  binding: the recipient persists a remote manifest from the link payload so the
  download page can fetch the blob. `FileDownload.vue` parses the `~blob~node`
  suffix and auto-registers before download.
- **Live seeder count** on every file (`seeds` populated by `list_my_files`,
  `list_exchange_files`, `file_meta`) - shown in "My Files" and the exchange grid.
- **Unavailable files hidden in the exchange.** The File Exchange catalog now
  filters out files with 0 live seeders (undownloadable).
- **Download progress bar + live seeder count** on the download page, driven by a
  new `file_download_progress(cid)` command (reads the iroh blob-store status →
  bytes transferred / total) polled every 0.7 s while fetching.
- **QR code for share links.** New reusable `FileQrModal.vue` (⊞ QR button in
  My Files, the exchange grid, and the download page) renders a scannable QR of
  the `sn://file/...` link for cross-device sharing.

### Notes
- File seeding is intentionally NOT announced to public trackers: doing so for
  free/Private-mode files would leak the CID and the fact that the node hosts it,
  defeating the anonymous mode. Discovery relies on the chain registry (public
  files) or the self-contained link (private files). WAN multi-seeder tracker
  announce is deferred to Stage 4 (mesh).

### Changed (share-link refinements)
- **Self-contained links carry FULL metadata** (both free AND paid):
  `sn://file/<cid>~<node>~<size>~<price>~<owner>~<name>`. Fixes paid links that
  previously copied a bare `sn://file/<cid>` - the recipient saw the file as a
  nameless "FREE" hash. Now name/extension, size, price and pay-to owner are all
  shown immediately, with no wait for on-chain indexing. The iroh transit hash
  equals the Blake3 cid, so it is not repeated.
- **Sales counter for paid files** + **total revenue** (`price × sales`) shown in
  "My Files".

### Added (My Files UX)
- **Two tabs in "My Files": "My Files" / "Downloaded".** Owned vs downloaded
  (non-owned, locally held) files are now separated.
- Downloaded files get **delete** (`delete_downloaded_file`), **copy link**, **QR**
  and a recorded **download date/time** (`downloadedAt`, persisted on fetch).
- Owned published files show their **publication date/time** (`publishedAt`).
- **File-type icon badge**: documents (txt/md/doc/pdf/…) green, images cyan,
  video magenta, audio amber, archives grey (`FileTypeIcon.vue`, also in the
  exchange grid).
- **Double-click a file name → open with the OS default app** (player/viewer/…).
  `open_file_native` exports a cached temp copy with the real name (the store
  path is the bare CID) so the OS resolves the correct application.
- Fixed the duplicated "seed" label in the status column.

### Planned
- Decide metadata layer: **Entity (Product) transactions** (typeGroup 2,
  supported on-chain) carry a `data` field (name + ipfsData CID) and support
  `Update` for versioning - the proper home for name/price/description, since
  Type 5 IPFS cannot. Pending product decision.
- Stage 3: paid-access flow (`filebuy:<cid>:<buyerId>` payment verification).
- Stage 4: mesh storage metrics + DHT announce/discovery.

## [1.1.4] 2026-06-24

### Verified
- **LIVE on-chain validation of Type 5 (IPFS):** broadcast a real CID stamp to
  SmartHoldem mainnet (tx `7a0f8f91…`, confirmed in block) for **~0.012 STH**
  via the dynamic fee - the native Rust signing is proven correct.

### Changed / Fixed
- **Type 5 IPFS is CID-only.** Discovered (via the live node) that IPFS
  transactions have `hasVendorField() == false`: the node forces the vendorField
  byte to `0x00` on re-serialization, so a non-empty vendorField breaks
  signature verification. Removed vendorField from the IPFS serializer/JSON,
  added the required `amount: "0"` field, and switched the publish fee from the
  5 STH static fee to the **minimum dynamic fee (~0.012 STH)**. On-chain we now
  record only the CID (integrity + timestamped authorship); name/price stay in
  the local manifest.

### Added
- **`FILE_EXCHANGE_PAGE_ALLOW` gate** (frontend/.env, comma-separated addresses):
  when set, the global "File Exchange" browse tab is shown only to listed
  addresses; everyone else gets just "My Files" (their own files + stats, plus
  files received via shared `f://` links). Unset → visible to all.

## [1.1.3] 2026-06-24

### Added
- **File Exchange v2.0 - Stage 2: on-chain publish + indexer.**
    - Native Rust signing/broadcast of a SmartHoldem **Type 5 (IPFS)** transaction
      (`sth_node.rs`: `serialize_ipfs` / `build_signed_ipfs` / `publish_ipfs`).
      Asset = raw base58-decoded CIDv0 multihash (no length prefix), matching the
      `@smartholdem/crypto` serializer exactly; file metadata (`name`/`price`/
      `type`) rides in the vendorField. Static fee 5 STH; legacy bip-schnorr
      signature (same scheme as transfers).
    - `publish_file` command + a **Publish** button / on-chain badge in My Files.
    - Background **chain indexer** (`files::spawn_indexer`): scans Type 5 IPFS
      transactions and registers discovered files (remote stubs) so other users'
      published files surface in the Exchange.

## [1.1.2] 2026-06-24

### Fixed
- **File Exchange `f://` open:** files now open in a dedicated native child
  window via the registered `f://` scheme (`open_file_webview`) instead of
  `window.open`, which the webview blocked as a local resource and the address
  bar mangled into `sth://f://…`. The `f://` host is normalized for the
  Windows `<cid>.f.localhost` mapping.
- **Delete file:** replaced the native `confirm()` (blocked inside the Tauri
  webview - "dialog.confirm not allowed") with an in-app confirmation modal.

## [1.1.1] 2026-06-24

### Added
- **SmartNet File Exchange (v2.0) - Stage 1:** decentralized file sharing
  foundation.
    - New `files.rs` backend module: real IPFS **CIDv0** generation
      (`base58btc(0x12 0x20 || sha256)`), hierarchical content store at
      `data/filesdata/<XX>/<CID>`, and a Sled registry
      (`files:cid:<cid>` + `user:files:<address>`).
    - Tauri commands: `add_file`, `list_my_files`, `list_exchange_files`,
      `file_meta`, `set_file_seeding`, `delete_file`, `file_disk_stats`, `pick_file`.
    - iroh-blobs seeding of added files (best-effort), GC-protected alongside dApps.
    - `f://<cid>` custom URI scheme with a free/owner access gate
      (paid-access verification arrives in Stage 3).
    - UI: **File Exchange** (card grid + New/Paid/Free sorting, reactive refresh)
      and **My Files** (disk stats + Delete/Stop-Seeding + Add File modal) pages,
      with `sn://files` / `sn://myfiles` aliases.
- Tauri window now launches **maximized** (fills the screen on startup) while
  keeping window chrome and resizability.
- Instant orphan-blob reclamation: deleting a dApp (or stopping file seeding) now
  refreshes the iroh GC keep-set from the remaining apps, so the next GC pass
  reclaims the orphaned content blob. Dedup-safe (shared blobs stay protected)
  and a no-op when the P2P node isn't running.


## [1.1.0] 

### Added
- **Storage proofs on the 3D globe:** connected peers are colored green when they
  verifiably hold the dApp blob (lightweight iroh-blobs `observe`), with a
  verified / partial / unverified legend.
- **Cold-boot sync splash** (`SyncSplash.vue` + `sync_status.rs`): shows
  "Synchronizing with network…" during the first blockchain scan to avoid a
  frozen-feeling UI.
- **Cross-app content deduplication (P3):** `dedup:index` registry maps content
  hashes to app ids so byte-identical dApps share a single physical blob.
- Locally bundled fonts via `@fontsource` (full offline support).

## [1.0.x] - dDNS & Marketplace (Phases 1–3c)

### Added
- **Decentralized `.sth` Name Service (dDNS):** domain registration, renewal,
  and a **Domains Market** (`DomainsMarket.vue`) with sorting and PREMIUM badges.
- **Atomic buy** flow (3 sequential, indexer-validated transactions) and a
  post-purchase "path of the hero" UX that auto-mints a cyberpunk draft dApp for
  the bought domain.
- Draft-app lockdown (no domain bind/buy until published) with a "Publish app →"
  CTA.

### Fixed
- `atomic_buy` memo parsing (a `buy:` tx has 4 parts, not 5).
- Shortened domain-purchase memo (`buydom:<name>:<zone>`) with backward
  compatibility.

## [1.0.0] - Core MVP

### Added
- Vue 3 + Tauri v2 desktop client: stateless thin-client for the SmartNet
  ecosystem.
- Built-in **wallet** (native Rust signing of SmartHoldem v2 transfers).
- **E2EE P2P chat** (X25519 + AES-256-GCM) over iroh Gossip, with groups,
  moderation, bans, and paid messaging.
- **dApp Store / publish pipeline:** drag-and-drop folder ingest, Blake3 merkle
  signing, content-addressed `sth://<id>` rendering via a custom URI scheme.
- **3D Earth telemetry** (`NodeTelemetryPanel` + `NodePlanet`) with GeoLite2 peer
  geolocation.
- On-chain **u:// username** registry, profiles, paid subscriptions, and a
  "Earn STH" background seeding mode.

[Unreleased]: https://github.com/smartholdem
[1.1.1]: https://github.com/smartholdem
[1.1.0]: https://github.com/smartholdem
