# ATV-Bootloader on USB flash disk #

In this guide, we will build an atv-bootloader based USB flash disk that can be used for standalone boot. We will also enable telnet so we don't need a USB keyboard attached and can do everything remotely using a telnet session using a wired network connection. Unfortunately, wireless is not supported at this time.

Since atv-bootloader contains all the disk tools required for creating GPT formatted partitions, one can use it to boot, partition and install Linux with a USB cdrom using only the AppleTV. Of course you do need a working Linux system to first create this USB flash disk.

We need our standard items for creating an AppleTV "Recovery" partition. These are boot.efi, the patched parted and hfs tools. See [extract boot.efi](BootEFIExtraction.md) for boot.efi, [install parted](InstallParted.md) for parted and [install hfs tools](InstallHFSTools.md) for hfs support. Test that you have to correct parted.
```
parted --version

parted (GNU parted) 1.8.8
Copyright (C) 2007 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by <http://parted.alioth.debian.org/cgi-bin/trac.cgi/browser/AUTHORS>.
```

Assuming you have these items extracted/built/installed, let's start.

Create the recovery partition on a USB pen disk, this does not require very much space so 64MB or greater pen drive is fine. I recommend a USB flash disk of 512MB or greater, that way you can also use the same disk to backup the AppleTV. Make sure the USB flash drive is unmounted, most Linux distros will auto-mount USB flash drives. If you have problems partitioning, you might have a [USB flash drive that needs fixing](FixingUSBFlashDrives.md).

**This guide assumes the device at "/dev/sdb" is the pen drive so remember to adjust this to match your device setup or very bad things will happen**

```
# zero the initial sectors
sudo dd if=/dev/zero of=/dev/sdb bs=4096 count=1M

# sync the system disk partition tables
sudo partprobe /dev/sdb

# create the GPT format
sudo parted -s /dev/sdb mklabel gpt

# create just a recovery partition
sudo parted -s /dev/sdb mkpart primary HFS 40s 69671s
sudo parted -s /dev/sdb set 1 atvrecv on

# sync the system disk partition tables
sudo partprobe /dev/sdb
```

Verify that it looks fine and the atvrecv flag is set
```
sudo parted -s /dev/sdb unit s print

Model: SanDisk Cruzer Micro (scsi)
Disk /dev/sdb: 501759s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End      Size     File system  Name     Flags  
 1      40s     69671s   69632s                primary  atvrecv
```

Format this partition hfsplus
```
# format it
sudo mkfs.hfsplus -v Recovery /dev/sdb1

# mount it
mkdir penboot
sudo mount /dev/sdb1 penboot

# download atv-bootloader (recovery.tar.gz) and install it
wget http://atv-bootloader.googlecode.com/files/recovery-0.6.tar.gz
sudo tar -xzf recovery-0.6.tar.gz
sudo cp -arp recovery/* penboot/
```

Remember to copy boot.efi to penboot/
```
sudo cp -ap boot.efi penboot/
```

Override the normal auto boot sequence as we need to get into a command-line enviroment and get telnetd started. Edit "penboot/com.apple.Boot.plist" on the recovery partition and change the atv-boot parameter from "auto"
```
<string>atv-boot=auto video=vesafb</string>
```

to "none"
```
<string>atv-boot=none video=vesafb</string>
```

When "none" is selected, atv-bootloader now drops to a command-line login, enables wired networking using DHCP, prints the network config so you know the IP address and starts the telnet demon.

Unmount the disk and we are done building atv-bootloader on a USB pen disk.
```
sudo umount penboot
```

# Usage #

Boot the AppleTV using the USB pen disk. Remember you have to force a "Recovery Boot" by holding "menu" and "-" buttons down on the Apple IR remote either during power-up or when the AppleTV OS is running.

You should see the kernel boot messages, the ifconfig dump and then a login prompt. Either login (user=root, password=root) using a USB keyboard or telnet in using the listed IP addresses. vi and nano are present as well as the parted and hfs tools for creating AppleTV GPT format partitions. You can also use this disk to [backup the AppleTV](ATVBackup.md) and [manually boot a LiveCD on a USB cdrom](BootingLiveCD.md).

Enjoy.
