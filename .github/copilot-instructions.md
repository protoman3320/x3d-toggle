# Copilot Instructions for x3d-toggle

## Architecture & Big Picture
- **Service Boundaries:** The project is split into a high-performance C backend (`x3d-toggle.c`) and a Bash-based heuristic daemon (`x3d-daemon`). 
- **The C Binary:** Acts as a secure, fast proxy to write to the kernel sysfs node (`/sys/devices/platform/AMDI*/amd_x3d_mode`). It also handles performant CPU utilization polling (`x3d-toggle check-load`).
- **The Daemon:** Runs as a systemd user service (`x3d-auto.service`). It evaluates gaming intent (via Steam, gamemode, desktop files) or compute load, then invokes the C binary via `pkexec`.
- **Terminology:** 
  - 🐰 **Rabbit Mode:** `cache` (prioritizes 3D V-Cache for gaming).
  - 🐆 **Cheetah Mode:** `frequency` (prioritizes high clocks for compute tasks).
  - 🦌 **Elk Mode:** `auto` (returns scheduling to the default kernel driver).

## Security & Permissions
- Never alter the permissions strategy. Hardware writes require root privileges. 
- The project **strictly uses PolicyKit** (`org.x3dtoggle.policy`) for privilege elevation. Do not implement `setuid` bits or raw `sudo` commands inside the user-facing GUI/daemon. Always use `pkexec /usr/bin/x3d-toggle` when invoking from user-space scripts.
- The C binary enforces safety by checking `geteuid() == 0` securely.

## Developer Workflows & Build Commands
- **Standard Build:** Run `make` to compile the C binary, or `sudo make install` to test global installation on Debian/Ubuntu targets.
- **Arch Linux Workflow:** For full integration testing (including `.desktop` generation, pacman hooks, and dependencies), use `makepkg -sri`.
- **Testing the Binary:** Once built, test manual switching via `sudo ./x3d-toggle [cache|frequency|auto|get]`. Note that without compatible AMD 3D V-Cache hardware, the sysfs node will not be present.

## Coding Conventions
- **C Code (`x3d-toggle.c`):** Use minimal standard libraries (`stdio.h`, `glob.h`, `unistd.h`). Avoid introducing heavyweight dependencies (like glib) to keep latency near zero. Optimize file reads when parsing `/proc/stat`.
- **Bash Scripts (`x3d-daemon`, `PKGBUILD`):** Maintain bash script safety. Use `shellcheck` directives for ignores where necessary (e.g., `# shellcheck disable=SC...`). Check for required binaries existing (e.g., `command -v notify-send`) before execution to gracefully degrade features.
- **GUI:** The GUI is implemented via `kdialog` (`x3d-toggle-gui`). Keep dialogs simple and native to KDE/Plasma conventions, but functional in other desktop environments.

## Development & Test Environment
Always contextualize suggestions, hardware-specific paths, and troubleshooting based on the user's primary workstation:
- **OS:** Garuda Linux x86_64 (`pacman` package manager)
- **Kernel:** Linux 7.0.0-rc1-1-cachyos-rc (custom/optimized kernel)
- **CPU:** AMD Ryzen 9 9950X3D (32 threads) @ 5.80 GHz (Target Hardware)
- **GPU:** AMD Radeon RX 7900 XTX [Discrete] (Mesa / radv)
- **Display Server / DE:** Wayland (KWin) within KDE Plasma 6.6.1
- **Shell / Terminal:** fish 4.5.0 / Konsole 25.12.2
