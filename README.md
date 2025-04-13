# pinenote-arch-stuff
Tools, PKGBUILD and others miscellanous stuff to make arch works on PineNote

## Initial Setup
Initial debian -> arch conversion can be made using hrdl's method here:
https://github.com/hrdl-github/pinenote-arch/tree/main

## Kernel Installation
[](/linux-pinenote) contains a PKGBUILD to build the kernel, as well as other necessary file
to automate initramfs generation.

I'd suggest setting up distcc to build it

The packages it generate install the kernel in the following way:
```
/boot/Image{,gz} # vmlinux image
/boot/dtbs # Device tree blobs
/etc/mkinitcpio.d/linux-pinenote.preset # mkinitcpio preset, for initramfs generation
/usr/lib/initcpio/6.12.11-1-pinenote # used to trigger mkinitcpio in pacman hooks
/usr/lib/modules/6.12.11-1-pinenote/ # kernel modules
```

### Initramfs Configuration
The default configuration for mkinitcpio lacks one module for a correct boot: `gpio-rockchip`

Edit the mkinitcpio and add it to the MODULES list:
```
MODULES=(gpio-rockchip)
```

Generate the initramfs with the correct modules:
```
$ mkinitcpio -P
```

### Booting the kernel
Edit `/boot/extlinux/extlinux.conf` so it properly load your newly installed kernel:

```
default l0
menu title U-Boot menu
prompt 1
timeout 10

label l0
        menu label Arch Linux PineNote
        linux /boot/Image
        initrd /boot/initramfs-linux.img
        fdt /boot/dtbs/rockchip/rk3566-pinenote-v1.2.dtb
        append root=/dev/mmcblk0p6 ignore_loglevel rw rootwait earlycon console=tty0 console=ttyS2,1500000n8 fw_devlink=off quiet loglevel=3 systemd.show_status=auto rd.udev.log_level=3 plymouth.ignore-serial-consoles vt.global_cursor_default=0

label l0r
        menu label Arch Linux PineNote (Fallback)
        linux /boot/Image
        initrd /boot/initramfs-linux-fallback.img
        fdt /boot/dtbs/rockchip/rk3566-pinenote-v1.2.dtb
        append root=/dev/mmcblk0p6 ignore_loglevel rw rootwait earlycon console=tty0 console=ttyS2,1500000n8 fw_devlink=off quiet loglevel=3 systemd.show_status=auto rd.udev.log_level=3 plymouth.ignore-serial-consoles vt.global_cursor_default=0
```
