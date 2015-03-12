# Booting a LiveCD #

This guide describes how to boot a Linux LiveCD directly from the AppleTV. It requires;
  1. AppleTV (of course).
  1. An existing USB flash drive with atv-bootloader installed.
  1. An existing LiveCD.
  1. An working Linux computer capable of telnet
  1. A network with a DHCP server (router, etc)
  1. A wired network connection to the AppleTV.
  1. A USB Hub (a powered USB hub is preferable but not required).
  1. A USB mouse.
  1. A USB keyboard.

Optional
  1. A USB cdrom drive (you can install from the contents of an iso)


---

# Ubuntu 7.10 Gutsy #
Let's get started. For this example, we are going to manually boot Ubuntu 7.10 Gutsy LiveCD using atv-bootloader. This guide uses a USB CD/DVD drive but you can boot using the extracted contents of a LiveCD onto a USB flash drive.

On a working Linux system, create a USB flash drive with atv-bootloader installed. You will need to first extract "boot.efi" and build and install parted and hfstools. See [extract boot.efi](BootEFIExtraction.md) for boot.efi, [install parted](InstallParted.md) for parted and [install hfs tools](InstallHFSTools.md) for hfs support].

Remember to **[backup the AppleTV](ATVBackup.md)** if you are going to over-write the internal PATA disk. You can use the same USB flash drive for this guide and skip the next section.

Once these tools are installed then create just the recovery partition on a USB pen drive, this does not require very much space so 64MB or greater pen drive is fine. I recommend a USB flash disk of 512MB or greater, that way you can also use the same disk to backup the AppleTV. This guide will assume the device at "/dev/sdb" is the pen drive so remember to adjust this to match your setup. If you have problems partitioning, you might have a [USB flash drive that needs fixing](FixingUSBFlashDrives.md).
```
# zero the initial sectors
sudo dd if=/dev/zero of=/dev/sdb bs=4096 count=1M

# create the GPT format
sudo parted -s /dev/sdb mklabel gpt

# create just a recovery partition
sudo parted -s /dev/sdb mkpart primary HFS 40s 69671s
sudo parted -s /dev/sdb set 1 atvrecv on

# update the system partition tables
sudo partprobe /dev/sdb

# format it
sudo mkfs.hfsplus -v Recovery /dev/sdb1

# mount it
mkdir penboot
sudo mount /dev/sdb1 penboot

# download atv-bootloader (recovery.tar.gz) and install it
wget http://atv-bootloader.googlecode.com/files/recovery-0.6.tar.gz
sudo tar -xzf recovery-0.6.tar.gz
sudo cp -arp recovery/* penboot/

# remember to copy boot.efi to penboot/
sudo cp -ap boot.efi penboot/

# unmount and cleanup
umount penboot
rmdir penboot
```

The current version of atv-bootloader will enable telnetd and drop to a login prompt if it does not find any valid boot configs. Power-on the AppleTV with the USB flash drive inserted, you might have to hold "menu" and '-' buttons down on the Apple IR remote to force USB probe and boot.

You should see the atv-bootloader logo (tux on the AppleTV), then kernel boot console messages (tux in the upper left corner), then a "penbuntu" login prompt. It's easer to do everything from a telnet session so telnet in using the IP address that is reported by atv-bootloader. Username is "root", password is "root".

Power-up the USB cdrom and insert your LiveCD. Wait until the cdrom spins up, then via a telnet session;
```
mkdir /cdrom
mount -o loop /dev/sr0 /cdrom
```

Find "isolinux.cfg" boot.conf. This is either at the root level of the cdrom or in a "boot" or "isolinux" directory. For Ubuntu 7.10 Gutsy the path is "/cdrom/isolinux/isolinux.cfg". Dump this file and locate the default boot setting", for Gutsy it looks like this;
```
DEFAULT /casper/vmlinuz
GFXBOOT bootlogo
GFXBOOT-BACKGROUND 0xB6875A
APPEND  file=/cdrom/preseed/ubuntu.seed boot=casper initrd=/casper/initrd.gz quiet splash --
```

Now we manually convert isolinux params into kexec params. See this link for more information about [understanding kexec params](Understandingkexec.md).  An important note, the AppleTV does not have a PC BIOS so video console mode changes will not work. With atv-bootlaoder, a "vesafb" will work as a console framebuffer and you should see the kernel boot messages. However, Ubuntu does not include "vesafb" in their initramfs so we will not see kernel boot messages. In addition we need to add "vesa" and change "splash" to "nosplash" to get X11 to come up, here are the resulting kexec params.
```
kexec --load /cdrom/casper/vmlinuz \
--initrd=/cdrom/casper/initrd.gz \
--command-line="file=/cdrom/preseed/ubuntu.seed boot=casper initrd=/casper/initrd.gz nosplash vesa" 
```

Doing the above, on the command-line results in the kexec load printing the following information. This means, kexec has loaded the kernel/initrd and params into RAM and has set flags that a kexec jump is pending. "1280x720x32" is the video resolution as passed by the AppleTV EFI firmware, 10028000 is the video base address in hexadecimal.
```
setup_linux_vesafb: 1280x720x32 @ 10028000 +708000
```

Now do the kexec jump
```
kexec -e
```

You should hear the cdrom spin up and start making disk access noises. With Gutsy, since the console framebuffer does not work, the display will look frozen. Wait until X11 starts. Then the display will change and if you are lucky, it will be perfect. Gusty is not so lucky as there seems to be some bug in X11 does not setup the initial video properly. What I see is a horizontal repeating desktop (four repeats). All is not lost, if you move the mouse around, you will notice that the mouse/cursor coordinates are correct. So move the cursor to the upper left, mouse down and move right slowly. The menu in the menubar will drop down. Go to System -> Administration -> Screens and Graphics. Select a resolution supported by your display. I picked 1280 x 720 to match the AppleTV EFI firmware setting. Click "OK" not "Test" and the display should change resolution and life is now good.

It takes a little practice to get the cursor in the correct place, think a little horizontal offset and watch the control/button highlighting. The control/button will highlight when the cursor is over top.

The other way is to open a terminal "Applications -> Accessories -> Terminal". The type "sudo displayconfig-gtk" and this will open the GUI display settings window.

If you have pre-formatted the destination disk, proceed to installing your distro. If not, then you can use the LiveCD to build and install parted and hfstools. See [install parted](InstallParted.md) for parted and [install hfs tools](InstallHFSTools.md) for hfs support].

Other LiveCD distributions will be similar steps. Extract the default ioslinux settings. Translate to kexec params. Turn off "splash", set "vga=normal" and add "video=vesafb". Then see what happens.

# Ubuntu 9.10 or greater #
Same as above but edit the file text.cfg in the syslinux folder on the root of your usb drive. Just change any reference to initrd.gz to initrd.ls. Thanks to Brian Gregson for this find.



---

# Do I really need a USB CD/DVD drive #
No, you can extract the contents of the LiveCD iso to a USB flash drive and install from that. Here's how using Ubuntu. On an existing Linux system
```
# Download the iso.
http://mirrors.us.kernel.org/ubuntu-releases/hardy/ubuntu-8.04-desktop-i386.iso

# mount the iso
mkdir iso
mount -o loop ubuntu-8.04-desktop-i386.iso iso

# mount your USB flash drive with
#   atv-bootloader on 1st partition, 
#   empty ext3 on 2nd partition.
mkdir usb
mount /dev/sdb2 usb

# copy selected contents of the iso into the flash
# ".disk" is important as this is what "casper" the 
# Ubuntu initrd used to find valid iso contents
#
cd iso
cp -rf casper dists install pics pool preseed .disk ../usb
cp isolinux/isolinux.cfg md5sum.txt README.diskdefines ../usb
cp casper/vmlinuz casper/initrd.gz install/mt86plus ../usb

# cd into the usb and make some changes
cd ../usb
mv isolinux.cfg syslinux.cfg
sed -i -e 's:/casper/::g' -e 's:/cdrom/::g' -e 's:/install/::g' syslinux.cfg
```

Now after atv-bootloader boots, telnet in and
```
mkdir tmp

mount /dev/sdb2 tmp

kexec --load tmp/casper/vmlinuz --initrd=tmp/casper/initrd.gz --command-line="preseed/ubuntu.seed boot=casper initrd=/casper/initrd.gz nosplash vesa video=vesafb"

kexec -e
```

And bingo into the Ubuntu LiveCD.


---

# Mythbuntu (Gutsy or Hardy) #
Here's an example for Mythbuntu, the default isolinux setting is
```
DEFAULT /casper/vmlinuz
GFXBOOT bootlogo
APPEND  boot=casper initrd=/casper/initrd.gz quiet splash --
```

The translated kexec params for cdrom boot are
```
kexec --load /cdrom/casper/vmlinuz \
--initrd=/cdrom/casper/initrd.gz \
--command-line="boot=casper initrd=/casper/initrd.gz nosplash vesa" 
```

Mythbuntu uses GRUB so atv-bootloader will be able to automatically find and translate GRUB's menu.lst boot settings.


---

# KnoppMyth #

Here's an example for KnoppMyth, the default isolinux setting is
```
DEFAULT linux
APPEND ramdisk_size=100000 init=/etc/init lang=us vga=791 initrd=minirt.gz nomce
```

The translated kexec params for cdrom boot are
```
kexec --load /cdrom/boot/isolinux/linux \
--initrd=/cdrom/boot/isolinux/minirt.gz \
--command-line="ramdisk_size=100000 init=/etc/init lang=us vga=normal initrd=minirt.gz nomce"

kexec -e
```

And the LILO translated kexec params for a manual "boot\_linux.sh" using "atv-boot=manual" are
```
mkdir /root/tmp
mount /dev/sda1 /root/tmp

kexec --load /root/tmp/boot/vmlinuz-2.6.18-chw-13 \
--initrd=/root/tmp/boot/initrd.gz \
--command-line="root=/dev/hda1 lang=us nomce vga=normal initrd=initrd.gz"

kexec -e
```


---

# MythDora 5 #

Here's an example for MythDora 5, the default isolinux setting is
```
label linux
  menu label ^Install or upgrade an existing system
  menu default
  kernel vmlinuz            
  append initrd=initrd.img noselinux selinux=off
```

The translated kexec params for cdrom boot are
```
kexec --load /cdrom/isolinux/vmlinuz \
--initrd=/cdrom/isolinux/initrd.img \
--command-line="initrd=initrd.img noselinux selinux=off video=vesafb xdriver=vesa"

kexec -e
```

And the GRUB translated kexec params for a manual "boot\_linux.sh" using "atv-boot=manual" are
```
mkdir /root/tmp
mount /dev/sda1 /root/tmp

kexec --load /root/tmp/vmlinuz-2.6.24.4-64.fc8 \
--initrd=/root/tmp/initrd-2.6.24.4-64.fc8.img \
--command-line="ro root=/dev/VolGroup00/LogVol00 video=vesafb rhgb quiet"

kexec -e
```


---

# Netboot Install #
Here's an example of installing using Ubuntu Hardy Netboot. This requires a network connection as Hardy will be installed from network sources. The NetBoot installer allows the installation of all the Ubuntu derivatives (Hardy desktop, Mythbuntu Frontend, etc) First download and extract the Hardy
Netboot files. This also works with Gutsy, just change the name below.
```
http://us.archive.ubuntu.com/ubuntu/dists/hardy/main/installer-i386/current/images/netboot/netboot.tar.gz

tar -xzf newboot.tar.gz
```

Now copy just "linux" and "initrd.gz" to the atv-bootloader USB flash drive
```
mkdir penboot

mount /dev/sdb1 penboot

cp ubuntu-installer/i386/linux penboot/
cp ubuntu-installer/i386/initrd.gz penboot/

umount penboot
```

Boot the AppleTV using atv-bootloader and connect using telnet. Then
```
mkdir tmp

mount /dev/sdb1 tmp

kexec --load tmp/linux --initrd=tmp/initrd.gz --command-line="vga=normal vesa video=vesafb"

kexec -e
```

The netboot installer will startup, follow the directions. Remember to pre-partition in atv-bootloader if you want to use GPT partitions with atv-bootloader installed on one of the partitions for selfboot.