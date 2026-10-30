# OpenCore EFI — ASUS TX GAMING B760M WIFI D4 (i5-12600KF + RTX 3060 12 GB)

A ready-to-use OpenCore EFI for the **ASUS TX GAMING B760M WIFI D4** motherboard,
paired with an **Intel Core i5-12600KF (Alder Lake)** and an **NVIDIA RTX 3060 12 GB**,
running **macOS 26 Tahoe** as a daily-driver Hackintosh.

> **Branch layout**
> - **`main`** (this branch) = **macOS 26 Tahoe** build — the currently running, verified setup.
> - **`macos-15-sequoia`** branch = the earlier **macOS 15 Sequoia** build.
>
> Generated with the **NullMoth "1401 for Windows"** OpenCore build, tuned for
> **macOS 26.7.2 (build 25G332)**.

---

## ⚠️ Before you start — important

1. **The `config.plist` in this repo has been SANITIZED.** The original SMBIOS serial
   number, MLB, SystemUUID and ROM were blanked to protect the author's identity and to
   avoid serial collisions. **You MUST generate your own SMBIOS** (see
   [Generate your SMBIOS](#generate-your-smbios)) before the first boot, or macOS/iServices
   will not work and you risk colliding with someone else's serial.
2. This is a third-party, community-maintained configuration. **Use at your own risk.**
   A wrong EFI can prevent the machine from booting — always keep a known-good backup of
   your working EFI on a separate USB stick.
3. NVIDIA acceleration relies on the **third-party NullMoth NVIDIA driver**, which is NOT
   Apple code. Read the [Graphics](#graphics--nvidia-rtx-3060--nullmoth-driver) section.

---

## Hardware specifications

| Component      | Model                                                              |
| -------------- | ----------------------------------------------------------------- |
| Motherboard    | ASUS TX GAMING B760M WIFI D4 (B760, LGA1700, Raptor Lake PCH)      |
| BIOS           | Validated on BIOS **1836**                                        |
| SMBIOS         | MacPro7,1 (SecureBootModel: Disabled)                             |
| CPU            | Intel Core i5-12600KF (Alder Lake-S, no iGPU)                     |
| GPU            | NVIDIA GeForce RTX 3060 **12 GB** (Ampere) — via NullMoth driver   |
| Memory         | DDR4 (fill in your own SMBIOS / SystemMemoryStatus)               |
| Ethernet       | Realtek RTL8125 2.5GbE — `RTL812xLucy.kext`                       |
| Wi‑Fi          | Intel AX201 (CNVi) — `AirportItlwm-Tahoe.kext` (+ `itlwm.kext`)   |
| Bluetooth      | Intel AX201 — `IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup` |
| Audio          | `VoodooHDA_NV.kext` — onboard ALC897 (7.1CH / SPDIF / Headphones) **and** NVIDIA HDMI audio, all working |
| Storage        | NVMe SSD — `NVMeFix.kext`                                         |
| **OS**         | **macOS 26.7.2 Tahoe (25G332) — tested**                          |

### Boot arguments (already configured)

```
debug=0x100 nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth
amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0
nvfbsample=1 gfxrst=4
```

---

## What works

| Feature               | Status / Notes                                                    |
| --------------------- | ----------------------------------------------------------------- |
| **macOS 26 Tahoe**    | ✅ **Tested (26.7.2)** — this branch                              |
| macOS 15 Sequoia      | ✅ Available on the `macos-15-sequoia` branch                      |
| CPU power management  | ✅ `CpuTopologyRebuild`, `SMCProcessor`, `SMCSuperIO`              |
| Ethernet (2.5G)       | ✅ `RTL812xLucy.kext`                                             |
| Wi‑Fi (AX201)         | ✅ `AirportItlwm-Tahoe.kext` (native menu) + `itlwm.kext` enabled  |
| Bluetooth (AX201)     | ✅ `IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup`   |
| Audio (all)           | ✅ `VoodooHDA_NV.kext` — ALC897 (7.1CH, SPDIF-out, Headphones) + NVIDIA Audio (HDMI/DP) |
| USB (mapping)         | ✅ `USBToolBox.kext` + `UTBDefault.kext` + SSDT-USBX               |
| NVIDIA RTX 3060       | ✅ Acceleration via NullMoth driver; ✅ HDMI/DP audio              |

---

## Generate your SMBIOS

The shipped `config.plist` has empty serials. Generate a valid `MacPro7,1` identity:

1. Download **GenSMBIOS** (or use **OCAT** → `PlatformInfo` → `Generate`).
2. Pick model `MacPro7,1` and generate:
   - `SystemSerialNumber`
   - `MLB`
   - `SystemUUID`
   - `ROM` (usually your en0 MAC address, 6 bytes)
3. Open `EFI/OC/config.plist` → `PlatformInfo` → `Generic` and fill in the four values.

> Never reuse someone else's serial. Keep yours private — do not publish your generated `config.plist`.

---

## BIOS Settings (ASUS TX GAMING B760M WIFI D4)

Flash a known-good BIOS first — this EFI was validated on **BIOS 1836**. Enter the firmware
setup with `Del` / `F2` and apply the following (use the `F9` search if a menu name differs):

**Boot / Security**

| Setting                            | Value |
| ---------------------------------- | ----- |
| Fast Boot                          | **Disabled** |
| Secure Boot → OS Type              | **Other OS** (or Secure Boot **Disabled**) |
| CSM (Compatibility Support Module) | **Disabled** (pure UEFI) |

**Advanced → CPU Configuration**

| Setting                                | Value |
| -------------------------------------- | ----- |
| Intel Virtualization Technology (VT-x) | **Enabled** |
| Intel VT-d                             | **Enabled** (the `DisableIoMapper` quirk handles it) |
| CFG Lock                               | **Disabled** if exposed; otherwise keep default (the `AppleXcpmCfgLock` quirk covers it) |
| Intel SGX                              | **Disabled** (if present) |

**Advanced → System Agent (SA) Configuration**

| Setting              | Value |
| -------------------- | ----- |
| Above 4G Decoding    | **Enabled** |
| Re-Size BAR Support  | **Disabled** (safest; enable only if your GPU/driver tolerates it) |

**Advanced → PCH Storage Configuration**

| Setting   | Value |
| --------- | ----- |
| SATA Mode | **AHCI** |

**Advanced → USB Configuration**

| Setting       | Value |
| ------------- | ----- |
| XHCI Hand-off | **Enabled** |

**Advanced → PCH-FW / Trusted Computing**

| Setting                                      | Value |
| -------------------------------------------- | ----- |
| Intel Platform Trust Technology (PTT / fTPM) | **Disabled** |

**Advanced → Onboard Devices Configuration**

| Setting                     | Value |
| --------------------------- | ----- |
| Serial Port / Parallel Port | **Disabled** (avoids macOS resource conflicts) |

Save with `F10`, then boot the USB / OpenCore entry and press **Reset NVRAM** once in the
OpenCore picker before installing.

---

## Installation / Deployment

### 1. Prepare a bootable macOS installer
Use the official `createinstallmedia` method or any trusted guide (see Links).

### 2. Mount the EFI (ESP) partition
On macOS:
```bash
# list disks and find your target ESP (e.g. diskNs1)
diskutil list
sudo diskutil mount diskNs1
```
Back up any existing `EFI/` folder first.

### 3. Copy this EFI
Copy the **entire `EFI/` folder** from this repo to the root of the EFI partition:
```
/Volumes/EFI/EFI/BOOT/BOOTx64.efi
/Volumes/EFI/EFI/OC/...
```

### 4. Configure the BIOS
Apply the settings in **[BIOS Settings](#bios-settings-asus-tx-gaming-b760m-wifi-d4)** above.

### 5. Boot
Select the OpenCore entry (or `BOOTx64.efi`) from your firmware boot menu. On first boot
use the OpenCore picker's **Reset NVRAM** entry once, then boot macOS.

> Note: `EFI/APPLE/` (Apple's DisplayPort firmware updater) exists on the author's live
> partition but is **not part of OpenCore** and has been omitted here — it is not needed.

---

## Graphics — NVIDIA RTX 3060 + NullMoth driver

macOS has **no native driver** for Turing/Ampere GPUs (GTX 16 / RTX 20 / 30 / 40 / 50).
Hardware acceleration for the RTX 3060 is provided by the **third-party NullMoth NVIDIA driver**.

### EFI side (already done)
- `config.plist` boot-args include the NullMoth switches (`nvfb=1 nvaccel=1 nvfbheads=4
  -nvkmsnosmooth nvfbsample=1 gfxrst=4`) and AMFI disables
  (`amfi_get_out_of_my_way=0x1 amfi=0x80`) required by the driver.
- `WhateverGreen.kext`, `NMHDA.kext` and `VoodooHDA_NV.kext` ship in `EFI/OC/Kexts`.
- `NullMothSafe.efi` is included under `EFI/OC/Tools` as a recovery/safe-mode helper.

### macOS side (you must install this)
1. Download the NullMoth NVIDIA driver package (version **2.6.4 or newer**) from the official site / repo (see Links).
2. Install from macOS:
   ```bash
   tar -xzf nullmoth-nvidia-<version>.tar.gz
   cd pkgroot
   sudo ./install.sh
   ```
3. Reboot. After install, the RTX 3060 provides accelerated rendering.

### Notes (NullMoth)
- **All audio works** through the bundled `VoodooHDA_NV.kext` (NullMoth's Voodoo HDA):
  onboard **Realtek ALC897** (7.1CH, SPDIF-out, Headphones) and **NVIDIA Audio** (HDMI/DP).
  Select any of them in **System Settings → Sound**. `AppleALC`/`NMHDA` are bundled but not
  needed on this build.
- **Chrome / Edge are fully GPU-accelerated** with NullMoth driver **2.6.4+**: WebGL and
  WebGL 2 work (ANGLE Metal Renderer on the RTX 3060). No CPU-rendering fallback on this
  version.

---

## Wi‑Fi / Bluetooth (Intel AX201)

On **macOS 26 Tahoe** the native Wi‑Fi menu is provided by **`AirportItlwm-Tahoe.kext`**;
**`itlwm.kext`** is also enabled as a companion/alternative (HeliPort app). The generic
`AirportItlwm.kext` is **disabled** on this build — do not enable it together with `itlwm`.
Bluetooth works via `IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup`.

> For macOS 15 Sequoia, use the `macos-15-sequoia` branch (that build uses the Sequoia-era
> itlwm/AirportItlwm set).

---

## Folder layout

```
EFI/
├── BOOT/
│   └── BOOTx64.efi
└── OC/
    ├── ACPI/      (SSDT-SBUS, SSDT-EC, SSDT-PLUG-ALT, SSDT-RTCAWAC, SSDT-USB-Reset, SSDT-USBX)
    ├── Drivers/   (HfsPlus, OpenCanopy, OpenRuntime, ResetNvramEntry)
    ├── Kexts/     (Lilu, VirtualSMC, WhateverGreen, VoodooHDA_NV, NMHDA, AppleALC,
    │               RTL812xLucy, AirportItlwm-Tahoe, itlwm, IntelBluetoothFirmware,
    │               USBToolBox, NVMeFix, CpuTopologyRebuild, ...)
    ├── Resources/ (OpenCanopy theme: Audio, Font, Image, Label)
    ├── Tools/     (NullMothSafe.efi)
    ├── OpenCore.efi
    └── config.plist  (SANITIZED — fill your SMBIOS before use)
```

---

## Links

- NullMoth NVIDIA driver: https://nullmothsystems.com · GitHub: https://github.com/nullmoth/nvidia-macos-driver
- Dortania OpenCore Install Guide: https://dortania.github.io/OpenCore-Install-Guide/
- OpenCore Auxiliary Tools (OCAT): https://github.com/ic005k/OCAT
- GenSMBIOS: https://github.com/corpnewt/GenSMBIOS
- HeliPort (itlwm Wi‑Fi app): https://github.com/OpenIntelWireless/HeliPort
- Intel Wi‑Fi/BT kexts: https://github.com/OpenIntelWireless
- This repository: https://github.com/Aiikisaraki/OpenCore-EFI-TX-GAMING-B760M-WIFI-D4

---

## Disclaimer

This project is provided "as is", without warranty of any kind. It is not affiliated with
or endorsed by Apple, ASUS, or NVIDIA. Follow your local laws. The author is not responsible
for any damage, data loss, or warranty void caused by using this configuration.
