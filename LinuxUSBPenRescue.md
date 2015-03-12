# AppleTV - Linux USB pen drive #

Here's the situation, you have made a bad choice in rebuilding a kernel and now your AppleTV Linux install panics on boot. How do you fix it? Typically you boot a LiveCD and restore the previous kernel but a cd based boot is not possible yet. Well, that's where a atv-bootloader based USB pen drive rescue boot saves the day.

In this section, we will build an atv-bootloader based USB pen drive that can be used for standalone boot. We will also enable telnet so we don't need a USB keyboard attached and can do everything remotely using a telnet session using a wired network connection.

# Details #

We need our standard items for creating an AppleTV recovery partition. These are boot.efi, the patched parted and hfs tools. See [extract boot.efi](BootEFIExtraction.md) for boot.efi, [install parted](InstallParted.md) for parted and [install hfs tools](InstallHFSTools.md) for hfs support].

Assuming you have these items extracted/built/installed, let's start.

Create just the recovery partition on a USB pen drive, this does not require very much space so 64MB or greater pen drive is fine. This guide will assume the device at "/dev/sdb" is the pen drive so remember to adjust this to match your setup. Also make sure the USB flash drive is unmounted, most Linux distros will auto-mount USB flash drives. If you have problems partitioning, you might have a [USB flash drive that needs fixing](FixingUSBFlashDrives.md).


```
# zero the initial sectors
sudo dd if=/dev/zero of=/dev/sdb bs=4096 count=1M

# create the GPT format
sudo parted -s /dev/sdb mklabel gpt

# create just a recovery partition
sudo parted -s /dev/sdb mkpart primary HFS 40s 69671s
sudo parted -s /dev/sdb set 1 atvrecv on

# sync the system partition tables
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
# remember to copy boot.efi too
sudo cp -ap boot.efi penboot/
```

We going to override the normal auto boot sequence as we need to get into a command-line enviroment to "fix" our problem. Now edit "penboot/com.apple.Boot.plist" on the recovery partition and change the atv-boot parameter from "auto"
```
<string>atv-boot=auto video=vesafb</string>
```

to "manual"
```
<string>atv-boot=manual video=vesafb</string>
```

When "manual" is selected, atv-bootloader now searches for a file called "boot\_linux.sh" on any disk partition and execute it. The search path starts with USB devices. The recovery.tar.gz contents contains a sample "boot\_linux.sh" with the kexec load commented out. So edit "penboot/boot\_linux.sh" and add the following
```
ifconfig eth0 0.0.0.0
/sbin/udhcpc

sleep 4

ifconfig

telnetd -l /bin/login
```

This does several things, enables wired networking using DHCP, prints the network config so you know the IP address and starts the telnet demon.

Ok, unmount and we are done building the USB rescue pen drive.
```
sudo umount penboot
```

# Usage #

Boot the AppleTV using the USB rescue pen drive. You should see the kernel boot messages, the ifconfig dump and then a login prompt. Either login (user=root, password=root) using a USB keyboard or telnet in using the listed IP addresses. vi and nano are present as well as the parted and hfs tools in case you need to create AppleTV type partitions.

Enjoy.
