# Changelog

## 0.2.9 — 2026-11-02

- The checksum manifest must name each patch file exactly once. Matching a
  whole line is satisfied by any matching line, so a manifest listing a
  candidate twice — the right hash and a truncated file's hash, which is what
  appending `sha256sum` output to the file produces — let the truncated patch
  install itself and delete the backup. Only an unchecked manifest edit could
  reach it, since `script/check` fails such a file, but it was reachable.
- `script/check` runs its corrupt-candidate case against an unguarded tree as
  well as a guarded one, so the forward probe is covered there and not only
  the reverse one.
- Corrects 0.2.7: the claim that `script/check` refuses a guard letting patch's
  own words reach stdout past a probe held for the reverse probe only, because
  that case built a guarded tree and nothing else.

## 0.2.8 — 2026-11-02

- Each circuit-breaker patch file is checksummed before it is used: its whole
  `sha256sum` line must appear in `lock/patch-checksums.txt`, which also refuses
  a candidate the manifest does not list. Handing the manifest to `sha256sum
  --check` is not enough — without `--strict` it skips a malformed line with
  only a warning and still exits 0, so a hash shortened by one character let a
  truncated patch install itself. No structural test can catch a patch file cut
  short at a hunk boundary: such a file applies cleanly and reverses cleanly,
  so the plugin is left part-patched while every probe reports success — the
  first three hunks of the v1.7 patch applied twice read "changed" and then
  "current" on a tree that was never whole. The cost is deliberate: editing a
  patch file means updating the checksum file in the same change, and
  `script/check` fails when the two disagree.

## 0.2.7 — 2026-11-02

- The lock guard no longer trusts the marker alone. `fingerprintFastFailures`
  is added by hunk 1, so a tree where that hunk applied and the other five
  did not carried the marker while the circuit breaker was not wired up, and
  the guard called it "current" — leaving the 250 ms PAM retry loop in place.
  Being whole is now proved by reverse-dry-running the candidates, and a
  part-patched tree stops the run and lists the hunks that are missing, since
  no further forward apply would repair it.
- Both probes triage patch's exit codes the same way. Reading every non-zero
  exit as an answer about the tree meant a failure of patch ITSELF — an
  unusable temporary directory, a truncated patch file — turned a wholly
  guarded plugin into "only partly applied" and failed the ship sync. A
  patch that cannot be used is now loud and names the file, whether the tree
  is guarded or not; the two cases used to disagree.
- The apply no longer leaves a backup. `--no-backup-if-mismatch` stops patch
  writing `Service.qml.orig` beside the live plugin, which its default did on
  every run because the hunks land at large offsets, and a leftover from an
  earlier run is removed — including a dangling symlink, and without the
  tidy-up ever being able to stop the guard. That removal is below the lock
  gate on purpose, because a write to the live plugin directory while the
  session is locked has crashed a shell before, and below the manifest check,
  so nothing is deleted in a directory the run then refuses.
- The apply's own output is captured and shown only when it fails. A
  successful apply can still be chatty — a read-only `Service.qml` in a
  writable directory draws "trying to patch anyway" — and that line in the
  ship sync's note is the same false alarm 0.2.6 was released to stop.
- The lock gate can no longer be walked past. Without `omarchy-shell` the
  lock state cannot be proved, and an unprovable state is now treated as
  locked rather than as safe; both paths are normalised before they are
  compared, so a trailing slash on the live directory no longer makes the
  live tree look like an inert staging copy.
- Smaller: a plugin that is already guarded on a machine missing one of the
  patch files now fails rather than reporting "current", because the files
  are checked first; and an unreadable `Service.qml` is reported instead of
  leaking grep's own error.
- A refused tree keeps its backup. The tidy-up runs only once the run has
  something true to report — a tree proved whole, or an apply that succeeded —
  because deleting the backup while rejecting a part-patched plugin takes the
  file away exactly when someone needs it. A failure to remove it is reported
  on stderr, so a successful run's stdout stays the single line the sync reads.
- `script/check` now also refuses to pass a guard that accepts a part-patched
  tree, one that accepts a MARKER-ONLY tree (the only case that enforces
  `--forward` on the reverse probe; a hunk-1-only tree is refused either way),
  one that accepts an unusable candidate patch instead of refusing it, one that
  does not defer when `omarchy-shell` is absent or the session is locked — read
  as the output being exactly the deferred line, the service file's hash
  unchanged and a planted backup still present, because a guard that prints
  "deferred" and then patches anyway passes a weaker test — one that lets
  patch's own words reach stdout past a probe, and one that prints anything beyond the single
  expected line on a clean apply or on `current`. Its lock cases say plainly
  when they are skipped, rather than passing in silence. The lock-gate case runs
  under a fake `HOME`, so the real plugin cannot be reached by any build.
- Smaller still: `--plugin-dir` is refused when it names nothing, and `realpath`
  is given `--`, so neither an empty nor a dash-leading path leaks realpath's
  own error; and a
  patch that cannot be used says so without blaming the patch file, since an
  unusable temporary directory lands there too.

## 0.2.6 — 2026-11-02

- Stopped the lock guard's patch probe reaching the caller. `lock/install`
  picks between the pre-v1.7 and v1.7 circuit-breaker patches by dry-running
  each, and the first one fails a hunk on a current plugin, which is the
  normal path. patch printed `1 out of 6 hunks FAILED` on stdout, and the
  ship sync folds that stream into its note, so a run that applied the v1.7
  patch and exited 0 read as a failed lock guard. Each probe's output is now
  held back and shown, named by candidate and with every failed hunk and its
  line, only when no candidate fits — the case where knowing what moved
  upstream is the point. The success line names the patch it applied.
- The guard no longer skips a candidate it could not use. patch's exit codes
  are triaged rather than merged: hunks that do not fit stay quiet, while a
  corrupt or truncated patch file stops the run and says so. A missing,
  empty or unreadable candidate is refused outright, which also closes a
  standing hole where an empty patch file dry-ran clean, was selected, and
  the script reported a change it had not made.
- `script/check` now also runs the guard against an unguarded stage. The
  copy it made of the live plugin is already guarded, so the run took the
  "current" early exit and never reached the probes, which is how the leaked
  line went unnoticed.

## 0.2.5 — 2026-10-01

- Replaced the workspace tile with a printscreen button between the
  volume-up and Fn buttons, leaving thirteen equal-width buttons centred as
  one row. The scissors-icon button opens the same Omasnap screenshot flow
  as the keyboard's SUPER SHIFT S and Print keys: it asks Hyprland to exec
  the script, which works from the renderer's boot-time environment where
  `WAYLAND_DISPLAY` is never set. Removed the workspace reading that only
  the tile used.

## 0.2.4 — 2026-09-18

- Fixed the Touch Bar's workspace tile reading DESKTOP after every boot. The
  renderer starts with the user's systemd manager, before Hyprland exists, so
  it never inherited the variable `hyprctl` uses to find the compositor. It
  now asks `hyprctl` for the first running Hyprland instance on every
  refresh, so the tile shows the workspace within a second of Hyprland
  starting and keeps working after a compositor restart.

## 0.2.3 — 2026-09-13

- Replaced the Touch Bar's return-to-stock button with keyboard backlight
  controls. They sit just right of the workspace tile, which keeps that tile
  centred between six actions on either side, and their icons follow the
  illumination keys on Apple keyboards: three rays over a dashed bar, short
  for dimmer and long for brighter. Each press moves the keyboard in 10%
  steps, through T1Bridge when it offers the keyboard backlight and through
  `brightnessctl` otherwise. The stock renderer is still one `leapfrog stock`
  away.
- Button groups now alternate between the theme's selection and accent fills
  by position rather than by a fixed colour per control, so neighbouring
  groups always stand apart. The Fn row alternates the same way.

## 0.2.2 — 2026-09-13

- Kept the Touch Bar renderer alive through a cold boot. The T1 hardware can
  take half a minute to answer after the launcher starts the renderer, and a
  single refused connection made the renderer exit, which left T1Bridge on its
  built-in bar until the next manual restart. The renderer now waits up to 90
  seconds for the hardware and reconnects for the rest of the session, so a
  suspend or a hardware hiccup no longer costs the Omarchy bar either.

## 0.2.1 — 2026-09-10

- Added an inert staging-tree mode for the main machine, so the exact marketplace
  lock plugin is guarded before it is copied to the MacBook.

## 0.2.0 — 2026-09-10

- Prevented an unavailable T1 keybag service from causing an unlimited
  lock-screen fingerprint retry loop. Three immediate PAM failures now pause
  Touch ID for that lock session while password unlock remains available.

## 0.1.0 — 2026-09-09

- Named and collected the T1Bridge integration as Leapfrog.
- Added the Omarchy-themed Touch Bar renderer, FaceTime camera relay, Touch ID
  PAM configuration, theme gallery, installers, and status command.
