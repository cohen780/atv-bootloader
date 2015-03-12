# How kexec works #

This is a guide for those wanting to better understand [kexec](http://www.ibm.com/developerworks/linux/library/l-kexec.html) and it's use in atv-bootloader. kexec has been used in Linux for several years. Everyone is familiar with "exec" and it's use to launch a Linux executable, well kexec is similar but is used to launch a Linux kernel. This allows a quick boot into a new kernel without having to go through PC BIOS. Another use is to launch a crash-dump kernel in case the running kernel panics (kdump). The Sony PS3 also uses this method to boot Linux, in this case a custom kernel/initramfs is located into firmware. This kernel then boot the "real" Linux system using kexec.

The previous two AppleTV bootloaders worked pretty well but had a single glaring omission. Nether could boot a Linux distro on a USB mass storage device (disk/flash/cdrom). The AppleTV EFI firmware has USB device support so it can load these bootloaders from a USB device, but USB device support was missing in the actual Linux bootloader. So while one could boot the actual bootloader on a USB disk/flash drive, because actual Linux bootloaders lacked USB device support, they could not see anything over the USB bus.

USB device could have been added to the previous bootloaders but that's a non-trivial task involving lot's of debugging. The easer path is to use the mach\_linux\_boot method to boot a stripped Linux kernel with a small filesystem containing kexec and use kexec to boot the "real" kernel". Since a Linux kernel already has USB device support, the only coding require was to find the "real" kernel and then kexec into it. That's the simplified version, it was actually more complex as the secondary design task was to alter mach\_linux\_boot to support initrd/iniramfs loading and remove EFI dependencies regarding the kernel. But that's another tech note for later.

It's pretty easy to use kexec once you understand what it needs as parameters and that kexec is a two step process.

  * The first step loads the new kernel, initrd/initramfs and kernel params into a section of RAM and informs the current kernel that a kexec jump is pending.

  * The second step actually tells the current kernel to reboot but instead of rebooting it will take the system down, then kexec to new kernel.

Let's take this in detail now. For this example, we will do a manual [KnoppMyth](http://mysettopbox.tv/) cdrom boot using a external USB cdrom. This assumes that you have created a USB flash drive with atv-bootloader installed and telnet enabled. It's much easer to do this using a telnet connection. Use the [Linux USB flash boot guide](LinuxUSBPenBoot.md) to create your USB boot disk and use it boot the AppleTV.

Mount the cdrom and dump "isolinux.cfg" file to see the kernel, initrd/initramfs and kernel params. The kernel and initrd are going to be loaded into RAM so the disk/cdrom where they are located will need to be mounted. Neither atv-bootloader nor kexec will automatically mount any disks.
```
mkdir cdrom
mount -o loop /dev/sr0 cdrom

cat cdrom/boot/isolinux.cfg
```

This give a long dump of "isolinux.cfg" but we are only interested in the default boot setting.
```
DEFAULT linux
APPEND ramdisk_size=100000 init=/etc/init lang=us vga=791 initrd=minirt.gz nomce
```

With isolinux, paths are relative to the location of "isolinux.cfg". The path relative to the "cdrom" mount point above for the kernel and initrd is
```
kernel -> cdrom/boot/linux
initrd -> cdrom/boot/minirt.gz
```

Now create a kexec load command. One thing to remember is that Linux on the AppleTV is sensitive to choice of console framebuffer. Pick the wrong one and you will not see any console output. With the above command-line params "vga=791" is bad so we replace it with "vga=normal". Since "splash" can also cause problems, if "splash" is present, do NOT include it. Sometimes you might need to include "video=vesafb". Here is the above "isolinux.cfg" default setting converted to kexec load command.
```
kexec --load cdrom/boot/linux \
--initrd=cdrom/boot/minirt.gz \
--command-line="ramdisk_size=100000 init=/etc/init lang=us vga=normal initrd=minirt.gz nomce"
```

Doing this command will result in kexec loading these items into RAM and setting kernel flags to inform the current kernel that a kexec jump is pending.

To actually do the jump to the new kernel
```
kexec -e
```

This will unmount any disks and do an immediate reboot into the new kernel. If you have made a "wise choice" about console framebuffer settings and your new kernel supports "vesafb", you should see the new kernel booting.

Now armed with this knowledge you can boot any Linux kernel even those that atv-bootloader cannot auto boot. For example, syslinux/isolinux support is currently broke and LILO support is missing (these will be fixed soon).

[KnoppMyth](http://mysettopbox.tv/) uses LILO so one can use the above guide to boot the KnoppMyth installer, install KnoppMyth to the internal PATA disk, then craft a "atv-boot=manual" invoked "boot\_linux.sh" that atv-bootloader uses to boot the installed KnoppMyth. Here is an example of such a "boot\_linux.sh" that can boot KnoppMyth KM5F27 installed to the internal PATA disk.
```
mkdir tmp
mount /dev/sda1 tmp

kexec --load tmp/boot/vmlinuz-2.6.18-chw-13 \
--initrd=tmp/boot/initrd.gz \
--command-line="root=/dev/hda1 initrd=initrd.gz vga=normal lang=us nomce"

umount tmp

kexec -e
```


Piece of cake.






