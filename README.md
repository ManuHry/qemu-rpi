# Ready-to-use Raspberry Pi QEMU commands

## Reference

- [Raspberry Pi boards (raspi0, raspi1ap, raspi2b, raspi3ap, raspi3b, raspi4b) — QEMU documentation](https://www.qemu.org/docs/master/system/arm/raspi.html)
- [Running Raspberry Pi OS on QEMU x64: Emulating a Pi on Your Ubuntu PC :: Martin Koníček - Personal site](https://www.martinkonicek.eu/posts/rpi-qemu/)
- [Emulating Raspberry Pi 4 with Qemu](https://gist.github.com/cGandom/23764ad5517c8ec1d7cd904b923ad863/)

> [!NOTE]
> Raspberry Pi 4 is not fully emulated in QEMU: it lacks PCI including USB devices!

## Original images download links

> [!NOTE]
> AlmaLinux 10 is not compatible with Raspberry Pi 3, as it support only MBR (not GPT)! Raspberry Pi OS 13 seems not to boot correctly by now.
>
> Links do not point to *\*-**latest**.\** files, explore parent folder to check if any new version is available.

- Raspberry Pi OS
  - [13 Lite](https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2025-12-04/2025-12-04-raspios-trixie-arm64-lite.img.xz) - 2.78 GB uncompressed
  - [12 Lite](https://downloads.raspberrypi.com/raspios_oldstable_lite_arm64/images/raspios_oldstable_lite_arm64-2025-11-24/2025-11-24-raspios-bookworm-arm64-lite.img.xz) - 2.56 GB uncompressed
  - [11 Lite](https://downloads.raspberrypi.com/raspios_oldstable_lite_arm64/images/raspios_oldstable_lite_arm64-2025-05-07/2025-05-06-raspios-bullseye-arm64-lite.img.xz) - 1.96 GB uncompressed
- AlmaLinux
  - [10 w/o GUI](https://repo.almalinux.org/almalinux/10/raspberrypi/images/AlmaLinux-10-RaspberryPi-gpt-10.1-20251201.aarch64.raw.xz) - 2.83 GB uncompressed
  - [9 w/o GUI](https://repo.almalinux.org/almalinux/9/raspberrypi/images/AlmaLinux-9-RaspberryPi-mbr-9.7-20251118.aarch64.raw.xz) - 2.83 GB uncompressed
  - [8 w/o GUI](https://repo.almalinux.org/almalinux/8/raspberrypi/images/AlmaLinux-8-RaspberryPi-mbr-8.10-20250331.aarch64.raw.xz) - 3.41 GB uncompressed - *yes, you read right*
- Rocky Linux
  - [10](https://dl.rockylinux.org/pub/rocky/10/images/aarch64/Rocky-10-SBC-RaspberryPi-10.1-20251122.2.aarch64.raw.xz) - 3.08 GB uncompressed
  - [9](https://dl.rockylinux.org/pub/sig/9/altarch/aarch64/images/RockyRpi_9.2.img.xz) - 3.52 GB uncompressed
  - [8](https://dl.rockylinux.org/pub/sig/8/altarch/aarch64/images/RockyRpi_8.8.img.xz) - 3.52 GB uncompressed

### Preparing images

> [!WARNING]
> It is highly recommended to change password as soon as possible!

1. Resize *\*.img* or *\*.raw* file to at least 4 GB:
   ```Shell
   qemu-img resize -f raw FILE_PATH 4G
   ```
2. Mount image and go to *0\*.fat* (first partition) to retrieve following files:
   - ***kernel8**.img* (Linux kernel)
   - ***initramfs-\*.el**\*.img* just for AlmaLinux 9+
   - ***bcm\*\*\*\*-rpi-\***.dtb* (Device tree, prefer use of bcm2710-rpi-3-b.dtb for Raspberry Pi 3 as it has better compatibility)
3. Assure remote access
   - For Raspberry Pi OS, to use `pi/raspberry` credentials, add following files in same path as kernel and DTB
     - ***ssh*** (empty file to activate sshd service)
     - ***userconfig**.txt* with following (encrypted password)
       ```
       pi:$6$c70VpvPsVNCG0YR5$l5vWWLsLko9Kj65gcQ8qvMkuOoRkEagI90qi3F/Y7rm8eNYZHW8CY6BOIKwMH7a3YYzZYL90zf304cAHLFaZE0
       ```
   - For AlmaLinux, to use `almalinux/almalinux` credentials, edit user-data in same path as kernel and DTB to allow SSH access with password:
     ```YAML
     ssh_pwauth: true
     ```

## How to emulate a Raspberry Pi

Simply use following commands (adapt paths to your needs), then:
- connect to localhost via SSH on port `2222`
- or use [https://localhost:9091](https://localhost:9091) if you installed [Cockpit](https://cockpit-project.org/):

> [!TIP]
> You can exit QEMU console with Ctrl+A then X, it will end emulation.

> [!NOTE]
> If you would like to kind-of emulate Rasperry Pi Zero 2 W, just limit memory in kernel adding `mem=512M` in `-append` argument. Use this only after preparing you OS to reduce time wasting, emulation and swap are slow!
> 
> Emulated CPU is identical: Cortex-A53 (4 cores), check [Raspberry Pi Zero 2 W product brief](https://datasheets.raspberrypi.com/rpizero2/raspberry-pi-zero-2-w-product-brief.pdf).

### Raspberry Pi OS 12

> [!NOTE]
> Since Debian 12 console should be sent to `ttyAMA1`.
> 
> `earlycon=pl011,0x3f201000` is also needed to get boot logs.

```Shell
qemu-system-aarch64 -nographic -machine raspi3b -kernel kernel8.img -dtb bcm2710-rpi-3-b.dtb -drive file=2025-11-24-raspios-bookworm-arm64-lite-passwd.img,format=raw,cache=writeback,aio=threads -netdev user,id=net0,hostfwd=tcp::2222-:22,hostfwd=tcp::9091-:9090 -device usb-net,netdev=net0 -append "earlycon=pl011,0x3f201000 console=ttyAMA1,115200 root=/dev/mmcblk0p2 rw rootwait dwc_otg.lpm_enable=0 dwc_otg.fiq_fsm_enable=0"
```

### Raspberry Pi OS 11
```Shell
qemu-system-aarch64 -nographic -machine raspi3b -kernel kernel8.img -dtb bcm2710-rpi-3-b.dtb -drive file=2025-05-06-raspios-bullseye-arm64-lite-passwd.img,format=raw,cache=writeback,aio=threads -netdev user,id=net0,hostfwd=tcp::2222-:22,hostfwd=tcp::9091-:9090 -device usb-net,netdev=net0 -append "console=ttyAMA0,115200 root=/dev/mmcblk0p2 rw rootwait dwc_otg.lpm_enable=0 dwc_otg.fiq_fsm_enable=0"
```

### AlmaLinux 10 *(slow, wait cloud-init to be complete)*

> [!CAUTION]
> Raspberry Pi 4 is needed, but network is not available by now! You will not gain any access, it will just boot and that is all...

```Shell
qemu-system-aarch64 -nographic -machine raspi4b -kernel kernel-6.12.47-20250916.v8.1.el10.img -initrd initramfs-6.12.47-20250916.v8.1.el10.img -dtb bcm2711-rpi-4-b.dtb -drive file=AlmaLinux-10-RaspberryPi-gpt-10.1-20251201.aarch64.raw,format=raw,cache=writeback,aio=threads -netdev user,id=net0,hostfwd=tcp::2222-:22,hostfwd=tcp::9091-:9090 -device usb-net,netdev=net0 -append "earlycon=pl011,0x3f201000 console=ttyAMA1,115200 root=PARTUUID=530e947f-26ce-402e-8562-a8c34939f03d rw rootwait dwc_otg.lpm_enable=0 dwc_otg.fiq_fsm_enable=0"
```

### AlmaLinux 9 *(slow, wait cloud-init to be complete)*

> [!NOTE]
> Since this version mounting initramfs is available.

```Shell
qemu-system-aarch64 -nographic -machine raspi3b -kernel kernel-6.12.47-20250916.v8.1.el9.img -initrd initramfs-6.12.47-20250916.v8.1.el9.img -dtb bcm2710-rpi-3-b.dtb -drive file=AlmaLinux-9-RaspberryPi-mbr-9.7-20251118.aarch64.raw,format=raw,cache=writeback,aio=threads -netdev user,id=net0,hostfwd=tcp::2222-:22,hostfwd=tcp::9091-:9090 -device usb-net,netdev=net0 -append "earlycon=pl011,0x3f201000 console=ttyAMA1,115200 root=/dev/mmcblk0p2 rw rootwait dwc_otg.lpm_enable=0 dwc_otg.fiq_fsm_enable=0"
```

### AlmaLinux 8 *(terribly slow, wait cloud-init to be complete)*

> [!NOTE]
> Mounting initramfs available for Rocky Linux 8+.

```Shell
qemu-system-aarch64 -nographic -machine raspi3b -kernel kernel-6.6.74-20250127.v8.1.el8.img -dtb bcm2710-rpi-3-b.dtb -drive file=AlmaLinux-8-RaspberryPi-mbr-8.10-20250331.aarch64.raw,format=raw,cache=writeback,aio=threads -netdev user,id=net0,hostfwd=tcp::2222-:22,hostfwd=tcp::9091-:9090 -device usb-net,netdev=net0 -append "earlycon=pl011,0x3f201000 console=ttyAMA1,115200 root=/dev/mmcblk0p2 rw rootwait dwc_otg.lpm_enable=0 dwc_otg.fiq_fsm_enable=0
```
