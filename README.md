<div align="center">
  <img src="./vex.png" alt="Vexylon" width="100%">
</div>

<div align="center">

<a href="https://github.com/Ellioth7207/Vexylon/releases"><img src="https://img.shields.io/badge/Download-TapHere-0A84FF?style=flat-square&logo=android&logoColor=white" alt="Download Vexylon" style="border:2px solid #0A84FF;border-radius:8px;padding:2px;"></a>
<a href="https://t.me/Vennec"><img src="https://img.shields.io/badge/Telegram-@VENNEC-26A5E4?style=flat-square&logo=telegram&logoColor=white" alt="Support Channel" style="border:2px solid #0A84FF;border-radius:8px;padding:2px;"></a>

</div>

# Vexylon

**Universal thermal-throttling disabler for CPU, GPU, Battery & Camera.**
For ROOT Users — one module, every chipset.

---

## Read Before You Flash

Vexylon **intentionally disables the thermal-protection subsystem** of your device (CPU, GPU, battery and camera throttling). This is not a "safe tuning" module — it removes a safety mechanism to sustain peak performance.

- May cause the device to run **significantly hotter** under sustained load.
- May **reduce the lifespan** of the battery and other hardware components.
- **Not recommended** for daily driving — intended for benchmarking / short sustained-performance sessions.
- Use **external cooling** if you plan to run it for extended periods.
- You are fully responsible for any thermal damage, battery degradation, or instability resulting from use of this module. **Use at your own risk (DWYOR).**

If any of the above is unacceptable for your use case, do not install this module.

---

## What is Vexylon

**Vexylon** is a universal root module that disables thermal throttling across the four subsystems that most commonly cap sustained performance on Android: **CPU**, **GPU**, **battery**, and **camera**. It auto-detects your root manager and SoC vendor at install time, applies a boot-time script tuned for that platform, and ships a live WebUI so you can verify exactly what was changed — no guessing, no black box.

| | |
|---|---|
| **Module ID** | `vennec_vexylon` |
| **Version** | `12.1-BetaTest` (versionCode `1230`) |
| **Author** | [@vennec](https://t.me/Vennec) |
| **Minimum Magisk** | v20.4 (versionCode `20400`) |

---

## Features

- **CPU thermal disable** — neutralizes thermal zone modes, cooling devices, and (on MediaTek) the PPM power/thermal manager.
- **GPU thermal disable** — chipset-aware handling for Adreno, Mali, and PowerVR, with a universal fallback for unlisted SoCs.
- **Battery thermal disable** — disables battery/BMS/charger thermal zones and raises warm/hot temperature thresholds where exposed.
- **Camera thermal disable** — disables ISP/camera thermal zones and related vendor properties.
- **Universal auto-detection** — one build adapts to Snapdragon, MediaTek, Unisoc, Exynos, and Google Tensor platforms; unknown SoCs still get the universal fallback instead of failing silently.
- **Root-manager aware** — detects Magisk (incl. Kitsune/Alpha builds), KernelSU, KernelSU Next, APatch, MamboSU, and KOWSU.
- **Safe, name-matched process handling** — only known thermal-daemon process names are targeted; nothing is killed by a blind pattern match.
- **Non-destructive by design** — no partition flashing, no persistent boot image patching; changes are applied at runtime and via a reversible module overlay.
- **Live WebUI dashboard** — real-time status for every subsystem this module touches, device information, relevant system properties, and live temperature sensors.
- **Full activity log** — every write/kill/bind action is logged with a timestamp for transparency and troubleshooting.

---

## How It Works

Vexylon operates in three stages:

1. **Install time (`customize.sh`)**
   Detects the active root manager and SoC family, prints device information and compatibility warnings, sets correct permissions, and extracts a static `system/vendor` overlay (MediaTek thermal binaries and `.conf` profiles neutralized in place).

2. **Boot time (`post-fs-data.sh` → `service.sh`)**
   Waits for `sys.boot_completed`, then a short grace period, before touching anything (so it never interferes with the boot sequence). It then, per subsystem:
   - Writes `disabled` to relevant `thermal_zone*/mode` nodes and resets cooling device states.
   - Applies chipset-specific GPU throttling paths (see table below).
   - Kills only known thermal-daemon process names (`thermal-engine`, `thermalloadalgod`, `mi_thermald`, etc. — never a broad pattern match).
   - Bind-mounts `/dev/null` over thermal HAL binaries and vendor thermal executables that are still present and non-empty, preventing them from re-spawning.
   - Logs every action (success or failure) to `vexylon_thermal.log` inside the module directory.

3. **Runtime (WebUI)**
   A dashboard, served from `webroot/`, queries the live device state through the KernelSU/MMRL shell bridge and reports **exactly** what the module has verified — it never claims a status it hasn't actually checked.

No sysfs node, property, or service is assumed to exist: every write is guarded by an existence check first, so the same build behaves safely across devices that don't expose a given node.

---

## Chipset / GPU Coverage

| Platform prefix | GPU family | Handling |
|---|---|---|
| `msm*`, `sdm*`, `sm*`, `sc*`, `qcom*`, `apq*` | Adreno (Snapdragon) | KGSL throttling nodes disabled; `thermal-engine` / `vendor.qti.hardware.perf` stopped |
| `mt*` | Mali (MediaTek) | `gpufreq` power-limit nodes disabled; MTK thermal services stopped |
| `unisoc*`, `ums*`, `sp*` | PowerVR / Mali (Unisoc) | `sprd_cpu_cooling` disabled; Unisoc thermal service stopped |
| `exynos*`, `universal*`, `s5e*` | Mali (Exynos) | Mali DVFS threshold and Samsung thermal service disabled |
| `gs*`, `tensor*`, `whitechapel*` | Mali (Google Tensor) | `thermal-engine` stopped, universal GPU paths applied |
| Anything else | Unknown | Universal fallback paths only — device still gets CPU/battery/camera handling |

---

## Compatibility

| Requirement | Details |
|---|---|
| **Root solution** | Magisk ≥ 20.4, Magisk Kitsune/Alpha/Delta, KernelSU, KernelSU Next, APatch, MamboSU, KOWSU |
| **Android version** | Android 10+ (API 29+) recommended; the installer warns (but does not hard-block) on lower API levels |
| **Architecture** | Universal — SoC family is detected at install time, not assumed |
| **WebUI runtime** | A KernelSU-compatible WebUI environment (KernelSU Next, MMRL, or equivalent `ksu.exec`/`mmrl.exec` bridge) is required for **live** data. Without it, the WebUI still loads but shows static placeholders only. |

---

## Installation

1. Download the latest `Vexylon_*.zip`
2. Open your root manager (Magisk Manager, KernelSU Manager, APatch, etc.) → **Modules** → **Install from storage**.
3. Select the downloaded ZIP and confirm.
4. Read the install log — it will show your detected root manager, device, and SoC, plus the same warning shown above.
5. **Reboot** your device to activate the module.

> Vexylon does not require Zygisk, a companion app, or any additional dependency to function.

---

## Verifying the Module Is Active

- **WebUI** — the fastest way; every row is a live check, not a static claim.
- **Log file** — `/data/adb/modules/vennec_vexylon/vexylon_thermal.log` contains a full timestamped record of every write, kill, and bind action performed at last boot, plus a final `X writes ok, Y failed` summary.
- **Notification** — on devices where a root shell (`su`) is available, Vexylon posts a one-time notification once the boot-time script finishes applying changes.

---

## Uninstallation

1. Open your root manager → **Modules**.
2. Remove/disable **Vexylon**.
3. Reboot.

Vexylon does not flash or modify any partition permanently. All effects are applied at runtime (property overrides, sysfs writes, and bind mounts), so removing the module and rebooting fully restores the stock thermal behavior for anything outside the static `system/vendor` overlay, which is removed with the module itself.

---

## Known Limitations

- Some OEM skins run a watchdog that respawns thermal daemons after the initial kill pass; Vexylon does not currently re-arm after boot.
- Effectiveness depends on how closely a device's vendor thermal implementation matches the paths this build targets — heavily customized OEM thermal stacks may only be partially covered by the universal fallback.
- The WebUI shows live data only when a KernelSU/MMRL-compatible shell bridge is available; outside that environment it renders with static placeholders.
- This is a **Beta** build (`12.1-BetaTest`) — behavior may change between releases.

---

## FAQ

**Q: My device still throttles after installing — why?**
A: Check the WebUI's "Thermal Protection" section first — if a row shows a non-disabled state, your device likely exposes a vendor-specific node this build doesn't yet target. Check `vexylon_thermal.log` for write failures and open an issue with your device's `Platform` and `Codename` (both visible in the WebUI).

**Q: Is this safe for daily use?**
A: No — see [Read Before You Flash](#️-read-before-you-flash). It's built for controlled, short sustained-performance sessions with adequate cooling.

**Q: Does this touch my bootloader, partitions, or void my warranty in a permanent way?**
A: No partitions are flashed and nothing is permanently patched. That said, running your device outside its designed thermal envelope carries its own hardware risk regardless of how it's applied — see the warning above.

**Q: Which root solutions are supported?**
A: Magisk (including Kitsune/Alpha/Delta builds), KernelSU, KernelSU Next, APatch, MamboSU, and KOWSU are all detected automatically.

---

## Attribution and Redistribution Policy

Re-uploading, mirroring, or redistributing Vexylon is permitted under the following terms.

**Attribution is required.** Every redistribution must clearly and visibly credit the original source:

> Source: @vennec

This attribution must not be removed, hidden, or replaced.

**Not permitted:**

- Claiming Vexylon as original work
- Removing, hiding, or replacing the `@vennec` attribution
- Presenting an unofficial build as an official Vexylon release
- Removing the original source or download information
- Changing, shortening, redirecting, or hiding the official links without authorization
- Using modified links in a way that misleads users about the project's origin

## Link Policy

Official links associated with Vexylon (download, source, and release links) must not be changed, replaced, shortened, redirected, or hidden without prior written permission from [Telegram : @ellioth7207](https://github.com/ellioth7207). This applies to links in redistribution posts and repackaged copies of the module alike.

## Disclaimer

Vexylon disables Android's thermal-throttling protection for the CPU, GPU, battery, and camera subsystems. The author assumes no responsibility for bootloops, system instability, data loss, device malfunction, incompatibility with specific devices, conflicts with other modules, overheating, accelerated hardware/battery degradation, or any other damage resulting from use of this module. Install and use at your own risk.

## Credits

| Role | Handle |
| --- | --- |
| Development and maintenance | [@vennec](https://t.me/Vennec) |
| Link authorization | [@ellioth7207](https://t.me/ellioth7207) |

<hr>

<p align="center"><sub>Copyright ©VENNEC . All attribution and link‑policy terms above apply to any redistribution of this project.</sub></p>

<div align="center">

<a href="https://i.ibb.co.com/TxNqw4c4/qr-ID1026576754000-03-09-26-1788411437-1788411438034.jpg"><img src="https://img.shields.io/badge/Donate-QRIS-FF9500?style=flat-square" alt="Donate" style="border:2px solid #0A84FF;border-radius:8px;padding:2px;"></a>

</div>

