# OpenCore EFI - Acer Nitro V15 (ANV15-51)

Koleksi konfigurasi OpenCore EFI untuk laptop **Acer Nitro V15 (ANV15-51)** yang menjalankan macOS (Sonoma / Sequoia / Tahoe). 

Repositori ini menyediakan **2 varian EFI**:
1. **`EFI-Nvidia`**: Konfigurasi utama dengan akselerasi grafis dGPU **NVIDIA GeForce RTX 2050** via NullMoth (`nvaccel`/`nvfb`, Metal 3).
2. **`EFI-igpu`**: Konfigurasi eksperimental untuk iGPU **Intel UHD Graphics (Raptor Lake)** menggunakan NootedGreen (spoof Tiger Lake `0x9A49`).

---

## Spesifikasi Perangkat Keras

| Komponen | Spesifikasi | Keterangan |
| :--- | :--- | :--- |
| **Model** | Acer Nitro V15 (ANV15-51) | Laptop Gaming |
| **Prosesor (CPU)** | Intel Core i5-13420H (Raptor Lake) | 8 Core (4P + 4E) / 12 Threads |
| **Memori (RAM)** | 24 GB DDR5 4800/5200 MHz | Dual Channel |
| **iGPU** | Intel® UHD Graphics (48 EU, Raptor Lake GT1) | Device ID `0xa7a8` |
| **dGPU** | NVIDIA GeForce RTX 2050 Mobile | 4 GB GDDR6 (GA107), Device ID `0x25ad` |
| **Layar** | 15.6" Full HD (1920x1080) IPS | 144Hz Refresh Rate |
| **Penyimpanan** | NVMe SSD M.2 PCIe Gen 4 | NVMeFix |
| **Audio** | Realtek Audio | AppleALC (`alcid=11`) + SimpleHDA |
| **Ethernet** | Realtek RTL8168/8111 PCI-E Gigabit | RealtekRTL8111.kext |
| **Wireless / BT** | Intel Wi-Fi / MTK | itlwm.kext + HeliPort |
| **Trackpad / Keyboard**| I2C HID Trackpad + PS/2 Keyboard | VoodooI2C + VoodooPS2Controller |
| **Bootloader** | OpenCore | SMBIOS `MacBookPro16,1` |

---

## Status Fitur (Hardware Support)

### Mode NVIDIA (`EFI-Nvidia`)
- [x] Booting macOS & Graphic Acceleration (Metal 3 via `nvaccel` & `nvfb`)
- [x] Internal Display 144Hz FHD
- [x] Audio Output (Speaker internal & Headphone jack via SimpleHDA / AppleALC)
- [x] Mikrofon Internal
- [x] Keyboard internal & Tombol Pengatur Kecerahan (`BrightnessKeys.kext`)
- [x] Trackpad I2C (Gesture multi-touch)
- [x] Port Ethernet (LAN RJ45)
- [x] Wi-Fi via `itlwm` (Gunakan aplikasi [HeliPort](https://github.com/OpenIntelWireless/HeliPort))
- [x] Indikator Baterai (`ECEnabler` + `SMCBatteryManager`)
- [x] Sleep, Wake, Restart, Shutdown
- [x] USB Port Mapping dasar

### Mode iGPU (`EFI-igpu`)
- [x] Booting macOS dengan NootedGreen spoofing
- [x] Backlight Control (`SSDT-PNLF`)
- [x] Audio, Trackpad, Wi-Fi, Ethernet, Battery sama dengan mode utama
- [!] Akselerasi grafis iGPU memerlukan patch user-space (lihat [README-igpu](EFI-igpu/README-igpu.md))

---

## Struktur Folder Repositori

```text
EFI-Acer-Nitro-V15/
├── EFI-Nvidia/                 # Varian Utama: Akselerasi dGPU RTX 2050
│   ├── BOOT/
│   │   └── BOOTx64.efi
│   └── OC/
│       ├── ACPI/              # SSDT-EC-USBX, SSDT-PMC, SSDT-PLUG-ALT
│       ├── Drivers/           # OpenRuntime, OpenHfsPlus, ResetNvramEntry
│       ├── Kexts/             # Lilu, VirtualSMC, VoodooI2C, SimpleHDA, dll.
│       ├── Tools/             # OpenShell, NullMothSafe
│       └── config.plist
├── EFI-igpu/                   # Varian Eksperimental: Intel UHD (NootedGreen)
│   ├── README-igpu.md          # Petunjuk setup khusus user-space NootedGreen
│   └── EFI/
│       ├── BOOT/
│       └── OC/
└── README.md
```

---

## Pengaturan BIOS (Acer Nitro V15)

Sebelum melakukan booting ke installer macOS atau OpenCore, atur opsi BIOS (tekan `F2` saat menyalakan laptop):

1. **Atur Supervisor Password**: Di tab **Security**, set `Set Supervisor Password` sementara agar opsi tersembunyi/terkunci dapat diubah.
2. **Disable Secure Boot**: Pada tab **Security / Boot**, ubah `Secure Boot` menjadi **Disabled**.
3. **SATA / VMD Controller**: Ubah mode storage controller dari **RST with Optane / VMD** ke **AHCI** (jika ada).
4. **Enable F12 Boot Menu**: Di tab **Main**, aktifkan opsi `F12 Boot Menu`.
5. **Disable Fast Boot**: Di tab **Boot**, nonaktifkan `Fast Boot`.
6. Simpan konfigurasi dan keluar (`F10`).

---

## Cara Pemasangan ke USB Flashdisk

1. Siapkan flashdisk berkapasitas minimal 16 GB (jika bersama installer macOS) atau flashdisk FAT32 untuk testing EFI.
2. Format partisi EFI sebagai **FAT32**.
3. Pilih salah satu varian EFI:
   - **Untuk Mode NVIDIA (Rekomendasi)**:
     Salin folder `BOOT` dan `OC` dari dalam folder `EFI-Nvidia/` ke dalam direktori `EFI/` pada partisi EFI flashdisk Anda:
     ```text
     /Volumes/EFI/
     └── EFI/
         ├── BOOT/
         │   └── BOOTx64.efi
         └── OC/
             ├── ACPI/
             ├── Drivers/
             ├── Kexts/
             └── config.plist
     ```
   - **Untuk Mode iGPU**:
     Salin seluruh folder `EFI` yang ada di dalam `EFI-igpu/` langsung ke root partisi EFI flashdisk Anda.
4. Tancapkan USB ke Acer Nitro V15, nyalakan dan tekan `F12` berulang kali, lalu pilih flashdisk OpenCore Anda.

---

## Catatan Penting Mengenai SMBIOS

> [!WARNING]
> Sangat disarankan untuk **mengganti Serial Number, Board Serial, dan SmUUID** sebelum menghubungkan laptop ke akun Apple ID / iCloud pribadi Anda.

Gunakan tool [GenSMBIOS](https://github.com/corpnewt/GenSMBIOS) dengan target model:
```text
MacBookPro16,1
```
Masukkan nilai yang dihasilkan ke dalam file `config.plist` di bagian `PlatformInfo` -> `Generic`.

---

## Ucapan Terima Kasih & Kredit

- [Apple](https://www.apple.com/) untuk sistem operasi macOS.
- [Acidanthera](https://github.com/acidanthera) untuk OpenCorePkg, Lilu, VirtualSMC, AppleALC, VoodooPS2, WhateverGreen, dan kext penting lainnya.
- Pengembang **NullMoth** & komunitas untuk patch driver akselerasi NVIDIA modern.
- Pengembang **NootedGreen** ([ChefKissInc](https://github.com/ChefKissInc)) & [JulioNectar](https://github.com/JulioNectar/Hackintosh-XPS13Plus-IrisXe-Tahoe) untuk riset akselerasi Intel Iris Xe / UHD generasi ke-12 dan ke-13.
- [OpenIntelWireless](https://github.com/OpenIntelWireless) untuk `itlwm` dan `HeliPort`.
- [VoodooI2C Team](https://github.com/VoodooI2C/VoodooI2C) untuk dukungan trackpad multi-touch I2C.
