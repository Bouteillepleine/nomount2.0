# Changelog

## v2.1.0

First release of the reworked hybrid metamodule.

### Mount
- **Mountless VFS redirection** — direct-path module files load at stock system
  paths via the `/dev/nomount` driver, with zero `/proc/mounts` entries.
- **Hybrid RRO overlay support** — module `**/overlay/*.apk` dirs are mounted as
  a real `overlayfs` (staged on tmpfs, because `/data` f2fs `casefold` is
  rejected by overlayfs as a lowerdir) so Android's `idmap2` / `OverlayManager`
  pipeline can enable them. Without this, RRO overlays stay `STATE_NO_IDMAP` and
  theming (e.g. OxygenCustomizer) breaks. Everything else stays mountless.
- **Self-mounting module blocklist** — skip modules that manage their own path
  redirection (built-in list + `/data/adb/nomount/blocklist`).

### Detection hiding (own footprint)
- Overlay mounts are registered with KernelSU's native umount
  (`kernel_umount` + `umount-config`) so they're `MNT_DETACH`ed inside DenyList
  apps' namespaces.
- `/dev/nomount` is hidden from non-root scanners via SUSFS `sus_path`.
- **Per-app UID isolation** — block specific UIDs so the VFS hook returns
  pristine stock for them.

### Manager & WebUI
- **Per-module manager tags** — each module's description is tagged with how
  it's served (`vfs` / `overlay` / `vfs + overlay`).
- **WebUI** — engine status/toggle, remount, bootloop-guard status + re-arm,
  modules list, active rules viewer, overlay-mounts list, and UID exclusions.

### Safety
- **Bootloop guard** — a boot counter self-disables NoMount after repeated
  failed boots and re-arms once the system boots healthy.
- **Install-time sha256 integrity check** — every bundled file is verified
  against a manifest at install; a corrupt or tampered zip aborts.

### Kernel
- Kernel patches for android12-5.10, android13-5.15, android14-6.1,
  android15-6.6, android16-6.12 (raw GKI + SUSFS-compatible variants). The
  recursion guard uses `current->journal_info` — never `android_oem_data1`,
  which OEMs like OnePlus use for their own per-task pointer (writing to it
  soft-locks the device at boot).
