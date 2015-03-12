# Create and format proper AppleTV partitions #

**April 27, 2008 - ATV-Bootloader has been updated**

This guide details how to properly create and format the required GPT partitions on the internal PATA disk using a [USB flash drive with atv-bootloader installed](LinuxUSBPenBoot.md). The internal PATA disk is typically "/dev/sdaX" and you can target an external USB disk by change the device "/dev/sdcX", "/dev/sddX", etc). Remember that these instructions are a guide not a script to cut and paste. They will need to be altered (drive identifiers, sector counts, etc) to suite your particular setup.


**WARNING**
If you are installing to the original AppleTV internal drive, you should [backup the original hard drive](ATVBackup.md).

# Details #

You don't actually need to create the "EFI" and "OSBoot" partitions, the AppleTV will boot fine without them. I always include them on the internal disk just in case the Apple EFI firmware tries any tricks on me in the future.
```
# zero /dev/sda first or pre-existing guid will not change
dd if=/dev/zero of=/dev/sda bs=4096 count=1M

# create initial gpt structures
parted -s /dev/sda mklabel gpt

# find max size of disk (see Disk  /dev/sda: XXXMB from listing)
parted -s /dev/sda print

# mine reports "Disk /dev/sda: 20.0GB" so that is our ending point

# create a 25MB "EFI" partition (starting at sector 40 is important)
parted -s /dev/sda mkpart primary fat32 40s 25M
parted -s /dev/sda set 1 boot on

# create a 25MB "Recovery" partition
parted -s /dev/sda mkpart primary HFS 25M 50M
parted -s /dev/sda set 2 atvrecv on

# create a 25MB "OSBoot" partition
parted -s /dev/sda mkpart primary HFS 50M 75M

#create the linux root partition
parted -s /dev/sda mkpart primary ext3 75M 18.9GB

#create the linux swap partition
parted -s /dev/sda mkpart primary linux-swap 18.9GB 20.0GB

# sync the system partition tables
partprobe /dev/sda

# verify the partitions
parted -s /dev/sda print

# you should see something similar (note boot and atvrecv flags)
Model: IC25N020 ATDA04-0 (scsi)
Disk /dev/sda: 20.0GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name     Flags  
 1      20.5kB  25.0MB  25.0MB               primary  boot   
 2      25.0MB  50.0MB  25.0MB               primary  atvrecv
 3      50.0MB  75.0MB  25.0MB               primary         
 4      75.0MB  18.9GB  18.8GB               primary         
 5      18.9GB  20.0GB  1104MB               primary         

# format the partitions
# we will let the LiveCD install setup swap
mkfs.msdos -F 32 -n EFI /dev/sda1
mkfs.hfsplus -v Recovery /dev/sda2
mkfs.hfsplus -v OSBoot /dev/sda3
mkfs.ext3  -b 4096 -L Linux /dev/sda4
sync 
```

**install atv-bootloader
```
# download recovery files
wget http://atv-bootloader.googlecode.com/files/recovery-0.6.tar.gz
tar -xzf recovery-0.6.tar.gz

# make some mount points
mkdir /mnt/osboot /mnt/recovery

# mount the partitions
fsck.hfsplus /dev/sda2
mount /dev/sda2 /mnt/recovery
mount /dev/sda3 /mnt/osboot

# copy atv-bootloader over
cp -arp recovery/* /mnt/osboot/
cp -arp recovery/* /mnt/recovery/

# remember to copy boot.efi, 
# grab it from the atv-bootloader USB flash disk

mkdir tmp
mount /dev/sdb1 tmp

cp -ap tmp/boot.efi /mnt/osboot
cp -ap tmp/boot.efi /mnt/recovery
```**

**Done, now proceed to install your linux distro
```
remember to install to /dev/sda4 with swap at /dev/sda5
```**
