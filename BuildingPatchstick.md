# Build a basic Patchstick #

**[Extract boot.efi](BootEFIExtraction.md)**

**[Partition for patchstick](PartitioningPatchstick.md)**

We have extracted boot.efi and partitioned a USB pen drive with a recovery and second hfsplus partitions using the above links. This guide assumes that the recovery partition is mounted at "/media/Recovery" and the second partition is mounted at "/media/Patchstick".

Download recovery.tar.gz and copy it to the recovery partition
```
# download recovery files
wget http://atv-bootloader.googlecode.com/files/recovery-0.6.tar.gz

sudo tar -xzf recovery-0.6.tar.gz
sudo cp -arp recovery/* /media/Recovery/
```

Don't forget to copy boot.efi to the recovery partition
```
sudo cp -ap boot.efi /media/Recovery
```

Now make a simple patchstick script to test functionality. There is an existing sample patchstick.sh in recovery.tar.gz and we are just moving to the second partition where we will place all the other bits. You could just keep everything on the recovery partition (make it larger if you do this) but this keeps booting and patching separate.
```
mv /media/Recovery/patchstick.sh /media/Patchstick/
```

edit com.apple.Boot.plist and change the following line
```
<string>atv-boot=auto video=vesafb</string>
```

to
```
<string>atv-boot=patchstick video=vesafb</string>
```

Now sync and unmount both partitions
```
sync
umount /media/Recovery
umount /media/Patchstick
```

Pull the USB pen drive and place into the USB port on the AppleTV and reboot/power on. You might have to use the IR remote and press "menu" and "-" buttons down during power on to force a USB recovery boot.

You should see the tux/atv logo, then Linux kernel boot and then finally
```
this is a patchstick stript
```

If you have a USB keyboard attached, you can login (user=root password=root) and poke around. The USB pen drive will be at "/dev/sdb" and the internal ata drive is at "/dev/sda".


---

# example patchstick.sh #

Fill in "patchstick.sh" with the patchstick function you want to perform. Here's an example of a sshd install contributed by James Abeler of [Apple Core, LLC](http://www.applecorellc.com/). Thanks James, for being the first prove this method of patching the AppleTV OS. This script assumes that you have created a directory "ssh" on the "Patchstick" partition of the USB flash drive and populate it with the required files.
```
#!/bin/bash

exec 2>/dev/console
exec 1>/dev/console

echo
echo "        --- aTV Flash 2.0.1 Upgrade ---"

echo "        * mounting OSBoot partition"
mkdir /mnt/OSBoot
fsck.hfsplus /dev/sda3
mount -t hfsplus -o rw,force /dev/sda3 /mnt/OSBoot

#echo "        * mounting stuff partition r/o"
mkdir /mnt/stuff
fsck.hfsplus /dev/sdb2
mount -t hfsplus -r /dev/sdb2 /mnt/stuff

echo "        * keeping the OSBoot partition r/w for plugins"
touch /mnt/OSBoot/.readwrite

if [ -f /mnt/rootfs/stuff/ssh/sshd ] && [ -f /mnt/rootfs/stuff/ssh/ssh.plist ]; then
  echo -n "        * Installing SSH daemon... "
  cp /mnt/rootfs/stuff/ssh/sshd /mnt/OSBoot/usr/sbin/sshd
  cp /mnt/rootfs/stuff/ssh/ssh /mnt/OSBoot/usr/bin/ssh
  cp /mnt/rootfs/stuff/ssh/ssh-add /mnt/OSBoot/usr/bin/ssh-add
  cp /mnt/rootfs/stuff/ssh/ssh-agent /mnt/OSBoot/usr/bin/ssh-agent
  cp /mnt/rootfs/stuff/ssh/ssh-keygen /mnt/OSBoot/usr/bin/ssh-keygen
  cp /mnt/rootfs/stuff/ssh/ssh-keyscan /mnt/OSBoot/usr/bin/ssh-keyscan
  cp /mnt/rootfs/stuff/ssh/scp /mnt/OSBoot/usr/bin/scp
  cp /mnt/rootfs/stuff/ssh/sftp-server /mnt/OSBoot/usr/libexec/sftp-server
  cp /mnt/rootfs/stuff/ssh/ssh-keysign /mnt/OSBoot/usr/libexec/ssh-keysign
  cp /mnt/rootfs/stuff/ssh/sshd-keygen-wrapper /mnt/OSBoot/usr/libexec/sshd-keygen-wrapper
  cp /mnt/rootfs/stuff/ssh/ssh.plist /mnt/OSBoot/System/Library/LaunchDaemons/ssh.plist
  
  chmod 755 /mnt/OSBoot/usr/sbin/sshd
  chmod 755 /mnt/OSBoot/usr/bin/ssh
  chmod 755 /mnt/OSBoot/usr/bin/ssh-add
  chmod 755 /mnt/OSBoot/usr/bin/ssh-agent
  chmod 755 /mnt/OSBoot/usr/bin/ssh-keygen
  chmod 755 /mnt/OSBoot/usr/bin/ssh-keyscan
  chmod 755 /mnt/OSBoot/usr/bin/scp
  chmod 755 /mnt/OSBoot/usr/libexec/sftp-server
  chmod 4755 /mnt/OSBoot/usr/libexec/ssh-keysign
  chmod 755 /mnt/OSBoot/usr/libexec/sshd-keygen-wrapper
  chmod 755 /mnt/OSBoot/System/Library/LaunchDaemons/ssh.plist

  # echo "        Adding Kerberos..."
  # cp -p -R /System/Library/Frameworks/OSXFrames/Kerberos.framework /OSBoot/System/Library/Frameworks/OSXFrames/
  # sed -i"" -e 's;^exec;DYLD_FRAMEWORK_PATH="/System/Library/Frameworks/OSXFrames" exec;' /OSBoot/usr/libexec/sshd-keygen-wrapper
  echo "        done."
else
  echo "        SSH files not found."
fi

#--------------------------------------------------------------------------

  sync &>/dev/null
  umount /mnt/OSBoot
  echo "        all done!"
  
echo
echo "        The aTV Flash software has been successfully installed.  You may now remove the flash drive and restart your Apple TV."
echo "        Thanks from AppleCoreLLC.com"
echo "        :-)"
sleep 100000
```


---

Here's an another example of installing "sshd". Nothing is auto mounted so you will need to mount any disk partitions manually. This also assumes that you have created a directory "ssh" on the "Patchstick" partition of the USB flash drive and populate it with the required files.
```

#!/bin/bash
# On Linux, bash is located at "/bin/bash" not "/sbin/bash"

# make some mount points
mkdir /stuff /OSBoot

# mount the USB flash drive "Patchstick" partition 
mount /dev/sdb2 /stuff

# always "fsck.hfsplus -f" hfsplus journaled partitions first
fsck.hfsplus -f /dev/sda3

# mount "OSBoot" forcing it read/write
mount -t hfsplus -o rw,force /dev/sda3 /OSBoot

# patchstick scripts under Linux look the same as under OSX
# it's just bash after all.

# install ssh
# (c) 2007 macTijn at awkwardtv dot org

if [ -f /stuff/ssh/sshd ] && [ -f /stuff/ssh/ssh.plist ]; then
  echo -n "        * Installing SSH daemon... "
  cp /stuff/ssh/sshd /OSBoot/usr/sbin/sshd
  cp /stuff/ssh/ssh /OSBoot/usr/bin/ssh
  cp /stuff/ssh/ssh-add /OSBoot/usr/bin/ssh-add
  cp /stuff/ssh/ssh-agent /OSBoot/usr/bin/ssh-agent
  cp /stuff/ssh/ssh-keygen /OSBoot/usr/bin/ssh-keygen
  cp /stuff/ssh/ssh-keyscan /OSBoot/usr/bin/ssh-keyscan
  cp /stuff/ssh/scp /OSBoot/usr/bin/scp
  cp /stuff/ssh/sftp-server /OSBoot/usr/libexec/sftp-server
  cp /stuff/ssh/ssh-keysign /OSBoot/usr/libexec/ssh-keysign
  cp /stuff/ssh/sshd-keygen-wrapper /OSBoot/usr/libexec/sshd-keygen-wrapper
  cp /stuff/ssh/ssh.plist /OSBoot/System/Library/LaunchDaemons/ssh.plist
  
  chmod 755 /OSBoot/usr/sbin/sshd
  chmod 755 /OSBoot/usr/bin/ssh
  chmod 755 /OSBoot/usr/bin/ssh-add
  chmod 755 /OSBoot/usr/bin/ssh-agent
  chmod 755 /OSBoot/usr/bin/ssh-keygen
  chmod 755 /OSBoot/usr/bin/ssh-keyscan
  chmod 755 /OSBoot/usr/bin/scp
  chmod 755 /OSBoot/usr/libexec/sftp-server
  chmod 4755 /OSBoot/usr/libexec/ssh-keysign
  chmod 755 /OSBoot/usr/libexec/sshd-keygen-wrapper
  chmod 755 /OSBoot/System/Library/LaunchDaemons/ssh.plist

  echo "        Done."
fi

```