# Support/FAQ #

This is a collection of various hint/tips to solutions for common errors or problems.


---

# My USB Flash drive does not boot #
A few things to check.
  1. Is the correct boot.efi present on the "recovery" partition? The md5 checksum for the correct boot.efi is
```
md5sum boot.efi
280323d8700e4cfef15116f7e50590e3  boot.efi
```
  1. Does the "recovery" partition have the correct GUID? While you can't see the GUID value, the patched parted should show the "atvrecv" flag which indicates the correct GUID.
```
sudo parted -s /dev/sdb unit s print

Model: SanDisk Cruzer Micro (scsi)
Disk /dev/sdb: 501759s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start   End      Size     File system  Name     Flags  
 1      40s     69671s   69632s                primary  atvrecv
```
  1. Does the "recovery" partition have the proper files?ubuntu:~$ ls -l /media/Recovery
 # Did you force a "recovery" boot by holding the "menu and "-" buttons down on the Apple IR Remote?

----
= X11 fails to launch =
X11 fails to load, you make changes in "/etc/X11/xorg.conf" to fix the problem and X11 still fails to load. WTF. X11 seems to be ignoring my changes. 

Some Linux installers will place "/etc/X11/xorg.conf.failsafe" for X11 to use if it encounters a failure in loading "/etc/X11/xorg.conf". Sometimes this failsafe is not correct and will also fail to load. Look at the "/var/log/Xorg.0.log" and you might see this
{{{
(++) Using config file: "/etc/X11/xorg.conf.failsafe"
}}}

Once X11 starts using the "/etc/X11/xorg.conf.failsafe", it becomes "stuck" using it and will not use "/etc/X11/xorg.conf". 

The solution is to "sudo rm "/etc/X11/xorg.conf.failsafe". Now you can fix "/etc/X11/xorg.conf" and move on.
```