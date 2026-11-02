# Changelog

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
