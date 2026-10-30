# EFI-igpu: Intel UHD (Raptor Lake) test EFI for the Acer Nitro V15

Test EFI for running the laptop on the Intel iGPU with NootedGreen, following
https://github.com/JulioNectar/Hackintosh-XPS13Plus-IrisXe-Tahoe (commit 6f03560).
The EFI on the internal disk (NVIDIA / nullmoth mode) is left as it was. Boot this one from a USB stick.

## How it differs from the NVIDIA EFI
- Kexts: + WhateverGreen 1.7.2, + NootedGreen (prebuilt from the XPS repo; replace it with your own build, see below)
- ACPI: + SSDT-PNLF (backlight, `_SB.PC00.GFX0`, same path as the XPS)
- DeviceProperties `PciRoot(0x0)/Pci(0x2,0x0)`: device-id / ig-platform-id 9A49 (Tiger Lake), as in the XPS config
- boot-args: `-nvoff -nmintelfboff` (the nullmoth NVIDIA driver and its Intel panel framebuffer stay off),
  `-wegnoegpu agdpmod=pikera`, the XPS NootedGreen arguments, and `ngreen-eu=48 ngreen-l3=4` for the 48 EU UHD
- csr-active-config `0x0A53` (XPS 0x0A13 + nullmoth 0x0A43)

The NVIDIA driver stays installed. With the internal EFI the laptop starts in NVIDIA mode as before.

## Steps
1. Format a USB stick as FAT32 (MBR or GPT) and copy the `EFI` folder to its root.
2. Restart, choose the USB stick in the Acer boot menu (F12), then macOS.
   Expected: a desktop with no acceleration yet (the Tiger Lake driver is not installed).
3. Check the iGPU spoof is active:
   `ioreg -r -c IOPCIDevice -n IGPU -d 1 | grep device-id` should show `<499a0000>`
   (if the device is called GFX0, use `-n GFX0`).
4. Still booted from the USB, install the user-space parts:
   ```sh
   git clone https://github.com/JulioNectar/Hackintosh-XPS13Plus-IrisXe-Tahoe.git ~/xe && cd ~/xe
   XE_FORCE=1 zsh install.sh      # XE_FORCE: macOS here is 26.7.2 (25G332), the repo targets 26.7.1 (25G241)
   ```
5. Copy the freshly built `~/xe/build/NootedGreen.kext` over `EFI/OC/Kexts/NootedGreen.kext` on the USB stick.
6. Restart from the USB again. Check:
   ```sh
   kmutil showloaded | grep -i -E 'NootedGreen|TGL'
   system_profiler SPDisplaysDataType | grep -E 'Chipset|Metal'
   ioreg -rc IntelAccelerator -d1 | grep xe-topo
   ```

## If something goes wrong
- Black screen or panic: pull the USB stick and start from the internal disk (NVIDIA mode).
- Artifacts (short dashes, broken textures): change `ngreen-l3=4` to `6` or `8`. The value for this chip is not
  known yet; the exact count is in register 0x9118 (see the XPS docs/INSTALL.md, read from a Linux live USB).
- To remove the iGPU stack: `cd ~/xe && zsh install.sh --uninstall`.

