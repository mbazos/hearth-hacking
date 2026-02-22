# Hearth Hacking

Turn a [Hearth Display](https://www.hearth.com/) into a fully usable Android tablet.

The Hearth Display runs Android 11 on a Rockchip rk3568 SoC but is locked down as a kiosk device. This project provides the tools and instructions to break out of the kiosk, install a launcher, browser, and gesture navigation, giving you a general-purpose tablet experience.

## What Gets Installed

| App | Purpose |
|-----|---------|
| **Nova Launcher** | Replaces the Hearth kiosk with a customizable home screen |
| **Firefox** | Full browser with extension support |
| **Fluid Navigation Gestures** | Adds swipe-based back/home/recents (the device has no nav buttons) |
| **PullTab Fullscreen** | Firefox extension that hides the address bar for more screen space |

All APKs and extensions are included as a multi-part RAR archive (`apps.part1.rar` through `apps.part7.rar`). Extract with `unrar x apps.part1.rar` to recreate the `apps/` folder.

## How It Works

1. **Connect via ADB** over the network to the Hearth Display
2. **Gain root access** (`adb root`) to override the system-level kiosk watchdog
3. **Disable the Hearth kiosk** packages (`com.hearth.hmf` and `com.nativeapp`) so they stop reclaiming the foreground
4. **Install apps** via `adb install` and set Nova Launcher as the default home
5. **All changes persist across reboots** — no need to re-apply after power cycling

The kiosk can be fully restored at any time by re-enabling the Hearth packages.

## Prerequisites

- ADB installed on your computer
- Hearth Display on the same network
- The device's IP address and ADB port

## Getting Started

See [HEARTH_HACKING.md](HEARTH_HACKING.md) for the full step-by-step guide covering installation, configuration, diagnostics, and how to restore the original kiosk.
