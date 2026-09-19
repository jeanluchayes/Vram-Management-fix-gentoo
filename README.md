# Linux VRAM Manager


A Bash-based utility for Linux gaming systems using the kernel DMEM cgroup interface. It installs and enables the available VRAM-management stack, selects a desktop-appropriate integration when one exists, applies a persistent `dmem.max` VRAM headroom limit, verifies the configuration, and optinally allows for reverting all changes.

> **Experimental:** the `dmem.max` ceiling is a workaround intended to leave a small amount of VRAM headroom instead of allowing `app.slice` to consume the full reported DMEM capacity. Results can vary by GPU, driver, kernel, game, workload, and desktop environment.

## Features

- Install and enable the base DMEM/VRAM-management stack
- Detect AMD, Intel, NVIDIA DMEM regions
- Detect the current user and systemd cgroup path automatically
- Detect DE and Wayland compositors
- Select a DE-specific foreground integration where a verified project exists
- change the VRAM safety margin in MiB
- create a systemd oneshot service that Applies the ceiling persistently
- Verify `dmem.capacity`, `dmem.max`, `dmem.current`, and service state
- Remove installed VRAM-management packages when requested

## Desktop / compositor support

Compatible desktop integrations:
- KDE Plasma
- GNOME
- Hyprland
- Niri

Other desktops and window managers should use Gamescope as the generic fallback.

## How it works

The base DMEM stack exposes device-memory cgroups. The custom workaround lowers:

```text
app.slice/dmem.max
```

below the reported device-memory capacity, leaving a configurable reserve.

Example for a 4 GiB GPU with 50 MiB of headroom:

```text
VRAM capacity : 4278190080 bytes
Safety margin : 50 MiB
VRAM ceiling  : 4225761280 bytes
```

The custom service is a **oneshot**. It applies the limit and exits; it does not continuously monitor VRAM. The kernel continues enforcing the cgroup limit after the service exits.

The helper handles multiple `vram` / `vidmem` regions in one `dmem.max` write so multi-GPU or multi-region systems do not have one entry overwrite another.

## Requirements

The persistent ceiling path requires:

- systemd
- cgroup v2
- a kernel with the DMEM cgroup controller
- a GPU driver exposing device memory through DMEM
- a systemd user hierarchy containing `app.slice`

Automatic package installation currently targets Arch-family and Fedora-family systems. Other systems can still work when the same kernel, systemd, cgroup, and driver requirements are satisfied.

## GPU support

For NVIDIA, working support still depends on a driver/kernel combination that exposes video memory through Linux DMEM. An NVIDIA GPU by itself does not guarantee that `dmem.capacity` will contain a usable `vidmem` entry.

## Installation / usage

### From source

```bash
chmod +x vram-manager.sh
./vram-manager.sh
```

### Binary

Run `Linux-VRAM-Manager-x86_64` from the GitHub release assets on x86_64 Linux.

## Menu

```text
1) Install / enable VRAM management
2) Apply / change VRAM ceiling
3) Verify current / post-reboot state
4) Remove custom VRAM ceiling
5) Remove everything
6) System / package status
7) Exit
```

## Files created by the custom limiter

```text
/usr/local/sbin/set-dmem-appslice-limit
/etc/systemd/system/dmemcg-appslice-limit.service
/etc/default/dmemcg-appslice-limit
/var/lib/linux-vram-manager/installed-packages
```

The live cgroup setting is under:

```text
/sys/fs/cgroup/.../app.slice/dmem.max
```

The package-installation record is used so **Remove everything** does not blindly remove VRAM packages that were already installed before the tool was run.

## What the custom limiter does not modify

- GPU firmware or GPU BIOS
- motherboard BIOS/UEFI settings
- the kernel image or kernel source
- game files
- Proton/Wine prefixes
- swap configuration
- physical VRAM capacity
- ordinary personal files or media

The `/sys/fs/cgroup` entries are kernel-managed interfaces, not ordinary files stored on disk.

## Upstream projects and credits

This project does not claim ownership of the upstream DMEM/foreground-management projects. It combines their available functionality with a separate configurable `dmem.max` headroom workaround.

- **dmemcg-booster:** https://gitlab.steamos.cloud/holo/dmemcg-booster
- **KDE KCGroups:** https://github.com/pixelcluster/kcgroups
- **GNOME VRAM Booster:** https://github.com/sachesi/gnome-vram-booster
- **Hyprland Focused Booster:** https://github.com/tumrin/hyprland-focused-booster
- **Niri Focused Booster:** https://github.com/1Naim/niri-focused-booster
- **Gamescope:** https://github.com/ValveSoftware/gamescope

The underlying `dmem.*` controller is provided by the Linux kernel; this project does not implement the kernel controller.

## AI disclosure

This project was developed with AI assistance. The source is published so the implementation can be inspected directly. The custom `dmem.max` workaround was personally modified and tested on CachyOS.

## License

The custom code in this repository is released under the MIT License. Third-party projects, packages, and kernel components referenced by this project remain under their respective upstream licenses.
