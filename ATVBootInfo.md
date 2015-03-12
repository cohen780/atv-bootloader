# AppleTV Boot Info #

All Intel Apple computers (including the AppleTV) use GPT as a partition format. The internal disk inside the AppleTV has four partitions. If you want to know more about GPT and GUIDs see http://en.wikipedia.org/wiki/GUID_Partition_Table. Also [the Apple Tech Note has good information](http://developer.apple.com/technotes/tn2006/tn2166.html).

The first partition is labeled "EFI" and is FAT32 but it has a special GUID. This partition is typically empty and only used for RAID drives on OS X desktop or server computers. The AppleTV disk does not really need this partition but I always include it on internal hard drives because you never know.

The second partition is labeled "Recovery". It's a HFS-plus format but it also has a special GUID. The AppleTV keeps a disk image (dmg) of the OSBoot files. It's called Recovery because the AppleTV firmware can restore the next partition (OSBoot) in case it detects problems. When you select "restore" from the configure menu, this is where the AppleTV pulls an OS image.

The third partition is labeled "OSBoot". It's also a HFS-plus format and has the normal GUID for HFS-plus. This holds the normally booted AppleTV OS (which is a derivative of OS X 10.4.x) and FrontRow.

The fourth partition is labeled "Media". It's also HFS-plus and it only holds media content. This partition can be reduced in size if you want to create more partitions for linux usage.

The AppleTV firmware boots by looking for a file called boot.efi on OSBoot. Boot.efi loads a darwin mach kernel (called mach\_kernel) and drivers (kext). If it can't find boot.efi, it then looks for boot.efi on Recovery. Another way is if the system is booted with the "Menu" and "-" keys on the Apple IR remote held down, it checks for a FAT32 or HFS partition with a Partition Type of {5265636F-7665-11AA-AA11-00306543ECAC}, first on a USB storage device, then on the internal HDD. If such a partition is found, it's mounted and a file called "boot.efi" is read from the root folder and executed.

We boot Linux by taking advantage of the recovery mode of the firmware. When the AppleTV OS boots, it resets an internal boot attempt count. If the boot attempt count goes to zero, the firmware will look over the USB bus for a Mass Storage Device with a Recovery partition and attempt boot. You can also force the USB probe by holding down the "menu" and "-" buttons on the IR remote at powerup.

So to get Linux to boot, we create a proper Recovery partition with boot.efi and a mach kernel that is actually a secondary Linux boot loader. The firmware loads boot.efi which loads our "mach kernel" and a dummy kext, at that point we have control and own the box.

Creating the proper disk format and partitions is critical for booting Linux on the AppleTV. The disk is GPT formatted and must have the proper GUID associated with specific partitions or the AppleTV will refuse to boot.
