# Skyworth E900V21C / Amlogic S905L (P211) Android 6.0.1 — Field Notes

This repository documents a **real** migration of one China Unicom Skyworth E900V21C set-top box (S905L/GXL P211, 1GB RAM / 8GB eMMC) from Android 4.4.2 to a working **Android 6.0.1 / API 23** environment, verified on **2026-11-16**.

**Scope:** an engineering case study, beginner-friendly Chinese step-by-step checklist, recovery notes, and evidence boundaries. **Not** a universal flashable ROM or guaranteed unbrick package.

## What actually worked

- Correct P211-specific board/DTB information and the original **640MiB** system partition layout (the upstream generic image expected **1GiB** and had P212 second-stage DTBs).
- Android 6 boot and system components adapted to the original board's layout.
- **Restoring the original, device-specific FIP Bootloader** after an unsuccessful flashing attempt. The restored 896KiB FIP was read back from the physical eMMC boot location and matched the original backup byte-for-byte.
- Android 6.0.1 booted, `p211`, `sys.boot_completed=1`, Ethernet/ADB, HDMI 1080p60, software reboot, and a limited H.264/AAC playback smoke test.

## Major caution

The experimental `E900V21C_P211_Android6.0.1_USB.img` built on 2026-11-15 is **NOT** a verified stand-alone one-click flash image. Although its embedded erase flags were zero, the Windows USB Burning Tool GUI had both **Erase Flash** and **Erase Bootloader** enabled, causing a boot failure that required a board-specific FIP restoration.

We do **not** redistribute proprietary firmware, bootloaders, recovery images, or device backups. Do **not** flash another device's FIP.

## Documentation

- [中文 README](README.md)
- [新手准备与流程 / Beginner walkthrough](docs/01-新手准备与流程.md)
- [适配原理与镜像 / Board and firmware design](docs/02-适配原理与镜像.md)
- [故障恢复 / Failure analysis and recovery](docs/03-故障恢复.md)
- [验收与已知问题 / Validation matrix](docs/04-验收与已知问题.md)
- [来源、版权与安全 / Provenance and security](docs/05-来源版权与安全.md)

Source background: [Ubuntu5020/AmLogic-S905L](https://github.com/Ubuntu5020/AmLogic-S905L), [natinusala/linux-amlogic-toolkit](https://github.com/natinusala/linux-amlogic-toolkit).

Original documentation: MIT. Third-party binaries retain their own licenses and are not included.