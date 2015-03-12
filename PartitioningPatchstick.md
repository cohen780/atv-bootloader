# Partition for a Patchstick #

A basic patchstick serves one function and that is to enable remote ssh access into the AppleTV. To do this, a patchstick must be bootable and be able to mount and copy various sshd required files into the OSBoot file structure.

So first things first, let's make a bootable pen drive. This assumes that you have looked over the partition for Linux section to understand the basics. You will need to build the [patched parted](InstallParted.md) and the [hfs support tools](InstallHFSTools.md).

This assumes the pen drive is at "/dev/sda". We are just going to make the recovery partition and a second to hold the patchstick script and sshd support files. The second partition could be ext2, ext3 or hfsplus. Fat is not supported at the current time as darwin/linux permissions do not translate correctly.

Wipe the pen drive
```
# zero /dev/sda first or pre-existing guid will not change
sudo dd if=/dev/zero of=/dev/sda bs=4096 count=1M
```

Create the gpt partition on the pen drive
```
# create initial gpt structures
sudo parted -s /dev/sda mklabel gpt
```

Create the recovery partition
```
# create a 25MB "Recovery" partition  (starting at sector 40 is important)
sudo parted -s /dev/sda mkpart primary HFS 40s 25M
sudo parted -s /dev/sda set 1 atvrecv on
```

Create the "patchstick" partition
```
# create a 100MB "patchstick" partition
sudo parted -s /dev/sda mkpart primary HFS 25M 125M
```

Verify the partitions
```
# verify the partitions
sudo parted /dev/sda
print
```

You should see something similar (note the atvrecv flag)
```
Model: IC25N020 ATDA04-0 (scsi)
Disk /dev/sda: 256.0MB
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End     Size    File system  Name     Flags  
 1      20.5kB  25.0MB  25.0MB               primary  atvrecv   
 2      25.0MB  125.0MB 100.0MB              primary 
```

Format the created partitions
```
sudo mkfs.hfsplus -v Recovery /dev/sda1
sudo mkfs.hfsplus -v Patchstick /dev/sda2
sync 
```

Done


