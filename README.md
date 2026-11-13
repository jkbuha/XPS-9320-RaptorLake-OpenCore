# Dell XPS 13 Plus 9320 macOS Tahoe with OpenCore

<!-- Add a screenshot as ./screenshot/screen.jpg -->

# Details

| OpenCore Version | 1.0.8 |
| --- | --- |
| macOS Version | 26.7.3 (Tahoe) |
| SMBios | MacBookAir9,1 |
| iGPU driver | [WhateverXe](https://github.com/jkbuha/WhateverXe/tree/xps-unified) (`xps-unified`) + WhateverXe Graphics |

# Hardware Specifications

| Hardware | Specification | Status |
| --- | --- | --- |
| CPU | Intel Core i7-1360P (Raptor Lake-P, 4P + 8E) | ✅ Working |
| RAM | LPDDR5 32GB | ✅ Working |
| GPU | Intel Iris Xe Graphics (`8086:A7A0`, 96 EU) | ✅ Working (Metal 2, 1536 MB) |
| Display | 13.4" 1920 x 1200 FHD+ eDP | ✅ Working |
| Audio | SoundWire: 2× RT1316 amplifiers + RT714 microphones (cAVS DSP) | ✅ Working (AudShim, speakers + mic) |
| WiFi | Intel AX211 (CNVi, `8086:51F1`) | ✅ Working (AirportItlwm) |
| Bluetooth | Intel AX211 | ✅ Working |
| SSD | Crucial P3 2TB | ✅ Working |
| Keyboard | - | ✅ Working |
| Trackpad | I2C haptic touchpad | ✅ Working (VoodooI2C, interrupt mode) |
| USB-C / Thunderbolt 4 | 2 ports | ✅ Working (USBToolBox map) |
| Webcam | MIPI (IPU6) camera | ❌ Not Working |
| Fingerprint Sensor | - | ❌ Not Working |
| Sleep / S4 | Hibernation (`hibernatemode 25`) | ✅ Working |
| Display sleep / wake | - | ✅ Working |

# Overview

The XPS 13 Plus 9320 now runs macOS Tahoe with **native Iris Xe acceleration**, using b00t0x's
[WhateverXe](https://github.com/b00t0x/WhateverXe) and the XPS fixes collected on the `xps-unified` branch of
[jkbuha/WhateverXe](https://github.com/jkbuha/WhateverXe/tree/xps-unified). That is the same kext used by the
[XPS 17 9720 repo](https://github.com/jkbuha/XPS-9720-AlderLake-OpenCore); the two laptops differ only in boot-args.

Built-in audio works through **AudShim**, a small kext that talks to the cAVS DSP, SoundWire, the RT1316 amplifiers
and the RT714 microphones (Intel SOF firmware + topology), plus its `XeSpeakers` audio plug-in for macOS.

# Graphics (WhateverXe)

Graphics needs both halves of WhateverXe:

1. `EFI/OC/Kexts/WhateverXe.kext` (in this repo): the Lilu plugin, built from `xps-unified`.
2. **WhateverXe Graphics** (Apple's Tiger Lake driver bundles, installed into `/Library/Extensions` after macOS is installed).
   Follow the installation steps in the [WhateverXe README](https://github.com/b00t0x/WhateverXe).

Boot-args specific to this machine:

| Boot-arg | Purpose |
| --- | --- |
| `-wex-resume-park` | Parks the firmware eDP scanout on hibernation resume so the display comes back cleanly |
| `audshimcapdb=6` | AudShim speaker gain cap (dB) |
| `hpmepp=64 hpmcst=0 hpmd3=1 hpmbt=0` | HybridPM power-management tuning |

The 9320's firmware leaves the internal panel's port in "mode 0"; the `xps-unified` build admits it through the
protected AUX lease, so **no extra flag is needed** (the 9720 uses `-wex-panel-handoff` instead).

# SMBIOS

The serial number, MLB, SystemUUID and ROM in `config.plist` are **blank**. Generate your own before booting:

1. Run [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) and generate a `MacBookAir9,1` SMBIOS.
2. Fill `PlatformInfo > Generic > SystemSerialNumber`, `MLB`, `SystemUUID`, and set `ROM` to your Wi-Fi/NIC MAC (6 bytes).
3. Check the serial is **not** valid on Apple's coverage site before using iCloud/iMessage.

# BIOS Settings

| Setting | Option |
| --- | --- |
| SATA/NVMe Operation | AHCI/NVMe (not RAID) |
| Fast Boot | Thorough |
| Secure Boot | Disabled |
| TPM 2.0 Security | Disabled |
| Intel SGX | Disabled |
| VT for Direct I/O | Disabled |
| Fingerprint Reader | Disabled |

# UEFI IFR edits

As with the other XPS repos, disable `CFG_LOCK` with modGRUBshell before installing. **Check the variable offsets
against your own BIOS version's IFR dump**; they change between Dell BIOS releases.

# Sleep / Hibernation

Hibernation (`sudo pmset -a hibernatemode 25`) works: the machine powers off fully and resumes into the same session,
with WhateverXe's `-wex-resume-park` handling the display hand-over. `HibernationFixup` is included.

# Rollback

Keep a known-good copy of your EFI. A simple pattern used on this machine: keep the previous `WhateverXe.kext` and
`config.plist` in a folder on the EFI and a small script that copies them back, which you can run from Recovery,
from macOS, or from a Linux install (`/boot/efi`) if macOS won't boot.

# Known Issues

- MIPI (IPU6) webcam and fingerprint reader are not supported.
- HDMI/DP **audio** from the iGPU does not work yet (see [b00t0x/WhateverXe#124](https://github.com/b00t0x/WhateverXe/issues/124)).
- **Undervolting:** keep the System Agent and I/O planes at 0 mV with the iGPU in use. Negative SA/IO offsets caused
  silent single-bit memory corruption (APFS damage) on the sister XPS 17 9720 while CPU/Cache offsets were fine.

# Credits

acidanthera (OpenCore, Lilu, VirtualSMC, AppleALC, HibernationFixup, RestrictEvents, BrightnessKeys),
[b00t0x](https://github.com/b00t0x/WhateverXe) (WhateverXe), zxystd (itlwm / IntelBluetoothFirmware),
VoodooI2C team, [JulioNectar](https://github.com/JulioNectar/Hackintosh-XPS13Plus-IrisXe-Tahoe) (AudShim, XPS 13 Plus Tahoe work),
anv1552 / Thatisjigen (HybridPM), dhinakg (AMFIPass), USBToolBox.

# Donate

If you like and use this project, feel free to contribute anything you can below:

Paypal: https://paypal.me/jkbuha

Bitcoin: bc1qjuy6hy5shqqdnyzzcjlc2h4ua47gpugphuxdae
