![http://atv-bootloader.googlecode.com/svn/branches/web_items/logos/xbmc.png](http://atv-bootloader.googlecode.com/svn/branches/web_items/logos/xbmc.png)

# Installing [LiveXBMC](http://xbmc.org/) #

LiveXBMC is a development release of XBMC on a small USB flash drive. It's still under development and as such unsupported by the main developers, see this [link to the forum thread](http://xbmc.org/forum/showthread.php?t=32853).

This one is dirt simple. LiveXBMC is installed by extracting the contents of the disk image that would be created for tradition PC hardware. This flash drive needs to be 512MB or larger.

LiveXBMC expects to boot off the first partition so we have run do a modified install of atv-bootloader. In this case, the first partition holds LiveXBMC and the second will be "Recovery" containing atv-bootloader. This is reversed from normal but the AppleTV EFI firmware does not care.

If your USB flash drive auto-mounts, umount it as we will explicitly mount it with this guide

We need our standard items for creating an AppleTV "Recovery" partition. These are boot.efi, the patched parted and hfs tools. See [extract boot.efi](BootEFIExtraction.md) for boot.efi, [install parted](InstallParted.md) for parted and [install hfs tools](InstallHFSTools.md) for hfs support. Test that you have to correct parted.
```
parted --version

parted (GNU parted) 1.8.8
Copyright (C) 2007 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Written by <http://parted.alioth.debian.org/cgi-bin/trac.cgi/browser/AUTHORS>.
```

Assuming you have these items extracted/built/installed, let's start. Create the partitions, this guide assumes the USB flash disk is located at "/dev/sdb", you will need to adjust this guide if your flash disk is located elsewhere.
```
# zero the initial sectors
sudo dd if=/dev/zero of=/dev/sdb bs=4096 count=1M

# sync the system disk partition tables
sudo partprobe /dev/sdb

# create the GPT format
sudo parted -s /dev/sdb mklabel gpt
```

Print the disk info with "sudo parted -s /dev/sdb unit s print" so we can figure out the partitioning
```
Model: Flash Drive SM_USB20 (scsi)
Disk /dev/sdb: 2030592s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start  End  Size  File system  Name  Flags
```

Total sectors = 2030592. So ending sector is 2030592 - 34 = 2030558. We will use 50MB for "Recovery", 50MB = 52,428,800 bytes or 52,428,800 / 512 = 102400 sectors. 2030558 - 102400 = 1928158, round down to keep the starting sector even and 1928149 is the ending sector of the first partition. Rock and roll.
```
# create the LiveXBMC partition
sudo parted -s /dev/sdb mkpart primary ext2 40s 1928149s

# create the recovery partition
sudo parted -s /dev/sdb mkpart primary HFS 1928150s 2030558s
sudo parted -s /dev/sdb set 2 atvrecv on

# sync the system disk partition tables
sudo partprobe /dev/sdb

sudo mkfs.ext2 -L LiveXBMC /dev/sdb1
sudo mkfs.hfsplus -v Recovery /dev/sdb2
```

Verify that it looks fine and the atvrecv flag is set
```
sudo parted -s /dev/sdb unit s print

Model: Flash Drive SM_USB20 (scsi)
Disk /dev/sdb: 2030592s
Sector size (logical/physical): 512B/512B
Partition Table: gpt

Number  Start     End       Size      File system  Name     Flags  
 1      40s       1928149s  1928110s  ext2         primary         
 2      1928150s  2030558s  102409s   hfs+         primary  atvrecv
```

Download the compressed disk image and the update
```
wget http://superb-east.dl.sourceforge.net/sourceforge/xbmc/LiveXBMCV2.12835.7z
wget http://superb-east.dl.sourceforge.net/sourceforge/xbmc/xbmc.12869.img.7z
```

Decompress both to extract the disk images
```
sudo apt-get install p7zip-full

7z x LiveXBMCV2.12835.7z
7z x xbmc.12869.img.7z
```


Make some mount points. Mount the USB flash drive partitions and the base xbmc disk image so we can get to the contents. Then copy the base xbmc contents to first partiton of the USB flash drive. Remember to replace "sdb" below with your device notation.
```
mkdir penboot penflash xbmc

sudo mount /dev/sdb1 penflash
sudo mount /dev/sdb2 penboot

sudo mount -o loop LiveXBMCV2.12835.img xbmc

sudo cp -arp xbmc/* penflash/
sync
```

Update the base image.
```
sudo cp -arp xbmc.12869.img penflash/xbmc.img
sync
```


Download "recovery.tar.gz", this is avt-bootloader. Copy it to the second partition ("recovery").
```
# download atv-bootloader (recovery.tar.gz) and install it
wget http://atv-bootloader.googlecode.com/files/recovery-0.6.tar.gz
sudo tar -xzf recovery-0.6.tar.gz
sudo cp -arp recovery/* penboot/
```

Remember to copy boot.efi to penboot/. See ["boot.efi" extraction](BootEFIExtraction.md) for details.
```
sudo cp -ap boot.efi penboot/
```

Create a custom "boot\_linux.sh" that will be used to boot LiveXBMC. Edit "penboot/boot\_linux.sh" and change it to the following
```
#!/bin/bash

mkdir /xbmc
mount /dev/sdb1 /xbmc

kexec --load /xbmc/vmlinuz \
--initrd=/xbmc/initrd0.img \
--command-line="initrd=initrd0.img boot=usb video=vesafb" 

kexec -e
```

Edit "penboot/com.apple.Boot.plist" on the first partition of the USB flash disk and change the atv-boot parameter to "manual"
```
<string>atv-boot=manual video=vesafb</string>
```

Clean up
```
sudo umount penboot penflash xbmc
rmdir penboot penflash xbmc
```

Done.

# Usage #

Boot the AppleTV using the USB flash disk. Remember you have to force a "Recovery Boot" by holding "menu" and "-" buttons down on the Apple IR remote either during power-up or when the AppleTV OS is running.

You should see the kernel boot messages, then LiveXBMC booting. There will be a liittle delay as the file system is read from the flash drive. LiveXBMC boots to a command prompt, start X11 using
```
startx
```

You can auto start X11 by adding "splash" to the kernel command line params. If you do this then edit "/etc/usplash.conf" to match your display resolution.

Enjoy.

# Post Boot Fixes #

  * Install openssh-server for remote ssh sessions. username is "xbmc", password is "password".
```
sudo apt-get install openssh-server
```

  * Add the following to "/etc/modprobe.d/options" to enable analog audio and Apple IR device support.
```
options snd-hda-intel model=imac24
options usbhid quirks=0x05ac:0x8241:0x10
```

  * Add the following to the Section "Screen" in "/etc/X11/xorg.conf".
```
    Option         "Coolbits" "1"
    Option         "UseEvents" "1"
```

  * Using the Microsoft MCE IR remote control (mceusb). Edit "/etc/lirc/hardware.conf" and change it to this
```
# /etc/lirc/hardware.conf
#
#Chosen Remote Control
REMOTE="Microsoft MCE IR Remote"
REMOTE_MODULES="lirc_dev lirc_mceusb2"
REMOTE_DRIVER=""
REMOTE_DEVICE="/dev/lirc0"
REMOTE_LIRCD_CONF=""
REMOTE_LIRCD_ARGS=""

#Chosen IR Transmitter
TRANSMITTER="None"
TRANSMITTER_MODULES=""
TRANSMITTER_DRIVER=""
TRANSMITTER_DEVICE=""
TRANSMITTER_LIRCD_CONF=""
TRANSMITTER_LIRCD_ARGS=""

#Enable lircd
START_LIRCD="true"

#Don't start lircmd even if there seems to be a good config file
#START_LIRCMD="false"

#Try to load appropriate kernel modules
LOAD_MODULES="true"

# Default configuration files for your hardware if any
LIRCMD_CONF=""

#Forcing noninteractive reconfiguration
#If lirc is to be reconfigured by an external application
#that doesn't have a debconf frontend available, the noninteractive
#frontend can be invoked and set to parse REMOTE and TRANSMITTER
#It will then populate all other variables without any user input
#If you would like to configure lirc via standard methods, be sure
#to leave this set to "false"
FORCE_NONINTERACTIVE_RECONFIGURATION="false"
START_LIRCMD=""
```

> Here is the "/etc/lirc/lircd.conf" for the mceusb remote
```
#
# RC-6 config file
#
# source: http://home.hccnet.nl/m.majoor/projects__remote_control.htm
#         http://home.hccnet.nl/m.majoor/pronto.pdf
#
# used by: Philips
#
#########
#
# Philips Media Center Edition remote control
# For use with the USB MCE ir receiver
#
# Dan Conti  dconti|acm.wwu.edu
#
# Updated with codes for MCE 2005 Remote additional buttons
# *, #, Teletext, Red, Green, Yellow & Blue Buttons
# Note: TV power button transmits no code until programmed.
# Updated 12th September 2005
# Graham Auld - mce|graham.auld.me.uk
#
# Radio, Print, RecTV are only available on the HP Media Center remote control
#

begin remote

  name mceusb
  bits           16
  flags RC6|CONST_LENGTH
  eps            30
  aeps          100

  header       2667   889
  one           444   444
  zero          444   444
  pre_data_bits 21
  pre_data      0x37FF0
  gap          105000
  toggle_bit     22
  rc6_mask     0x100000000


      begin codes

        Blue    0x00007ba1
        Yellow  0x00007ba2
        Green   0x00007ba3
        Red     0x00007ba4
        Teletext        0x00007ba5

# starts at af
        Radio    0x00007baf
        Print    0x00007bb1
        Videos   0x00007bb5
        Pictures 0x00007bb6
        RecTV    0x00007bb7
        Music    0x00007bb8
        TV       0x00007bb9
# no ba - d8

        Guide    0x00007bd9
        LiveTV   0x00007bda
        DVD      0x00007bdb
        Back     0x00007bdc
        OK       0x00007bdd
        Right    0x00007bde
        Left     0x00007bdf
        Down     0x00007be0
        Up       0x00007be1

        Star       0x00007be2
        Hash       0x00007be3

        Replay   0x00007be4
        Skip     0x00007be5
        Stop     0x00007be6
        Pause    0x00007be7
        Record   0x00007be8
        Play     0x00007be9
        Rewind   0x00007bea
        Forward  0x00007beb
        ChanDown 0x00007bec
        ChanUp   0x00007bed
        VolDown  0x00007bee
        VolUp    0x00007bef
        More     0x00007bf0
        Mute     0x00007bf1
        Home     0x00007bf2
        Power    0x00007bf3
        Enter    0x00007bf4
        Clear    0x00007bf5
        Nine     0x00007bf6
        Eight    0x00007bf7
        Seven    0x00007bf8
        Six      0x00007bf9
        Five     0x00007bfa
        Four     0x00007bfb
        Three    0x00007bfc
        Two      0x00007bfd
        One      0x00007bfe
        Zero     0x00007bff
      end codes

end remote
```

> Then
```
sudo /etc/init.d/lirc restart
```

> Test using "irw", when you press a button on the remote control, you should see a response and LiveXBMC should see it now.
```
username# irw
000000037ff07be1 00 Up mceusb 
000000037ff07be1 01 Up mceusb 
000000037ff07be0 00 Down mceusb 
000000037ff07be0 01 Down mceusb 
000000037ff07bde 00 Right mceusb 
000000037ff07bde 01 Right mceusb 
000000037ff07bdf 00 Left mceusb 
```