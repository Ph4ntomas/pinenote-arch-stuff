# pinenote-arch-stuff
Tools, PKGBUILD and others miscellanous stuff to make arch works on PineNote

## Initial Setup
Initial debian -> arch conversion can be made using artoo's method here:
https://github.com/hrdl-github/pinenote-arch/tree/main

## Kernel Installation



### Linux Pinenote 6.12.11
[linux-pinenote](./linux-pinenote) contains a PKGBUILD to build the kernel, as well as other necessary file
to automate initramfs generation.

This kernel is built from the same sources the debian image built, and should behave the same.

The packages it generate install the kernel in the following way:
```
/boot/Image{,gz} # vmlinux image
/boot/dtbs # Device tree blobs
/etc/mkinitcpio.d/linux-pinenote.preset # mkinitcpio preset, for initramfs generation
/usr/lib/initcpio/6.12.11-1-pinenote # used to trigger mkinitcpio in pacman hooks
/usr/lib/modules/6.12.11-1-pinenote/ # kernel modules
```

#### rockchip_ebc configuration
Here is the default configuration from the debian image:

```
direct_mode=0 
auto_refresh=1 
refresh_threshold=60 
split_area_limit=0 
panel_reflection=1 
prepare_prev_before_a2=0 
dclk_select=0
```

On debian, there are set via modprobe, by writing the following file:
/etc/modprobe.d/rockchip_ebc.conf
```
options rockchip_ebc direct_mode=0 auto_refresh=1 refresh_threshold=60 split_area_limit=0 panel_reflection=1 prepare_prev_before_a2=0 dclk_select=0
```

### Linux Pinenote - Hrdl's Pixel Scheduling 
[linux-pinenote-pixelsched](./linux-pinenote-pixelsched) contains a newer revision of the graphic driver, written by hrdl.

This package is build from their latest commit here:
https://git.sr.ht/~hrdl/linux/tree/c88c6c96ed6dfaaa919a31c93f3d9d7e86a047a7

Here is the package structure:
```
/boot/Image{,gz} # vmlinux image
/boot/dtbs # Device tree blobs
/etc/mkinitcpio.d/linux-pinenote-pixelsched.preset # mkinitcpio preset, for initramfs generation
/usr/lib/initcpio/6.12.0-1-pinenote-pixelsched # used to trigger mkinitcpio in pacman hooks
/usr/lib/modules/6.12.0-1-pinenote-pixelsched/ # kernel modules
```

#### rockchip_ebc configuration
hrdl suggest the following parameter when using his implemetation:
```
use_neon=15 # use neon-based functions for blitting. 15 -> All neon backed features are enabled
direct_mode=1 # compute waveforms in software (software LUT)
early_cancellation=1 # allow cancelling ongoing A2 updates
early_cancellation_addition=2 # number of additional frames to drive a pixel when cancelling it
```

The easiest way to set the configuration is to create or edit `/etc/modprobe.d/rockchip_ebc.conf` so that it contains the following:
```
options rockchip_ebc use_neon=15 direct_mode=1 early_cancellation=1 early_cancellation_addition=2
```

### Linux Pinenote 6.15-rc3 - hrdl
[linux-pinenote-hrdl](./linux-pinenote-hrdl) contains a newer revision of the graphic driver, again by hrdl, which should allow operating at 60 to 85 Hz
This version was rebased on the latest release candidate from the upstream kernel kernel (6.15-rc3), available here:
https://git.sr.ht/~hrdl/linux/commit/bddeeb47679c8a871c95c557664b890c17a4679f

Here is the package structure:
```
/boot/Image{,gz} # vmlinux image
/boot/dtbs # Device tree blobs
/etc/mkinitcpio.d/linux-pinenote-hrdl.preset # mkinitcpio preset, for initramfs generation
/usr/lib/initcpio/6.15.0-rc3-1-pinenote-hrdl # used to trigger mkinitcpio in pacman hooks
/usr/lib/modules/6.15.0-rc3-1-pinenote-hrdl/ # kernel modules
```

#### rockchip_ebc configuration
hrdl suggest the following parameter when using this version:
```
dithering_method=2 # Dithering method, 0-2. 2 uses a 32x32 blue-noise texture
default_hint=0x80 # Hint to use for pixels not covered otherwise
early_cancellation_addition=2 # Number of additional frames to drive a pixel when cancelling it
redraw_delay=200 # number of hardware frames to delay redraws
```

The easiest way to set the configuration is to create or edit `/etc/modprobe.d/rockchip_ebc.conf` so that it contains the following:
```
options rockchip_ebc dithering_method=2 default_hint=0x80 early_cancellation_addition=2 redraw_delay=200
```

> [!IMPORTANT]
> This revision of the graphic driver uses a custom waveform format, so you'll have to convert your waveform file.

The conversion script can be found here:
https://git.sr.ht/~hrdl/pinenote-shared/tree/f0e5c756ad4fcceb98b67a80d1d7d05ab7fe1c75/item/helpers/ebc_waveform_tools

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

## D-Bus Service

A D-Bus Daemon is under development over at https://github.com/PNDeb/pinenote_dbus_service.
It's goal is to manage some control & settings 

The packages is split into `pinenote-dbus-service`, for the service itself, and `pinenote-dbus-service-utils` for extra binaries.

The main packages install the dbus daemon in the following way:
```
usr/bin/pinenote_dbus_service # main binary
usr/lib/systemd/system/pinenote_dbus.service # systemd style service
usr/share/dbus-1/system-services/org.pinenote.ebc.service # D-Bus activatable service
usr/share/dbus-1/system.d/pinenote.conf # D-Bus security config
usr/share/licenses/pinenote-dbus-service/COPYING
usr/share/licenses/pinenote-dbus-service/LICENSE-APACHE
usr/share/licenses/pinenote-dbus-service/LICENSE-MIT
```

The sub-package install everything else under `/usr/bin` (and add the licence in their own subdirectory)

### Starting the D-Bus Service

The Service can be started the systemd way:
```bash
$ sudo systemctl start pinenote_dbus.service
```

To make it start on every boot, run the following instead:
```bash
$ sudo systemctl enable --now pinenote_dbus.service
```

However, since it's installed as a Bus Activatable service, it can also be started by sending a 
message via dbus-send :

```bash
# explicitely send a message to D-Bus
$ dbus-send --system --print-reply --dest=org.freedesktop.DBus / org.freedesktop.DBus.StartServiceByName string:org.pinenote.ebc uint32:0
# or interact with the org.pinenote.ebs destination
$ dbus-send --system --print-reply --dest=org.pinenote.ebc /ebc org.pinenote.ebc.<MESSAGE>
```

I would still suggest to uses the systemd method, as I haven't really tinker much with the bus activation method.
