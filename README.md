# Lenovo 82H8 OpenCore EFI

[Đọc bằng tiếng Việt](README.vi.md)

My OpenCore EFI for the Lenovo 82H8 laptop. This is made for my hardware, so
it may not work unchanged on every 82H8 variant.

## Laptop specs

- CPU: Intel Core i5-1135G7
- GPU: Intel Iris Xe Graphics
- RAM: 12 GB DDR4
- Wi-Fi/Bluetooth: Intel Wireless-AC 9560

## Current issues

- Animations are laggy.
- Sleep is currently broken.
- Apple Music is currently broken.

## Quick setup

> **Important:** During the initial setup/recovery phase, graphics may be laggy.
> WhateverXe needs its
> [graphics package](https://github.com/b00t0x/WhateverXe-Graphics/releases),
> which can only be installed after macOS is set up.

1. Make a backup of your current EFI before changing anything.
2. Create a macOS installer USB using the normal Apple/OpenCore method.
3. Mount the USB's EFI partition.
4. Copy the `BOOT` and `OC` folders from this repo into the partition's `EFI`
   folder:

   ```text
   EFI/
   ├── BOOT/
   └── OC/
   ```

5. Open `OC/config.plist` with ProperTree and run **OC Snapshot** if files were
   added or removed.
6. Run `ocvalidate` from the same OpenCore release as `OpenCore.efi`.
7. Boot the installer in UEFI mode.

For further instructions on getting macOS Recovery, follow the
[OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/).

The repo root is already the EFI payload. Do not create an extra
`EFI/EFI/` folder.

## Important changes before daily use

The config uses `MacBookPro16,2` as its SMBIOS, but the serial number, MLB, UUID,
and ROM are empty. Generate your own values with GenSMBIOS and fill them under
`PlatformInfo -> Generic`. Never use someone else's values or commit them to
GitHub.

The main hardware-specific parts are:

- `alcid=13` for audio
- Intel Iris Xe framebuffer properties
- VoodooI2C for the touchpad
- VoodooPS2 for the keyboard
- USBToolBox plus the included `UTBMap.kext`
- `ForbiddenItlwm` for Intel Wi-Fi/AirDrop
- Intel Bluetooth firmware and fixup kexts

If you change the Wi-Fi card, audio codec, display, or USB layout, this EFI will
probably need changes too.

## BIOS settings

Names vary by BIOS, but these are usually worth checking:

- Secure Boot: Disabled
- Fast Boot: Disabled
- CSM/Legacy boot: Disabled
- Boot mode: UEFI
- SATA mode: AHCI, if available
- CFG-Lock: Disabled, if available
- Internal Wi-Fi: Enabled

Keep a backup EFI or Windows recovery USB nearby. Hackintosh debugging is much
less stressful when you can still boot the original system.

## Kexts

All of these are enabled in `OC/config.plist -> Kernel -> Add`.

| Kext | Version | What it does |
| --- | ---: | --- |
| [Lilu](https://github.com/acidanthera/Lilu) | 1.7.2 | Main patching framework. |
| [VirtualSMC](https://github.com/acidanthera/VirtualSMC) | 1.3.8 | Provides the SMC interface macOS expects. |
| [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) | 2.9.1 | I2C touchpad support. Includes VoodooGPIO, VoodooI2CServices, and VoodooInput. |
| VoodooI2CHID | 1.0 | HID support for the I2C touchpad. |
| [VoodooPS2Controller](https://github.com/acidanthera/VoodooPS2) | 2.3.7 | PS/2 input support, including its keyboard plugin. |
| AMFIPass | 1.4.1 | AMFI compatibility support. |
| [AppleALC](https://github.com/acidanthera/AppleALC) | 1.9.8 | Audio support; this config uses layout 13. |
| BrightnessKeys | 1.0.3 | Brightness hotkeys. |
| SMCBatteryManager | 1.3.8 | Battery and charging information. |
| SMCProcessor | 1.3.8 | Processor SMC sensors. |
| USBToolBox | 1.2.0 | USB mapping support. |
| UTBMap | 1.1 | USB map for this laptop. |
| [NVMeFix](https://github.com/acidanthera/NVMeFix) | 1.1.3 | NVMe stability and power management. |
| IntelBluetoothFirmware | 2.5.1 | Intel Bluetooth firmware support. |
| BlueToolFixup | 2.7.2 | Bluetooth fixes for newer macOS versions. |
| BTLEAdvFixup | 0.9.0 | Bluetooth LE advertising/AirDrop support. |
| [WhateverXe](https://github.com/b00t0x/WhateverXe) | 0.8.2 | Intel Iris Xe graphics support. |
| ForbiddenItlwm | 0.5.1 | Intel Wi-Fi and related AirDrop support. |

`ForbiddenItlwm` and `BTLEAdvFixup` are limited in the config to kernel 25.x,
so this setup is aimed at Tahoe-era macOS. Check compatibility if using another
macOS version.

## ACPI and drivers

The included SSDTs handle the laptop EC/USB power, CPU plugin, AWAC clock,
backlight, and I2C `_OSI` behavior:

- `SSDT-AWAC.aml`
- `SSDT-EC-USBX-LAPTOP.aml`
- `SSDT-PLUG-DRTNIA.aml`
- `SSDT-PNLF.aml`
- `SSDT-XOSI.aml`

The UEFI drivers are:

- `HfsPlus.efi`
- `OpenRuntime.efi`
- `OpenCanopy.efi`
- `ResetNvramEntry.efi`

## Notes

After installation, test sleep, restart, shutdown, audio, brightness, keyboard,
touchpad, every USB port, Wi-Fi, Bluetooth, and external displays before copying
the EFI to the internal drive.

If something breaks, try this order:

1. Boot the backup EFI.
2. Reset NVRAM from the OpenCore picker.
3. Undo the latest config or kext change.
4. Boot with verbose output and check the panic/log output.

This repo contains the EFI only. It does not include macOS or an installer.
