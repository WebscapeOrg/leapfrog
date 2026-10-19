# Leapfrog

Leapfrog is the local integration layer around upstream T1Bridge. It owns the
Omarchy Touch Bar renderer, the FaceTime-camera compatibility relay, the PAM
configuration used for Touch ID, and the theme gallery used to design the
renderer.

- Do not vendor or modify upstream T1Bridge. Keep it as a package dependency.
- Never commit Apple firmware, T1 keybags, enrolled fingerprints, crash dumps,
  generated theme thumbnails, or machine credentials.
- `install.sh` must remain user-space only. Root-owned files belong under
  `system/` and are applied only by the explicit recovery-shell installer.
- Keep the Touch Bar's black background, white text, and 6:1 minimum text
  contrast unless Dave explicitly changes those rules.
- Run `script/check` before committing.

## Asking Dave for decisions: Settlers of Questan

Before you start work in a repository, read `~/.local/share/pickle/agent-instructions.md`
and follow it. It is how you ask Dave for decisions, and it says where his earlier answers are
recorded. Do not copy its contents into this file or any other file: it is kept in one place so
that a change to it reaches every agent.
