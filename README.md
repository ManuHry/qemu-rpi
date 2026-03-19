# Ready-to-use QEMU commands for Raspberry Pi 3 emulation

## Reference

- [Raspberry Pi boards (raspi0, raspi1ap, raspi2b, raspi3ap, raspi3b, raspi4b) — QEMU documentation](https://www.qemu.org/docs/master/system/arm/raspi.html)
- [Running Raspberry Pi OS on QEMU x64: Emulating a Pi on Your Ubuntu PC :: Martin Koníček - Personal site](https://www.martinkonicek.eu/posts/rpi-qemu/)
- [Emulating Raspberry Pi 4 with Qemu](https://gist.github.com/cGandom/23764ad5517c8ec1d7cd904b923ad863/)

## Original images download links

> [!NOTE]
> AlmaLinux 10 is not compatible with Raspberry Pi 3, as it support only MBR (not GPT)!
>
> Raspberry Pi OS 13 seems not to boot correctly by now.

- [Raspberry Pi OS 12 lite](https://downloads.raspberrypi.com/raspios_oldstable_lite_arm64/images/raspios_oldstable_lite_arm64-2025-11-24/2025-11-24-raspios-bookworm-arm64-lite.img.xz)
- [Raspberry Pi OS 11 lite](https://downloads.raspberrypi.com/raspios_oldstable_lite_arm64/images/raspios_oldstable_lite_arm64-2025-05-07/2025-05-06-raspios-bullseye-arm64-lite.img.xz)
- [AlmaLinux 9 - no GUI](https://repo.almalinux.org/almalinux/9/raspberrypi/images/AlmaLinux-9-RaspberryPi-mbr-9.7-20251118.aarch64.raw.xz)
- [AlmaLinux 8 - no GUI](https://repo.almalinux.org/almalinux/8/raspberrypi/images/AlmaLinux-8-RaspberryPi-mbr-8.10-20250331.aarch64.raw.xz)

### Preparing images

> [!WARNING]
> It is highly recommended to change password as soon as possible!

1. Resize *.img or *.raw file to at least 4 GB:
   ```Shell
   qemu-img resize -f raw FILE_PATH 4G
   ```
2. Mount image and go to '0*.fat' to retrieve following files:
   - kernel8.img (Linux kernel)
   - bcm****-rpi-*.dtb (Device tree, for Raspberry Pi 3 use bcm2710-rpi-3-b.dtb)
3. For Raspberry Pi OS, add following files in /boot/ (same path as kernel and DTB)
   - ssh (empty file)
   - userconfig.txt with following
        ```
        pi:$6$c70VpvPsVNCG0YR5$l5vWWLsLko9Kj65gcQ8qvMkuOoRkEagI90qi3F/Y7rm8eNYZHW8CY6BOIKwMH7a3YYzZYL90zf304cAHLFaZE0
        ```

## How to emulate Raspberry Pi 3

> [!NOTE]
> Raspberry Pi 4 is not fully emulated in QEMU, and for example lacks PCI including USB devices!

Simply use following commands (adapt paths to your needs), then connect to localhost via SSH on port 2222:

> [!TIP]
> You can exit QEMU console with Ctrl+A then X, it will end emulation.

### Raspberry Pi OS 12

> [!NOTE]
> Since Debian 12 console should be sent to ttyAMA1.
>
> *earlycon=pl011,0x3f201000* is also needed to get boot logs.

```Shell
qemu-system-aarch64 -nographic -machine raspi3b -kernel deb12\kernel8.img -dtb deb12\bcm2710-rpi-3-b.dtb -drive file=deb12\2025-11-24-raspios-bookworm-arm64-lite-passwd.img,format=raw -netdev user,id=net0,hostfwd=tcp::2222-:22 -device usb-net,netdev=net0 -append "earlycon=pl011,0x3f201000 console=ttyAMA1,115200 root=/dev/mmcblk0p2 rw rootwait dwc_otg.lpm_enable=0 dwc_otg.fiq_fsm_enable=0"
```

### Raspberry Pi OS 11
```Shell
qemu-system-aarch64 -nographic -machine raspi3b -kernel deb11\kernel8.img -dtb deb11\bcm2710-rpi-3-b.dtb -drive file=deb11\2025-05-06-raspios-bullseye-arm64-lite-passwd.img,format=raw -netdev user,id=net0,hostfwd=tcp::2222-:22 -device usb-net,netdev=net0 -append "console=ttyAMA0,115200 root=/dev/mmcblk0p2 rw rootwait dwc_otg.lpm_enable=0 dwc_otg.fiq_fsm_enable=0"
```
