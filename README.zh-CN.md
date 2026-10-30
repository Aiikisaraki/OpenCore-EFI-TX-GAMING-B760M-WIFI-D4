# OpenCore EFI —— 华硕 TX GAMING B760M WIFI D4（i5-12600KF + RTX 3060）

一套可直接使用的 OpenCore EFI，适用于 **华硕 TX GAMING B760M WIFI D4** 主板，
搭配 **Intel 酷睿 i5-12600KF（Alder Lake）** 与 **NVIDIA RTX 3060（12GB）**，
用于黑苹果（Hackintosh）安装 macOS。

> 本 EFI 由 **NullMoth「1401 for Windows」** 版 OpenCore 生成，针对 **macOS 15 Sequoia**
> 调校（主力、已测试）。另有一份实验性分支/变体，额外提供适用于 macOS 26 Tahoe 的
> AirportItlwm 构建。

---

## ⚠️ 开始前的重要说明

1. **本仓库中的 `config.plist` 已做脱敏处理。** 原始的 SMBIOS 序列号、MLB、SystemUUID
   与 ROM 已被清空，以保护作者隐私并避免序列号冲突。**首次启动前你必须自己生成 SMBIOS**
   （见[生成你自己的 SMBIOS](#生成你自己的-smbios)），否则 macOS / iServices 无法正常工作，
   且可能与他人序列号冲突。
2. 这是第三方、社区维护的配置，**风险自负**。错误的 EFI 可能导致机器无法启动——
   请务必在一根独立的 U 盘上保留一份可用的备份 EFI。
3. 显卡硬件加速依赖**第三方 NullMoth NVIDIA 驱动**，并非苹果官方代码。请仔细阅读
   [显卡章节](#显卡--nvidia-rtx-3060--nullmoth-驱动)。

---

## 硬件配置

| 部件           | 型号                                                              |
| -------------- | ----------------------------------------------------------------- |
| 主板           | 华硕 TX GAMING B760M WIFI D4（B760，LGA1700，Raptor Lake PCH）     |
| BIOS / SMBIOS  | MacPro7,1（SecureBootModel：Disabled）                            |
| CPU            | Intel 酷睿 i5-12600KF（Alder Lake-S，无核显）                      |
| 显卡           | NVIDIA GeForce RTX 3060 12GB（Ampere）—— 经 NullMoth 驱动加速     |
| 内存           | DDR4（请在自己的 SMBIOS / SystemMemoryStatus 中设置）             |
| 有线网卡       | 瑞昱 RTL8125 2.5GbE —— `RTL812xLucy.kext`                         |
| Wi‑Fi / 蓝牙   | 英特尔 AX201（CNVi）—— `itlwm.kext` + `IntelBluetoothFirmware`    |
| 声卡           | 瑞昱 ALC897 —— `AppleALC.kext` + NullMoth `NMHDA.kext` / `VoodooHDA_NV.kext` |
| 存储           | NVMe SSD —— `NVMeFix.kext`                                        |

### 启动参数（已配置好）

```
-v debug=0x100 nvfb=1 nvaccel=1 nvfbheads=4 -nvkmsnosmooth
amfi_get_out_of_my_way=0x1 amfi=0x80 ipc_control_port_options=0
nvfbsample=1 gfxrst=4 -nvoff
```

---

## 功能状态

| 功能                 | 状态 / 说明                                                  |
| -------------------- | ----------------------------------------------------------- |
| macOS 15 Sequoia     | ✅ 已测试                                                    |
| macOS 26 Tahoe       | ⚠️ 实验性（请使用带 Sequoia/Tahoe AirportItlwm 的变体）     |
| CPU 电源管理         | ✅ `CpuTopologyRebuild.kext`、`SMCProcessor`、`SMCSuperIO`   |
| 有线网（2.5G）       | ✅ `RTL812xLucy.kext`                                        |
| Wi‑Fi（AX201）       | ✅ `itlwm.kext` + HeliPort 应用（Sequoia）                   |
| 蓝牙（AX201）        | ✅ `IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup` |
| 音频（ALC897）       | ✅ `AppleALC.kext` + NullMoth 音频 kext                      |
| USB（定制）          | ✅ `USBToolBox.kext` + `UTBDefault.kext` + SSDT-USBX         |
| NVIDIA RTX 3060      | ✅ 经 NullMoth 驱动加速；✅ HDMI/DP 音频（`VoodooHDA_NV`）    |

---

## 生成你自己的 SMBIOS

仓库自带的 `config.plist` 序列号为空。请生成一个合法的 `MacPro7,1` 身份：

1. 下载 **GenSMBIOS**（或用 **OCAT** → `PlatformInfo` → `Generate`）。
2. 选择机型 `MacPro7,1`，生成：
   - `SystemSerialNumber`
   - `MLB`
   - `SystemUUID`
   - `ROM`（通常为 en0 的 MAC 地址，6 字节）
3. 打开 `EFI/OC/config.plist` → `PlatformInfo` → `Generic`，填入上述四项。

> 切勿使用他人的序列号。请妥善保管自己的序列号——不要公开你生成的 `config.plist`。

---

## 安装 / 部署方法

### 1. 准备可启动的 macOS 安装盘
使用官方 `createinstallmedia` 方法或任何可信教程（见链接）。

### 2. 挂载 EFI（ESP）分区
在 macOS 上：
```bash
# 列出磁盘，找到目标 ESP（例如 diskNs1）
diskutil list
sudo diskutil mount diskNs1
```
先备份原有的 `EFI/` 文件夹。

### 3. 复制本 EFI
将仓库中的**整个 `EFI/` 文件夹**复制到 EFI 分区根目录：
```
/Volumes/EFI/EFI/BOOT/BOOTx64.efi
/Volumes/EFI/EFI/OC/...
```

### 4. BIOS 设置（推荐）
| 选项                       | 值                 |
| -------------------------- | ------------------ |
| 快速启动（Fast Boot）      | 关闭               |
| 安全启动（Secure Boot）    | 关闭               |
| CSM / 兼容支持模块         | 关闭（纯 UEFI）    |
|  Above 4G Decoding         | 开启               |
| SATA 模式                  | AHCI               |
| XHCI Hand-off              | 开启               |
| VT-d                       | 开启*（已由 quirk 处理） |
| CFG Lock                   | 尽量关闭（已启用 `AppleXcpmCfgLock` quirk 作为兜底） |

\* 已配置 `DisableIoMapper` 等相关 quirk，保留 VT-d 开启无碍。

### 5. 启动
在主板启动菜单中选择 OpenCore 引导项（或 `BOOTx64.efi`）。首次启动时，先在
OpenCore 选择界面用一次 **Reset NVRAM**，再启动 macOS。

---

## 显卡 —— NVIDIA RTX 3060 + NullMoth 驱动

macOS **没有** Turing/Ampere 显卡（GTX 16 / RTX 20 / 30 / 40 / 50）的原生驱动。
RTX 3060 的硬件加速由**第三方 NullMoth NVIDIA 驱动**提供。

### EFI 侧（已完成）
- `config.plist` 的启动参数已包含 NullMoth 所需开关（`nvfb=1 nvaccel=1 nvfbheads=4
  -nvkmsnosmooth nvfbsample=1 gfxrst=4 -nvoff`）以及 AMFI 禁用
  （`amfi_get_out_of_my_way=0x1 amfi=0x80`）。
- `WhateverGreen.kext` 与 `NMHDA.kext` / `VoodooHDA_NV.kext` 已随 `EFI/OC/Kexts` 提供。
- `NullMothSafe.efi` 已放入 `EFI/OC/Tools`，作为恢复/安全模式辅助工具。

### macOS 侧（需自行安装）
1. 从官网 / 仓库下载 NullMoth NVIDIA 驱动包（见链接）。
2. 在 macOS 中安装：
   ```bash
   tar -xzf nullmoth-nvidia-<version>.tar.gz
   cd pkgroot
   sudo ./install.sh
   ```
3. 重启。安装完成后，RTX 3060 即可提供加速渲染。

### 已知说明（NullMoth）
- **HDMI / DisplayPort 音频在本机是可用的**，通过随附的 `VoodooHDA_NV.kext`
  （NullMoth 的 Voodoo HDA）提供。输出设备显示为 **“NVIDIA Audio”**
  （厂商：Voodoo，2 声道，44.1 kHz），在 **系统设置 → 声音** 中选择即可。
- **Chrome / Edge 默认回退到 CPU 渲染**（这些浏览器无 GPU 合成）；Safari 及多数原生
  应用可正常使用 GPU。

---

## Wi‑Fi / 蓝牙（Intel AX201）

在 **macOS 15 Sequoia** 下，原生 `AirportItlwm` 仅支持到 Sonoma，因此本 EFI 使用
**`itlwm.kext` + HeliPort 应用** 实现 Wi‑Fi。**切勿**同时加载 `itlwm` 与 `AirportItlwm`。
对于 macOS 26 Tahoe，请使用附带 `AirportItlwm-Tahoe.kext` 的变体。蓝牙由
`IntelBluetoothFirmware` + `IntelBTPatcher` + `BlueToolFixup` 提供。

---

## 目录结构

```
EFI/
├── BOOT/
│   └── BOOTx64.efi
└── OC/
    ├── ACPI/      (SSDT-SBUS、SSDT-EC、SSDT-PLUG-ALT、SSDT-RTCAWAC、SSDT-USB-Reset、SSDT-USBX)
    ├── Drivers/   (HfsPlus、OpenCanopy、OpenRuntime、ResetNvramEntry)
    ├── Kexts/     (Lilu、VirtualSMC、AppleALC、WhateverGreen、NMHDA、VoodooHDA_NV、
    │               RTL812xLucy、itlwm、IntelBluetoothFirmware、USBToolBox、NVMeFix 等)
    ├── Resources/ (OpenCanopy 主题：Audio、Font、Image、Label)
    ├── Tools/     (NullMothSafe.efi)
    ├── OpenCore.efi
    └── config.plist  （已脱敏 —— 使用前请填入你的 SMBIOS）
```

---

## 相关链接

- NullMoth NVIDIA 驱动：https://nullmothsystems.com · GitHub：https://github.com/nullmoth/nvidia-macos-driver
- Dortania OpenCore 安装指南：https://dortania.github.io/OpenCore-Install-Guide/
- OpenCore 辅助工具 OCAT：https://github.com/ic005k/OCAT
- GenSMBIOS：https://github.com/corpnewt/GenSMBIOS
- HeliPort（itlwm Wi‑Fi 应用）：https://github.com/OpenIntelWireless/HeliPort
- 英特尔 Wi‑Fi/蓝牙 kext：https://github.com/OpenIntelWireless
- 本仓库：https://github.com/Aiikisaraki/OpenCore-EFI-TX-GAMING-B760M-WIFI-D4

---

## 免责声明

本项目按“现状”提供，不附带任何担保，与 Apple、华硕、NVIDIA 无隶属或背书关系。
请遵守当地法律法规。因使用本配置导致的任何损坏、数据丢失或保修失效，作者不承担责任。
