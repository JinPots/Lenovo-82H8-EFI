# OpenCore EFI cho Lenovo 82H8

[Read in English](README.md)

Đây là OpenCore EFI của mình cho laptop Lenovo 82H8. EFI này được làm cho phần
cứng của mình, nên có thể không hoạt động nguyên trạng trên mọi phiên bản 82H8.

## Thông số laptop

- CPU: Intel Core i5-1135G7
- GPU: Intel Iris Xe Graphics
- RAM: 12 GB DDR4
- Wi-Fi/Bluetooth: Intel Wireless-AC 9560

## Các vấn đề hiện tại

- Hiệu ứng chuyển động còn bị lag.
- Sleep hiện chưa hoạt động.
- Apple Music hiện chưa hoạt động.

## Cài đặt nhanh

> **Quan trọng:** Trong giai đoạn cài đặt/khôi phục ban đầu, đồ họa có thể bị
> lag. WhateverXe cần
> [gói đồ họa này](https://github.com/b00t0x/WhateverXe-Graphics/releases),
> nhưng chỉ có thể cài sau khi macOS đã được thiết lập.

1. Sao lưu EFI hiện tại trước khi thay đổi.
2. Tạo USB cài macOS theo phương pháp thông thường của Apple/OpenCore.
3. Mount phân vùng EFI của USB.
4. Chép hai thư mục `BOOT` và `OC` từ repo này vào thư mục `EFI`:

   ```text
   EFI/
   ├── BOOT/
   └── OC/
   ```

5. Mở `OC/config.plist` bằng ProperTree và chạy **OC Snapshot** nếu đã thêm
   hoặc xóa file.
6. Chạy `ocvalidate` cùng phiên bản OpenCore với `OpenCore.efi`.
7. Boot trình cài đặt ở chế độ UEFI.

Xem hướng dẫn chi tiết hơn về cách lấy macOS Recovery trong
[OpenCore Install Guide](https://dortania.github.io/OpenCore-Install-Guide/).

Thư mục gốc của repo đã là phần payload của EFI. Đừng tạo thêm thư mục
`EFI/EFI/`.

## Việc cần làm trước khi sử dụng hằng ngày

Config đang dùng SMBIOS `MacBookPro16,2`, nhưng serial number, MLB, UUID và ROM
đang để trống. Hãy tự tạo các giá trị này bằng GenSMBIOS và điền vào
`PlatformInfo -> Generic`. Không dùng giá trị của người khác và không commit
chúng lên GitHub.

Các phần phụ thuộc nhiều vào phần cứng gồm:

- `alcid=13` cho âm thanh
- Properties framebuffer cho Intel Iris Xe
- VoodooI2C cho touchpad
- VoodooPS2 cho bàn phím
- USBToolBox cùng `UTBMap.kext`
- `ForbiddenItlwm` cho Wi-Fi Intel
- Các kext firmware và fixup cho Bluetooth Intel

Nếu thay card Wi-Fi, codec âm thanh, màn hình hoặc bố cục USB thì EFI này có thể
cũng cần được chỉnh sửa.

## Thiết lập BIOS

Tên tùy chọn có thể khác nhau tùy phiên bản BIOS, nhưng thường nên kiểm tra:

- Secure Boot: Disabled
- Fast Boot: Disabled
- CSM/Legacy boot: Disabled
- Boot mode: UEFI
- SATA mode: AHCI, nếu có
- CFG-Lock: Disabled, nếu có
- Internal Wi-Fi: Enabled

Nên giữ một EFI dự phòng hoặc USB khôi phục Windows. Việc debug Hackintosh sẽ
dễ hơn nhiều khi vẫn có thể boot vào hệ thống gốc.

## Các kext được sử dụng

Tất cả kext dưới đây đều được bật trong `OC/config.plist -> Kernel -> Add`.

| Kext | Phiên bản | Chức năng |
| --- | ---: | --- |
| [Lilu](https://github.com/acidanthera/Lilu) | 1.7.2 | Framework patch chính. |
| [VirtualSMC](https://github.com/acidanthera/VirtualSMC) | 1.3.8 | Cung cấp giao diện SMC mà macOS cần. |
| [VoodooI2C](https://github.com/VoodooI2C/VoodooI2C) | 2.9.1 | Hỗ trợ touchpad I2C, gồm VoodooGPIO, VoodooI2CServices và VoodooInput. |
| VoodooI2CHID | 1.0 | Hỗ trợ HID cho touchpad I2C. |
| [VoodooPS2Controller](https://github.com/acidanthera/VoodooPS2) | 2.3.7 | Hỗ trợ thiết bị PS/2, gồm plugin bàn phím. |
| AMFIPass | 1.4.1 | Hỗ trợ tương thích AMFI. |
| [AppleALC](https://github.com/acidanthera/AppleALC) | 1.9.8 | Hỗ trợ âm thanh; config này dùng layout 13. |
| BrightnessKeys | 1.0.3 | Phím tắt chỉnh độ sáng. |
| SMCBatteryManager | 1.3.8 | Thông tin pin và sạc. |
| SMCProcessor | 1.3.8 | Cảm biến SMC của CPU. |
| USBToolBox | 1.2.0 | Hỗ trợ map USB. |
| UTBMap | 1.1 | USB map cho laptop này. |
| [NVMeFix](https://github.com/acidanthera/NVMeFix) | 1.1.3 | Tăng độ ổn định và quản lý nguồn NVMe. |
| IntelBluetoothFirmware | 2.5.1 | Hỗ trợ firmware Bluetooth Intel. |
| BlueToolFixup | 2.7.2 | Sửa lỗi Bluetooth trên macOS mới. |
| BTLEAdvFixup | 0.9.0 | Hỗ trợ quảng bá Bluetooth LE/AirDrop. |
| [WhateverXe](https://github.com/b00t0x/WhateverXe) | 0.8.2 | Hỗ trợ đồ họa Intel Iris Xe. |
| ForbiddenItlwm | 0.5.1 | Hỗ trợ Wi-Fi Intel và một phần AirDrop. |

Trong config, `ForbiddenItlwm` và `BTLEAdvFixup` chỉ giới hạn cho kernel 25.x,
nên setup này hướng tới macOS 26 Tahoe. Nếu dùng phiên bản macOS khác,
hãy kiểm tra lại khả năng tương thích.

## ACPI và driver

Các SSDT đi kèm xử lý EC/nguồn USB, CPU plugin, đồng hồ AWAC, đèn nền và hành vi
`_OSI` của I2C:

- `SSDT-AWAC.aml`
- `SSDT-EC-USBX-LAPTOP.aml`
- `SSDT-PLUG-DRTNIA.aml`
- `SSDT-PNLF.aml`
- `SSDT-XOSI.aml`

Các UEFI driver gồm:

- `HfsPlus.efi`
- `OpenRuntime.efi`
- `OpenCanopy.efi`
- `ResetNvramEntry.efi`

## Ghi chú

Sau khi cài xong, hãy kiểm tra sleep, restart, shutdown, âm thanh, độ sáng, bàn
phím, touchpad, mọi cổng USB, Wi-Fi, Bluetooth và màn hình ngoài trước khi chép
EFI vào ổ trong.

Nếu có lỗi, thử theo thứ tự:

1. Boot EFI dự phòng.
2. Reset NVRAM từ OpenCore picker.
3. Hoàn tác thay đổi config hoặc kext gần nhất.
4. Boot verbose và kiểm tra log/panic.

Repo này chỉ chứa EFI, không chứa macOS hoặc bộ cài macOS.
