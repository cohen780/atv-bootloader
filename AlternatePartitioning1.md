# AppleTV/Linux combo #

Make sure you first [backup](ATVBackup.md) if you intend on changing the original AppleTV internal PATA.

This take a little setup but might be worth it for some users. I do not recommend this procedure for first time installation of Linux. Installing Linux to a external USB disk is easer and does not require the AppleTV to be opened. Once you open it, Apple can and will void the warrantee.

With this procedure, we are going to put both the AppleTV OS and Linux on an internal ATA disk. To boot Linux, we use a USB pen drive with just atv-bootloader installed. When atv-bootloader is loaded, it will search for GRUBs menu.lst and find it on the Linux partition on the internal ATA drive and boot it. Pull the USB pen drive and reboot -- presto back to the AppleTV OS.

And just for kicks we are also going to move from the standard 40GB 4200 rpm drive to a 100GB 7200 rpm drive. This method requires a "Factory Restore" as the disk partition UUIDs will change and that confuses efi firmware. A "Factory Restore" is a recovery mode for the AppleTV where a disk image located on the recovery partition is reinstalled to OSBoot. **Any content on OSBoot and Media partition will be lost.** There might be other ways around this, feel free to investigate.

So, a nutshell, we are going to create GPT partitions on the new disk that match the original disk. Add two more two partitions for Linux usage. Copy the contents of the original recovery partition to the new disk. Then install Linux. Boot to AppleTV OS and do the "Factory Restore" then update to the newer version. Boot to Linux and do the post install fixes. Remember that these instructions are a guide not a script to cut and paste. They will need to be altered (drive identifiers, sector counts, etc) to suite your particular setup.

# Combo with on New Internal Disk" #

---

For reference
```
"sdb" is the original AppleTV in a external USB enclosure
"sdc" is the new 100GB in a external USB enclosure
```

Follow the LiveCD instructions but stop before the partition section.

Let's see how the original AppleTV drive is setup so we can copy the partitioning.
```
sudo parted -s /dev/sdb unit s print
```

You should see something similar but don't worry if the absolute sector numbers are slightly diffferent.
```
Model: IC25N040 ATCS04-0 (scsi)
Disk /dev/sdb: 78140160s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start     End        Size       File system  Name      Flags  
 1      40s       69671s     69632s     fat32        EFI       boot   
 2      69672s    888823s    819152s    hfs+         Recovery  atvrecv
 3      888824s   2732015s   1843192s   hfs+         OSBoot           
 4      2732016s  77878015s  75146000s  hfs+         Media            
```

Now exit parted and GPT format the new disk
```
# always zero the starting sectors of the disk first
sudo dd if=/dev/zero of=/dev/sdc bs=4096 count=1M

# create the GPT format
sudo parted -s /dev/sdc mklabel gpt
```

Use parted on the new disk to find the exact number of sectors.
```
sudo parted -s /dev/sdc unit s print
Model: ST910021  3MH0Y3XW (scsi)
Disk /dev/sdb: 195371568s
Sector size (logical/physical): 512B/512B
Partition Table: gpt
```

This 100GB disk has 195371568 sectors but we can't use them all as GPT has a second partition table at the end and we do not want to overwrite this secondary table. Subtract 97 from 195371568 gives 195371471 and that's our final end point. Create partitions on the new disk that match the AppleTV
```
# 1-EFI
sudo parted -s /dev/sdc mkpart primary fat32 40s 69671s
sudo parted -s /dev/sdc set 1 boot on

# 2-Recovery
sudo parted -s /dev/sdc mkpart primary HFS 69672s 888823s
sudo parted -s /dev/sdc set 2 atvrecv on

# 3-OSBoot
sudo parted -s /dev/sdc mkpart primary HFS 888824s 2732015s

# 4-Media
sudo parted -s /dev/sdc mkpart primary HFS 2732016s 77878015s
```

Add the Linux EXT3 and Swap.
```
# 5-EXT3
sudo parted -s /dev/sdc mkpart primary ext3 77878016s 193323471s

# 6-Swap
sudo parted -s /dev/sdc mkpart primary linux-swap 193323472s 195371471s
```

Format the partitions, note that we are using journaled hfsplus to match the original disk format. We let the Linux install deal with swap.
```
# sync the system partition tables
sudo partprobe /dev/sdc

# format the partitions
sudo mkfs.msdos -F 32 -n EFI /dev/sdc1
sudo mkfs.hfsplus -v Recovery /dev/sdc2
sudo mkfs.hfsplus -J -v OSBoot /dev/sdc3
sudo mkfs.hfsplus -J -v Media /dev/sdc4
sudo mkfs.ext3  -b 4096 -L Linux /dev/sdc5
```

Setup for the clone by making two mount points called "src" and "dst". The source partition gets mounted at "src", the destination gets mounted at "dst" and the contents get copied over. Only copy the recovery partition as OSBoot and Media will be re-created during the "Factory Restore" process.

One thing to note here about mounting journaled hfsplus file systems. Linux will not normally mount a journaled hfsplus for write access. This is because the kernel hfsplus module is not capable of replaying the journal if the disk was not unmounted cleanly. Bad things will happen when you write to an journaled file system that is "dirty". But the "fsck.hfsplus" from the hfs tools we built earler can replay the journal and fix any issues and then we can then we can safely force the mount read/write.
```
# make sure the original AppleTV has a clean file system
sudo fsck.hfsplus /dev/sdb2
sudo fsck.hfsplus /dev/sdb3
sudo fsck.hfsplus /dev/sdb4

# create the mount points
mkdir src dst

# clone Recovery
sudo mount /dev/sdb2 src
sudo mount -t hfsplus -o rw,force /dev/sdc2 dst
sudo cp -arp src/* dst/
sudo umount src dst

# done with the mount points so delete them
sudo rmdir src dst
```

Now follow one of the guides and install Linux to /dev/sdc5 with swap on /dev/sdc6. Remember de-select all  partitons other than sdc5 and sdc6, install grub and build and run "gptsync" after the install to fix up the MBR so efi firmware is happy.

Make a USB pen drive with just a recovery partition and the contents of recovery.tar.gz. atv-bootloader is just the contents of recovery.tar.gz (note, I am leaving the version number off so I do not have to fix bad links every time the version changes).

After installing Linux to the new disk, re-install the disk into the AppleTV and power up. It should boot to the recovery screen, select "Factory Reset" and wait until it is done. Now you can upgrade to the new version of the AppleTV OS if desired.

Boot to Linux with the USB pen inserted and do the post install fixes for Linux.

Now, you have both the AppleTV OS and Linux installed on the internal ATA disk.

Power up with USB pen removed -> AppleTV OS,

Power up with USB pen installed -> LInux.

You might have to force USB boot by holding the IR remote "menu" and "-" buttons down.