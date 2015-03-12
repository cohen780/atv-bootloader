![http://atv-bootloader.googlecode.com/svn/branches/web_items/logos/ubuntu.png](http://atv-bootloader.googlecode.com/svn/branches/web_items/logos/ubuntu.png)

# Install [Ubuntu 7.10 Gutsy](http://www.ubuntu.com/getubuntu/download/) #

These instructions are for the installation of [Ubuntu 7.10 Gutsy](http://www.ubuntu.com/getubuntu/download/) from the LiveCD. This consist of a) install Gutsy, b) post install fixes and last c) post boot fixes.

**Install Ubuntu Gutsy**

> This assumes /dev/sda4 is "/" and /dev/sda5 is "swap". If your disk partitioning is different you will need to adjust this guide to match.

> Click on the "Install" desktop icon

> Prepare disk space
```
select "Manual"
```

> Prepare Partitions
```
select /dev/sda4 and "edit"
change "use as" to ext3
change mount point to "/" (without quotes)
check the format check box.

select /dev/sda5 and "edit"
change "use as" to swap

if you see any other marked partitions
select them and "edit"
change "use as" to dontuse

Install grub as we need the grub structure (menu.lst) to boot. We will run gptsync to fix up the MBR to one that efi firmware likes
so that the boot to the tux/atv logo drops from 30-60 seconds to 11.
This is done later when the install is running native on the AppleTV.
```

**Fix console framebuffer support**

> Ubuntu has a strange console framebuffer setup so we need to fix it for the AppleTV.

> If you are installing to a USB disk, unmount any USB drive partitions and then unplug/replug the USB drive to get the partitions re-mounted. The rest assumes the disk is mounted at /media/disk
```
sudo mount --bind /dev /media/disk/dev
#
sudo chroot /media/disk
df to make sure you are in the correct root filesystem
"/" should be mounted on /dev/sda4

#------------------------------------------------
#install ssh server
sudo apt-get install openssh-server
```

> You might see the following error but that's ok, sshd is installed but dpkg can not start it in chroot
```
invoke-rc.d: initscript ssh, action "restart" failed.
dpkg: error processing openssh-server (--configure):
 subprocess post-installation script returned error exit status 1
Errors were encountered while processing:
 openssh-server
E: Sub-process /usr/bin/dpkg returned an error code (1)
```

> Fix console framebuffer support by adding fncon and vesafb to the initramfs. Also add "8139too" to prevent "8139cp" from attempting to load.
```
#------------------------------------------------
# enable vesafb console support
sudo nano /etc/modprobe.d/blacklist-framebuffer and comment out "blacklist vesafb"
	#blacklist vesafb

sudo nano /etc/initramfs-tools/modules and add 
	fbcon
	vesafb
	8139too

sudo update-initramfs -u
```

> Done inside the chroot, exit and unmount /media/disk/dev
```
# exit chroot
exit
sudo umount /media/disk/dev
```

> Now you can a) reinstall the drive back into the AppleTV or b) install the drive into a 2.5 to USB drive adapter. Power on the AppleTV and you should see the tux/atv logo in 30-60 seconds. Then you will see the atv-bootloader kernel booting (tux in the upper left corner). After that your linux install should boot. If you are using a USB drive converter, you might have to use the IR remote and press "menu" and "-" buttons down during power on to force a USB recovery boot.

> You will have console framebuffer support but X11 will not startup when booting the AppleTV. X11 support requires the nvidia binary driver which installed later.

**Running Ubuntu Gutsy**

> There are a few things to complete the install once you have Ubuntu running. First update everthing then reboot.
```
sudo apt-get update
sudo apt-get upgrade
sudo reboot
```

> Several additional items need to be updated/patched for supporting AppleTV hardware. The following are on separate pages as the procedures are similar for other debian based distributions.
    * [Update nvidia driver](UpdateNvidiaDriver.md)
    * [Update audio driver](UpdateAudioDriver.md)
    * [Update IR driver](UpdateIRDriver.md)
    * [Update the boot disk MBR with gptsync](UpdateMBR.md)
    * [Install wireless driver](InstallWireless.md)

> Ok, that was not too hard, reboot to pickup the changes.
```
#------------------------------------------------
# pickup all the changes
sudo reboot
```

**Updating Ubuntu Gutsy**

> A few things to watch out for. If you take an update that changes the kernel or kernel modules, then you will have to redo the nvidia binary driver install and patch/rebuild the audio driver and IR driver. Because of this, you should disable kernel updates.
