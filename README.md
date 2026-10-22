# UniXNU-braveark

Device repository for running [UniXNU](https://github.com/UniXNU/UniXNU) on the
**Brave Ark** tablet (KU1381, Qualcomm SM8635 "cliffsp", QRD-based board).

Derived from [Allenwdk/UniXNU-uke](https://github.com/Allenwdk/UniXNU-uke)
(Xiaomi Pad 7, SM7675). Both SoCs are Palawan, and every SoC-level value the
kernel side uses was checked against the Brave Ark stock device tree
(`dtb_idx 4` CliffsP + `dtbo_idx 24` Cliffs QRD) and is the same as uke's:
GICv3 `0x17100000`/`0x17180000`, UFS `0x1d84000` (PHY `0x1d80000`, SMMU stream
`0x60`), the 19.2 MHz timer, the `cont_splash_region` at `0xe3940000`, the
`llcc_lpi` panic-log region at `0xff800000` and the vendor ramdisk at
`0x1000000`. The device identity is `brave,braveark` / `qcom,cliffsp`.

Internal names from the uke and sheng ports (`-uketextconsole`, `sheng-*`
boot arguments, `UKE:` log prefixes, `UkeBootMemoryDxe`) are kept on purpose:
the kernel patches and kits refer to them.

## Status

| Stage | State |
|---|---|
| 1. UEFI firmware (Mu-Silicium `bravearkPkg`) | works via `fastboot boot` |
| 2. UniXNU menu | works |
| 3. Payload | works from `vendor_boot` (see below) |
| 4. DarwinBoot / 5. XNU entry | next: first kernel from the macOS workflow |

## Hardware data

- 12 GB RAM, UFS, Android 16 on a vendor 14 BSP, slot `_a`, unlocked bootloader.
- `userdata`: `sda12`, 4 KiB LBA `0x24c628`–`0x3a8b7fa` (225 GiB).
- Panel: 1840×2880 video-mode DSC (`Panel_KU1380_1840_2880_dsc.xml`).
- Touch and pen: Novatek `NVT-ts-spi` on GENI SPI `0xa90000`, IRQ GPIO 129.
- GPU: Adreno 735 (`qcom,adreno-gpu-gen7-11-0`); the native GPU driver knows only the Adreno 740.

## Boot path

The Brave ABL takes the generic ramdisk from `init_boot` for boot image v4 and
ignores the ramdisk of a `fastboot boot` image, and `init_boot` is only 8 MiB.
So, as on uke, the UniXNU payload is the **vendor ramdisk of `vendor_boot`**
(96 MiB), built from the device's own `vendor_boot` with
`scripts/patch-vendor-boot.py`, while the UEFI image is started with
`fastboot boot`. Back up `vendor_boot` first (`fastboot reboot fastboot`,
then `fastboot fetch vendor_boot_a`); flashing that backup restores Android.

## Building

- Kernel: `.github/workflows/build-kernel.yml` on GitHub's macOS runners
  (Xcode is required), against `Allenwdk/UniXNU` branch `uke`.
- Firmware, payload and boot image: Linux, with a Mu-Silicium checkout that has
  the `bravearkPkg` platform (`unixnu build firmware --uefi …`,
  `unixnu bootimg --dtb <braveark.dtb> --ramdisk none`).

No Apple, Qualcomm or Brave firmware, kernelcaches or images belong in this
repository.
