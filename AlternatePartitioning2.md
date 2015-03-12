# AppleTV/Linux combo #

Make sure you first [backup](ATVBackup.md) as you will be altering the original AppleTV internal PATA partition structure.

This take a little setup but might be worth it for some users. I do not recommend this procedure for first time installation of Linux. Installing Linux to a external USB disk is easer.

With this procedure, we are going to put both the AppleTV OS and Linux on the internal PATA disk. To boot Linux, we use a USB pen drive with just atv-bootloader installed. When atv-bootloader is loaded, it will search for GRUBs menu.lst and find it on the Linux partition on the internal ATA drive and boot it. Pull the USB pen drive and reboot -- presto back to the AppleTV OS.

This method requires a "Factory Restore" as the disk partition UUIDs will change and that confuses efi firmware. A "Factory Restore" is a recovery mode for the AppleTV where a disk image located on the recovery partition is reinstalled to OSBoot. **Any content on OSBoot and Media partition will be lost.** There might be other ways around this, feel free to investigate.

So, a nutshell, we are going to use the existing GPT partition and shrink the "Media" partition. In the recovered space, add two more two partitions for Linux usage. Then install Linux. Boot to AppleTV OS and do the "Factory Restore" then update to the newer version. Boot to Linux and do the post install fixes. Remember that these instructions are a guide not a script to cut and paste. They will need to be altered (drive identifiers, sector counts, etc) to suite your particular setup.

# Combo with "Media" Shrink" #

---

For reference
```
"sda" is the original AppleTV
```

Follow the LiveCD instructions but stop before the partition section.

Let's see how the original AppleTV drive is setup so we can copy the partitioning.
```
parted -s /dev/sda unit s print
```

You should see something similar but don't worry if the absolute sector numbers are slightly diffferent.
```
Model: IC25N040 ATCS04-0 (scsi)
Disk /dev/sda: 78140160s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start     End        Size       File system  Name      Flags  
 1      40s       69671s     69632s     fat32        EFI       boot   
 2      69672s    888823s    819152s    hfs+         Recovery  atvrecv
 3      888824s   2732015s   1843192s   hfs+         OSBoot           
 4      2732016s  77878015s  75146000s  hfs+         Media            
```

Shrink the "Media" partition by deleting it, then create a new partition that is reduced in size. "Media" is 75146000 `*` 512 = 38,474,752,000 bytes in size. Let's trim that down to 18,474,752,000 or 18,474,752,000 / 512 = 36083500 sectors. Round it down by one sector (36083499) so that the next sector aligns on an even sector count.
```
parted -s /dev/sda rm 4

parted -s /dev/sda mkpart primary HFS 2732016s 36083499s
```


Add the Linux EXT3 and Swap. Since we want swap to be 512MB, start by total sectors 78140160 - 34 = 78140126. 512MB = 536,870,912 bytes or 536,870,912 / 512 = 1048576 sectors. So 78140126 - 1048576 = 77091550. That the ending point for the ext3 partition. Round it down by one sector (77091549) so that swap aligns on an even sector count.
```
# 5-EXT3
parted -s /dev/sda mkpart primary ext3 36083500s 77091549s

# 6-Swap
parted -s /dev/sda mkpart primary linux-swap 77091550s 78140126s
```

Re-sync the system partition tables.
```
partprobe /dev/sda
```

Verify the disk partitioning.
```
Model: IC25N040 ATCS04-0 (scsi)
Disk /dev/sda: 78140160s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start     End        Size       File system  Name      Flags  
 1      40s       69671s     69632s     fat32        EFI       boot   
 2      69672s    888823s    819152s    hfs+         Recovery  atvrecv
 3      888824s   2732015s   1843192s   hfs+         OSBoot           
 4      2732016s  36083499s  33351483s  hfs+         Media            
 5      36083500s 77091549s  41008049s                    
 6      77091550s 78140126s  1048576s                   
```

Format the new partitions, note that we are using journaled hfsplus to match the original "Media" format. We let the Linux install deal with swap.
```
mkfs.hfsplus -J -v Media /dev/sda4
mkfs.ext3  -b 4096 -L Linux /dev/sda5
```

Follow one of the [install guides at the bottom of the previous page](InstallingLinux.md) and install Linux to /dev/sda5 with swap on /dev/sda6. Remember de-select all  partitons other than sda5 and sda6, install grub and build and run "gptsync" after the install to fix up the MBR so efi firmware is happy.

Make a USB pen drive with just a recovery partition and the contents of recovery.tar.gz. atv-bootloader is just the contents of recovery.tar.gz (note, I am leaving the version number off so I do not have to fix bad links every time the version changes).

After installing Linux, reboot. It should boot to the recovery screen, select "Factory Reset" and wait until it is done. Now you can upgrade to the new version of the AppleTV OS if desired.

Boot to Linux with the USB pen inserted and do the post install fixes for Linux.

Now, you have both the AppleTV OS and Linux installed on the internal ATA disk.

Power up with USB pen removed -> AppleTV OS,

Power up with USB pen installed -> LInux.

You might have to force USB boot by holding the IR remote "menu" and "-" buttons down.