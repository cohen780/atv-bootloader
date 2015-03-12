# Partition for Bootable USB Linux Install #

A bootable USB drive with partitions per-formatted for a Linux install. You can use this guide for a USB flash or hard disk.

First the organization, We want this to be self contained with atv-bootloader installed on the disk so the disk can boot on the AppleTV. Since this is a USB drive, AppleTV EFI will never check for "EFI or "OSBoot" so we don't need to include them. This means three partitions - our friend "Recovery" with atv-bootloader, "ext3" for the root file system and "swap".

Do this on a working Linux system or a LiveCD. We need our standard items for creating an AppleTV "Recovery" partition. These are boot.efi, the patched parted and hfs tools. See [extract boot.efi](BootEFIExtraction.md) for boot.efi, [install parted](InstallParted.md) for parted and [install hfs tools](InstallHFSTools.md) for hfs support. Test that you have to correct parted.
```
parted --version

parted (GNU parted) 1.8.8
Copyright (C) 2007 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by <http://parted.alioth.debian.org/cgi-bin/trac.cgi/browser/AUTHORS>.
```

Assuming you have these items extracted/built/installed, let's start. I'm using a USB flash drive. This USB flash is virgin out of the box. Check it for funny partitions and formating that might indicate the presence of those [[pain-in-the-rear Windows additions like U3](http://www.u3.com/smart/default.aspx). Insert the USB flash drive, my Linux mounted it at "/dev/sdb", "fdisk -l /dev/sdb" shows
```
Disk /dev/sdb: 8053 MB, 8053063680 bytes
183 heads, 32 sectors/track, 2685 cylinders
Units = cylinders of 5856 * 512 = 2998272 bytes
Disk identifier: 0xc3072e18

   Device Boot      Start         End      Blocks   Id  System
/dev/sdb1   *           1        2686     7864304    c  W95 FAT32 (LBA)
```

Single partition with a plain jane FAT32 format. Wipe the disk and proceed with creating our GPT partitions.
```
# umount the partition (Linux will have auto-mounted it)
sudo umount /dev/sdb1

# zero the initial sectors
sudo dd if=/dev/zero of=/dev/sdb bs=4096 count=1M

# sync the system disk partition tables
sudo partprobe /dev/sdb

# create the GPT format
sudo parted -s /dev/sdb mklabel gpt
```

Do a "sudo parted -s /dev/sdb unit s print" so we know the sector count
```
Model: Corsair Flash Voyager (scsi)
Disk /dev/sdb: 15728640s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start  End  Size  File system  Name  Flags
```

Figure out the partition sizes. We want a 512MB swap. The ext3 takes the rest. Sector count is 15728640, ending sector is 15728640s - 34s = 15728606s. That's 15,728,606 `*` 512 = 8,053,046,272 bytes. 8,053,046,272 - 536,870,912 (512MB swap) = 7,516,175,360 bytes or 14,680,030 sectors for ext3. Adjust the end of ext3 down by 1 so swap starts on an even boundary.
```
# create just a recovery partition
sudo parted -s /dev/sdb mkpart primary HFS 40s 69671s
sudo parted -s /dev/sdb set 1 atvrecv on
sudo parted -s /dev/sdb mkpart primary ext3 69672s 14680029s
sudo parted -s /dev/sdb mkpart primary linux-swap 14680030s 15728606s
```

Sync the partition tables and format the partitions
```
# sync the system disk partition tables
sudo partprobe /dev/sdb

# format the partitions
sudo mkfs.hfsplus -v Recovery /dev/sdb1
sudo mkfs.ext3  -b 4096 -L Linux /dev/sdb2

# let the installer handle swap.
```

Verify your partitioning, "atvrecv" must be present.
```
# parted -s /dev/sdb unit s print
Model: Corsair Flash Voyager (scsi)
Disk /dev/sdb: 15728640s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start      End        Size       File system  Name     Flags  
 1      40s        69671s     69632s     hfs+         primary  atvrecv
 2      69672s     14680029s  14610358s  ext3         primary         
 3      14680030s  15728606s  1048577s                primary         
```

Install atv-bootloader
```
# download recovery files
wget http://atv-bootloader.googlecode.com/files/recovery-0.6.tar.gz
tar -xzf recovery-0.6.tar.gz

# make some mount points
mkdir /mnt/recovery

# mount the USB flash drive
mount /dev/sdb1 /mnt/recovery

sudo cp -arp recovery/* /mnt/recovery/

# remember to copy boot.efi 
sudo cp -ap boot.efi /mnt/recovery
```

Done with the prep-work, pick the your Linux CD Installer and proceed to boot and installing from CD.







