# Backup and Restore #

Note: This requires "recovery-0.6.tar.gz" or greater. The previous versions do not contain all the required command-line tools.

You have bought an AppleTV and want to alter the original internal PATA disk. To be able to restore the original contents, you need to make a backup. This is highly recommended and does not take much storage space. The AppleTV comes into two flavors, a 40GB version and a 160GB version. This method works with either and takes the same amount of storage space and time to perform.

What we do is take advantage of the self-restoring capability that is already built into the AppleTV EFI fireware. If we properly create the first three partitions (EFI, Recovery and OSBoot) and populate "Recovery", the AppleTV EFI firmware will go into a recovery boot. Then select "Factory Restore" and it will rebuild/populate the "OSBoot" and "Media" partitions.

**Warning** You might use "dd" to copy the entire disk but that's a waste of time and storage space and "dd" is not the correct tool to copy GPT format disks. Usage of dd to copy the entire disk is fortuitous, it might work, but CAN fail under certain conditions and is therefore unreliable. Proper GPT format includes a secondary partition map in the ending sectors on the disk. If these sectors do not exist because your destination disk is smaller than your source disk, it will not be present after using "dd". Failure to include the secondary partition map or failure to place it in the proper location will lead to unreliable results. Using "dd" for USB pen drives further flawed by inconstancies in flash based disk geometry. Even for non-GPT format, the use of "dd" can lead to unreliable results when used to image a USB flash based pen drive.

# Backup #
Let's get started, first build [atv-bootloader on a USB pen disk](LinuxUSBPenBoot.md) with telnet support. You will need a USB flash drive of 512MB or greater. Flash drives of this size are dirt cheap and I recommend buying one specific for this purpose. In addition to the "Recovery" partition that atv-bootloader uses, make a second partition called "backup" that is equal to or greater than 256MB in size. Make this partition "ext3" so the AppleTV EFI firmware ignores it.

This is the USB flash drive created in [atv-bootloader on a USB pen disk](LinuxUSBPenBoot.md)
```
parted -s /dev/sdb unit s print

Model: SanDisk Cruzer Micro (scsi)
Disk /dev/sdb: 501759s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End      Size     File system  Name     Flags  
 1      40s     69671s   69632s   hfs+         primary  atvrecv
```

Add the ext3 partition The ending sector is the max sectors - 34 sectors (501759s - 34s = 501725s)
```
sudo parted -s /dev/sdb mkpart primary ext3 69672s 501725s

# check it
parted -s /dev/sdb unit s print
Model: SanDisk Cruzer Micro (scsi)
Disk /dev/sdb: 501759s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End      Size     File system  Name     Flags  
 1      40s     69671s   69632s   hfs+         primary  atvrecv
 2      69672s  501725s  432053s               primary         

# sync the system disk partition tables
sudo partprobe /dev/sdb

# format this partition ext3
sudo mkfs.ext3 /dev/sdb2
```

Boot the AppleTV using this USB flash drive and telnet in. Username is "root", password is "root". Nothing is mounted so make two mount points and mount the second partition of the USB flash drive on one and the "Recovery" partition on the internal PATA disk on the other. Clone the contents of "Recovery" to our backup partition. You don't have to copy the contents of "EFI" as there is nothing to copy. That correct, "EFI" is empty. Note that we fsck.hfsplus the "Recovery" partition even though it will be mounted read only.
```
mkdir src dst

# mount the destination, we don't need to fsck as this is an ext3 partition
mount /dev/sdb2 dst

# mount the source
fsck.hfsplus /dev/sda2
mount -t hfsplus  /dev/sda2 src

cp -arp src/* dst/
# force a disk buffer flush
sync

umount src dst
# clean up by removing the mount points
rmdir src dst
```

Run "parted" and make a copy of the original partitioning for the internal PATA disk. If we alter the original partitioning, the internal PATA disk can be re-partitioned back to the original later using this copy as a guide. Copy this to the first partition of the USB flash drive for safe keeping.
```
mkdir tmp

# mount 
fsck.hfsplus /dev/sdb1
mount /dev/sdb1 tmp

parted -s /dev/sda unit s print > tmp/org_gtp.txt
```

Check the file using "cat tmp/org\_gtp.txt", for a 40GB disk it should look something like this.
```
Model: FUJITSU  K00FT7125M1W (scsi)
Disk /dev/sda: 78140160s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start     End        Size       File system  Name      Flags  
 1      40s       69671s     69632s     fat32        EFI       boot   
 2      69672s    888823s    819152s    hfs+         Recovery  atvrecv
 3      888824s   2732015s   1843192s   hfs+         OSBoot           
 4      2732016s  77878015s  75146000s  hfs+         Media            
```

```
umount tmp
# clean up by removing the mount points
rmdir tmp
```

Remove the USB flash drive and store it in a safe place. If you want, you can also tar and gzip the contents of the "backup" partition on the USB flash drive to another location. That was quick and painless.

For a 160GB disk, the end result will look something like this
```
# parted -s /dev/sda unit s print
Model: ATA SAMSUNG HM160JC (scsi)
Disk /dev/sda: 312581808s
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Number  Start     End         Size        File system  Name Flags
 1      40s       69671s      69632s      fat32        primary boot
 2      69672s    888823s     819152s     hfs+         primary atvrecv
 3      888824s   2732015s    1843192s    hfs+ primary
 4      2732016s  312319663s  309587648s  hfs+ primary
```

# Restore #

Boot the AppleTV using the same USB flash drive and telnet in. Again nothing is mounted. Mount the first partition of the USB flash drive and copy over the dump of the original partitioning.
```
mkdir tmp

# mount
fsck.hfsplus /dev/sdb1
mount /dev/sdb1 tmp

cp tmp/org_gtp.txt ./

umount tmp
rmdir tmp
```

Wipe the partition structure of the internal PATA disk. **Make sure you have the correct disk/partition or bad things will happen**
```
dd if=/dev/zero of=/dev/sda bs=4096 count=1M
```

Using the contents of "org\_gtp.txt" and "parted", re-create the original partitioning. I create and format all four partitions. Remember "OSBoot" and "Media" are formatted hfsplus journaled but "Recovery" is not.
```
# create the GPT format
parted -s /dev/sda mklabel gpt

# 1-EFI
parted -s /dev/sda mkpart primary fat32 40s 69671s
parted -s /dev/sda set 1 boot on

# 2-Recovery
parted -s /dev/sda mkpart primary HFS 69672s 888823s
parted -s /dev/sda set 2 atvrecv on

# 3-OSBoot
parted -s /dev/sda mkpart primary HFS 888824s 2732015s

# 4-Media
parted -s /dev/sda mkpart primary HFS 2732016s 77878015s

# make file systems for the four partitions
mkfs.msdos -F 32 -n EFI /dev/sda1
mkfs.hfsplus -v Recovery /dev/sda2
mkfs.hfsplus -J -v OSBoot /dev/sda3
mkfs.hfsplus -J -v Media /dev/sda4


# use partprobe to re-sync the new partitions to the system partition tables
partprobe /dev/sda
```

Mount and copy the original contents of "Recovery" from your "backup" partition into the newly created "Recovery" partition.
```
mkdir src dst

# mount the backup
mount /dev/sdb2 src

fsck.hfsplus -f /dev/sda2
mount -t hfsplus -o rw,force /dev/sda2 dst

cp -arp src/* dst/
# force a disk buffer flush
sync

umount src dst
# clean up by removing the mount points
rmdir src dst
```


Remove the USB flash drive and power cycle the AppleTV. It will see that contents of "OSBoot" is missing and go into a recovery boot. Select "Factory Restore" and the AppleTV EFI firmware will do the rest. It might do several reboots as it reconfigures itself.

When it finally comes up, presto, a factory fresh AppleTV.


# Opps I did not make a backup #

**Update - April 29, 2008 - Apple seems to have purged their previous updates. The following are now missing from mesu.apple.com. This means that one will no longer be able to recover back to the 1.x series. Bummer.** I've not tested this procedure with the 2.0.2 dmg update but it seems like it should work. I'll test it when I get a chance. You might run into issues if your AppleTV was at the original EFI firmware as the 2.0.x series will expect the updated EFI firmware that occurred during the 1.0.x to 2.0.x updates.
```
1.0.1 - at http://mesu.apple.com/data/OS/061-2988.20070620.bHy75/2Z694-5248-45.dmg
2.0.0 - at http://mesu.apple.com/data/OS/061-3561.20080212.ScoH6/2Z694-5274-109.dmg
2.0.1 - at http://mesu.apple.com/data/OS/061-4375.20080328.gt5er/2Z694-5387-25.dmg
```

Still present is the current 2.0.2 update.
```
2.0.2 - at http://mesu.apple.com/data/OS/061-4632.2080414.gt5rW/2Z694-5428-3.dmg
```

So you messed up and did bad things to the internal PATA disk without making a backup. All is not lost. You can save the day by using the AppleTV 2.0.2 update to re-create the AppleTV disk. Do this step on a working Linux box, the proceed to the "Restore" section above.

Start by downloading the update, converting it and mouting. Do this on a working Linux system or LiveCD. We going to use same methods as in [extract "boot.efi"](BootEFIExtraction.md) so see that section if you need to build dmg2img.

```
# download the AppleTV 2.0.2 update
wget http://mesu.apple.com/data/OS/061-4632.2080414.gt5rW/2Z694-5428-3.dmg
#
# convert it to an img format
dmg2img 2Z694-5428-3.dmg atv.img

# create a mount point
mkdir atv-update

# mount the converted dmg disk image
sudo mount -o loop -t hfsplus atv.img atv-update
```

The AppleTV "Recovery" partition has the following files. The byte count of your "mach\_kernel.prelink" and "OS.dmg" might be different, this is from an AppleTV rev 1.0. Note that "Desktop DB" and "Desktop DF" will be created by the AppleTV during recovery mode and do not need to be manually created.
```
-rw-r--r-- 1 root root     45590 Feb 16  2007 BootLogo.png
-rw-r--r-- 1 root   80      1024 Mar 15  2007 Desktop DB
-rw-r--r-- 1 root   80         2 Mar 15  2007 Desktop DF
-rw-rw-r-- 1 root   80 207475830 Mar 15  2007 OS.dmg
-rw-r--r-- 1 root root    298800 Feb 16  2007 boot.efi
-rw-r--r-- 1 root   80       520 Mar 15  2007 com.apple.Boot.plist
-rw-r--r-- 1 root root   6306364 Mar 15  2007 mach_kernel.prelink
```

Create a directory called "staging" and copy a few items from the update.
```
mkdir staging

cp -arp atv-update/mach_kernel.prelink staging/
cp -arp atv-update/System/Library/CoreServices/boot.efi staging/
cp -arp atv-update/System/Library/CoreServices/BootLogo.png staging/
cp -arp atv-update/System/Library/CoreServices/com.apple.Boot.plist staging/
```

Copy 2Z694-5428-3.dmg into your staging directory as OS.dmg. That's correct, the update becomes the "OS.dmg" image that the AppleTV EFI firmware will use to create the contents of "OSBoot".
```
cp -arp 2Z694-5428-3.dmg staging/OS.dmg
```

Edit "staging/com.apple.Boot.plist" and change it to look just like this. You will be changing "Boot Logo" and "Kernel Flags" strings.
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple Computer//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>Background Color</key>
        <integer>0</integer>
        <key>Boot Fail Logo</key>
        <string></string>
        <key>Boot Logo</key>
        <string>BootLogo.png</string>
        <key>Kernel</key>
        <string>mach_kernel</string>
        <key>Kernel Cache</key>
        <string>mach_kernel.prelink</string>
        <key>Kernel Flags</key>
        <string>rp=file:///OS.dmg</string>
</dict>
</plist>
```

Clean up
```
sudo umount atv-update
rmdir atv-update

rm 2Z694-5428-3.dmg
```

Done, the contents of the directory "staging" can now be copied to the "backup" partition on your Linux USB pen rescue disk. Proceed to the Restore section above.