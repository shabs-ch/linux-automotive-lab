# Day 0 – Tracing and reverting a custom WSL kernel (17 Oct 2026)

## Finding
- `uname -r` showed `6.18.40.1-microsoft-standard-WSL2+`, while `wsl --version`
  reported Microsoft's shipped kernel `6.18.54.1`.
- `/proc/version` showed the builder `shabn@Shabs` and Ubuntu's gcc 13.3.0, built 1 Aug 2026:
  a custom kernel compiled on my PC by a tool installer.
- `.wslconfig` contained `kernel=C:\\Users\\shabn\\wslkernel\\bzImage`.

## Risk
- 14 stable releases behind Microsoft's kernel, so likely missing security fixes.
- In WSL 2 one kernel is shared by all distributions, so it affected my Ubuntu as well.

## Change (reversible)
- Commented out the `kernel=` line, then ran `wsl --shutdown`.
- Kept `bzImage` on disk so the change can be undone.

## Verification
- `uname -r` → `6.18.54.1-microsoft-standard-WSL2` (no `+`)
- `/proc/version` → built by Microsoft's build system, 6 Oct 2026

## What I learned
- Ubuntu kernel doesn't run in Windows. Windows has it's own linux kernel. My previous attempt with ADDM was also running only a customized version of windows kernel for linux.
