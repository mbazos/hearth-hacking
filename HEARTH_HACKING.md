# Hearth Device - Full Android Tablet Experience

Turn the Hearth Display into a fully usable Android tablet by installing a launcher, browser, and gesture navigation. This guide covers disabling the Hearth kiosk, installing apps via ADB, and restoring the device to its original state.

## Device Info
- **Device**: Hearth Display (rk3568 / Rockchip SoC)
- **ADB Target**: `<ip address>:<port>`
- **OS**: Android 11 (SDK 30), arm64-v8a
- **Kiosk**: Managed by Hearth (`com.hearth.hmf` / `com.nativeapp`)

## Apps & Extensions

| App | Version | Purpose |
|-----|---------|---------|
| Nova Launcher | 8.3.5 | Home screen replacement with customizable grid, gestures, and app drawer |
| Firefox | 143.0.4 (Fenix) | Full-featured browser with extension support |
| Fluid Navigation Gestures | 2.0-beta11 | Swipe-based navigation (back, home, recents) replacing the missing nav bar |
| PullTab Fullscreen | 1.1 (Firefox extension) | Hides the Firefox address bar to maximize screen space |

All APKs and extensions are stored as a multi-part RAR archive (`apps.part1.rar` through `apps.part7.rar`). Extract with `unrar x apps.part1.rar` to recreate the `apps/` folder.

## Quick Start

- **First time?** Install apps (see [Nova Launcher](#nova-launcher), [Firefox](#firefox), [Fluid Navigation Gestures](#fluid-navigation-gestures) sections below), then follow [Full Tablet Mode Setup](#full-tablet-mode-setup).
- **Already set up?** The setup persists across reboots — Nova Launcher stays as the default home and Hearth packages stay disabled.
- **Want to restore kiosk mode?** Follow [Restore Hearth Kiosk](#restore-hearth-kiosk).

---

## Full Tablet Mode Setup

Permanently disable the Hearth kiosk and set Nova Launcher as the default home. This persists across reboots.

```bash
# 1. Connect & root
adb connect <ip>:<port>
adb -s <ip>:<port> root

# 2. Disable Hearth packages (persists across reboots)
adb -s <ip>:<port> shell "pm disable-user --user 0 com.hearth.hmf"
adb -s <ip>:<port> shell "pm disable-user --user 0 com.nativeapp"

# 3. Set Nova Launcher as default home
adb -s <ip>:<port> shell "cmd package set-home-activity com.teslacoilsw.launcher/.NovaLauncher"

# 4. Reboot to verify persistence
adb -s <ip>:<port> reboot
```

After reboot, reconnect and verify:
```bash
adb connect <ip>:<port>

# Verify Nova Launcher is in foreground
adb -s <ip>:<port> shell dumpsys activity activities | grep mResumedActivity

# Verify Hearth packages are still disabled
adb -s <ip>:<port> shell pm list packages -d
```

---

## Restore Hearth Kiosk

Fully revert to the original Hearth kiosk experience:

```bash
# 1. Re-enable Hearth packages
adb -s <ip>:<port> shell "pm enable com.hearth.hmf"
adb -s <ip>:<port> shell "pm enable com.nativeapp"

# 2. Clear Nova Launcher's default home status
adb -s <ip>:<port> shell "cmd package set-home-activity com.android.settings/.FallbackHome"

# 3. Reboot — Hearth watchdog will resume and relaunch nativeapp
adb -s <ip>:<port> reboot
```

## Known Issue: Hearth Kiosk Reclaims Foreground

The Hearth management framework (`com.hearth.hmf`) runs with **system-level privileges** and acts as a persistent watchdog service. It automatically relaunches the native kiosk app (`com.nativeapp`) when it's stopped, making it extremely difficult to keep other apps in the foreground. Any browser you launch will likely be pushed to the background almost immediately.

**Root access is required** to override this behavior. The **solution** is to disable the Hearth packages using `pm disable-user --user 0`, which prevents them from reclaiming the foreground. This disabled state **persists across reboots** — Hearth packages stay disabled until explicitly re-enabled with `pm enable`.

Additionally, `cmd package set-home-activity` permanently sets the default launcher, so Nova Launcher remains the home screen after reboot without needing to re-select it. See [Full Tablet Mode Setup](#full-tablet-mode-setup) for the complete procedure and [Restore Hearth Kiosk](#restore-hearth-kiosk) to revert.

---

## Nova Launcher

Nova Launcher replaces the Hearth kiosk as the default home screen, giving you an app drawer, customizable grid, and gesture shortcuts.

### 1. Install Nova Launcher

```bash
adb -s <ip address>:<port> install apps/com.teslacoilsw.launcher_81029_\(8.3.5\)-81029_minAPI26\(arm64-v8a,armeabi-v7a\)\(nodpi\)_apkmirror.com.apk
```

### 2. Launch Nova Launcher

```bash
adb -s <ip address>:<port> shell "am start -n com.teslacoilsw.launcher/com.teslacoilsw.launcher.NovaLauncher"
```

When prompted, select Nova Launcher as the default home app (choose "Always").

### 3. Download (if not using bundled APK)

Download from [APKMirror](https://www.apkmirror.com/apk/teslacoil-software/nova-launcher/) and place in the `apps/` folder.

---

## Fluid Navigation Gestures

The Hearth Display has no physical navigation buttons or on-screen nav bar. Fluid Navigation Gestures adds swipe-based gestures from the screen edges for back, home, and recents.

### 1. Install Fluid Navigation Gestures

```bash
adb -s <ip address>:<port> install apps/com.fb.fluid_2.0-beta11-178_minAPI22\(arm64-v8a,armeabi-v7a,x86,x86_64\)\(nodpi\)_apkmirror.com.apk
```

### 2. Grant Accessibility Permission

Fluid Navigation Gestures requires Accessibility Service access. You can enable it via:
```bash
adb -s <ip address>:<port> shell settings put secure enabled_accessibility_services com.fb.fluid/com.fb.fluid.service.AccessibilityService
```

### 3. Launch Fluid Navigation Gestures

```bash
adb -s <ip address>:<port> shell "am start -n com.fb.fluid/.ui.ActivitySettings"
```

Configure your preferred gesture triggers (edge swipes for back, home, recents).

### 4. Download (if not using bundled APK)

Download from [APKMirror](https://www.apkmirror.com/apk/francisco-barroso/fluid-navigation-gestures/) and place in the `apps/` folder.

---

## Firefox

### 1. Verify ADB Connection

```bash
adb -s <ip address>:<port> get-state
```

### 2. Enable Root Access (Required for Hearth Kiosk Override)

```bash
adb -s <ip address>:<port> root
```

**Note**: After running `adb root`, the direct IP connection may go offline. If this happens, use the mDNS connection instead (visible in `adb devices -l` as `adb-<serial>-<hash>._adb-tls-connect._tcp`). Verify root access with:

```bash
adb -s <device-identifier> shell id
# Should show: uid=0(root) gid=0(root) ...
```

### 3. Uninstall Broken Firefox (if present)

```bash
adb -s <ip address>:<port> uninstall org.mozilla.firefox
```

### 4. Download Firefox APK (Version 143.0.4 - Tested & Working)

**Recommended**: Firefox 143.0.4 has been tested and works well with Android 11 on this device.

```bash
# Download Firefox 143.0.4 from Mozilla Archive
curl -L -o /tmp/firefox-143.0.4.apk "https://archive.mozilla.org/pub/fenix/releases/143.0.4/android/fenix-143.0.4-android-arm64-v8a/fenix-143.0.4.multi.android-arm64-v8a.apk"
```

**Alternative - Latest Release**:
```bash
# Get latest Firefox release version
LATEST_RELEASE=$(curl -s https://api.github.com/repos/mozilla-mobile/firefox-android/releases | grep '"tag_name": "fenix-' | head -1 | cut -d'"' -f4)
VERSION=$(echo $LATEST_RELEASE | sed 's/fenix-v//')

# Download arm64-v8a APK
curl -L -o /tmp/firefox-release.apk "https://github.com/mozilla-mobile/firefox-android/releases/download/${LATEST_RELEASE}/fenix-${VERSION}-arm64-v8a.apk"
```

**Legacy Fennec 68.11.0** (not recommended):
```bash
curl -L -o /tmp/firefox-arm64.apk "https://download.mozilla.org/?product=fennec-latest&os=android&lang=multi"
```

### 5. Install Firefox

```bash
adb -s <ip address>:<port> install /tmp/firefox-143.0.4.apk
```

**Note**: Installation may take 30-60 seconds due to APK size (~111MB).

### 6. Disable Hearth Packages (Keeps Browser in Foreground)

**IMPORTANT**: This step requires root access (Step 2). Without this, Hearth will reclaim the foreground. If you've already completed the [Full Tablet Mode Setup](#full-tablet-mode-setup), this step is already done.

```bash
adb -s <ip address>:<port> shell "pm disable-user --user 0 com.hearth.hmf"
adb -s <ip address>:<port> shell "pm disable-user --user 0 com.nativeapp"
```

**Expected output:**
```
Package com.hearth.hmf new state: disabled-user
Package com.nativeapp new state: disabled-user
```

### 7. Launch Firefox

```bash
adb -s <ip address>:<port> shell "am start -n org.mozilla.firefox/org.mozilla.fenix.HomeActivity"
```

### 8. Install PullTab Fullscreen Extension (Optional - Hides Address Bar)

**Extension**: PullTab Fullscreen - hides the address bar to maximize screen space on mobile devices.

Download and push the extension:
```bash
# Download PullTab Fullscreen extension
curl -L -o /tmp/pulltab_fullscreen-1.1.xpi "https://addons.mozilla.org/firefox/downloads/file/4522717/pulltab_fullscreen-1.1.xpi"

# Push to device
adb -s <ip address>:<port> push /tmp/pulltab_fullscreen-1.1.xpi /sdcard/Download/pulltab_fullscreen-1.1.xpi
```

**Manual Installation in Firefox**:
1. Open Firefox on the device
2. Navigate to: `about:config`
3. Search for and enable: `xpinstall.signatures.required` (set to false if needed)
4. Navigate to: `file:///sdcard/Download/pulltab_fullscreen-1.1.xpi`
5. Or use Firefox menu → Add-ons → Install add-on from file

**Verified Working**: This extension works perfectly with Firefox 143.0.4 to hide the address bar.

### 9. Verify Firefox Is in Foreground

Check that Firefox is running:
```bash
adb -s <ip address>:<port> shell ps -A | grep firefox
```

Verify Firefox is the active app:
```bash
adb -s <ip address>:<port> shell dumpsys activity activities | grep "mResumedActivity"
# Should show: mResumedActivity: ActivityRecord{...} org.mozilla.firefox/.App
```

### 10. Re-enable Hearth (Optional)

To restore Hearth, see [Restore Hearth Kiosk](#restore-hearth-kiosk).

---

## Diagnostics

Check which activity is in the foreground:
```bash
adb -s <ip address>:<port> shell dumpsys activity activities | grep "mResumedActivity"
```

Check lock task state:
```bash
adb -s <ip address>:<port> shell dumpsys activity activities | grep "mLockTaskModeState"
```

Check for crashes:
```bash
adb -s <ip address>:<port> shell logcat -d -t 100 | grep -i -E "crash|fatal|AndroidRuntime"
```

## Notes

### Tested Configuration
- **Device**: Hearth Display (rk3568 / Rockchip SoC)
- **Android Version**: Android 11 (SDK 30), arm64-v8a
- **Nova Launcher**: 8.3.5 (home screen replacement)
- **Firefox**: 143.0.4 (Fenix - tested & recommended, ~111MB)
- **Fluid Navigation Gestures**: 2.0-beta11 (edge swipe navigation)
- **PullTab Fullscreen**: 1.1 (Firefox extension - hides address bar)

### App Versions
The bundled app versions were tested and working as of February 2026. Newer versions of these applications may be available and could offer improvements — especially Firefox, which receives frequent updates. When looking for newer versions, ensure they are compatible with Android 11 (API level 30) and the arm64-v8a architecture. The versions included in this repo should continue to work as a fallback.

### How It Works
- **Root access is required** to effectively override the Hearth kiosk system.
- `com.hearth.hmf` runs with **system-level privileges** (UID: system) and acts as a watchdog that automatically relaunches `com.nativeapp`.
- **Solution**: Use `pm disable-user --user 0` to disable both Hearth packages. This **persists across reboots**.
- `cmd package set-home-activity` sets the permanent default launcher (also persists across reboots).
- **⚠️ WARNING**: Do NOT use `kill -9` on system processes - it will break ADB connectivity and may require device reboot.
- To restore Hearth, re-enable with `pm enable` and reset the home activity (see [Restore Hearth Kiosk](#restore-hearth-kiosk)).
