# Development Blog #

This blog contains the development notes of what I'm working on. Check back frequently and you can see what I'm currently doing regarding atv-bootloader and Linux installation.

# October 4, 2008 #
Well, this has been a fun month, a triple play, atvusb-creator, xbmc-launcher and XBMC/Boxee for Mac, all for the AppleTV. And remember, you can use atvusb-creator to make basic atv-bootloaders. There's more to come under atvusb-creator including a creators for a few USB based Linux distros.

# August 9, 2008 #
I'm back. I've been vectored the last few weeks on a work related crunch and that's over now so I'll be able to spend time on atv-bootloader again. Coming up in the next few weeks is a new version that addresses a few minor issues and hopefully the release of the special project that I've been developing. This special project should make it much, much easier to create an atv-bootloader USB flash drive for Linux usage.


# July 16, 2008 #
This just hot off the presses. Seems someone has done the dirty on good old osputil. Be prepared for LED and fan control coming to an Linux based AppleTV near you. See the discussion groups thread "Random rebooting problem" for a play by play.

# July 8, 2008 #
Yes I know about the release of Knoppmyth 5.5. A install guide is brewing and should pop out this weekend. My time is being occupied with the rapid development of a special project that I can't talk about yet. I can say it's going very, very cool.

# July 4, 2008 #
And another bingo. I've been looking for a way to turn off the nvidia binary driver use of TurboCache. TurboCache basically uses part of system memory for video memory. Useful if you want to say you video card has more memory than it physically has but totally useless on something like the AppleTV where with only 256MB of physical ram, stealing 64MB for vram hurts. But no more, even though NVIDIA says there's no way to disable TurboCache under Linux, there is. This comes from a post on the nvidia forums which says to add the following in the Driver "nvidia" section of xorg.conf and restart xorg.
```
Option "RegistryDwords" "RMDisableRenderToSysmem=1"
```

This seems to work with nvidia binary versions 173.14.09 and 169.12.
```
Before:
(II) NVIDIA(0): NVIDIA GPU GeForce Go 7300 (G72) at PCI:1:0:0 (GPU-0)
(--) NVIDIA(0): Memory: 131072 kBytes
(--) NVIDIA(0): VideoBIOS: 05.72.22.68.00

After:
(II) NVIDIA(0): NVIDIA GPU GeForce Go 7300 (G72) at PCI:1:0:0 (GPU-0)
(--) NVIDIA(0): Memory: 65536 kBytes
(--) NVIDIA(0): VideoBIOS: 05.72.22.68.00
```

# July 3, 2008 #
Bingo! On another note, I have boot.efi extraction from the AppleTV 2.02 update from Windows using single GPL tool. This solved the last item of creating an OSX/Linux/Windows application that can create atv-bootloader on a USB flash drive dynamically from a downloaded disk image. It will make hacking the AppleTV for Linux and patchsticks using a pre-crafted disk image really easy and most important completely legal.  So what's this magic GPL tool? You will never guess. 7-Zip. That right 7-Zip now knows how to open a compressed DMG image. You need to the the alpha 3 release [7-Zip 4.59 alpha 3](http://www.7-zip.org/alpha/7z459a3.exe). Extraction is a two step from the command line. The first step extract the actual hfsplus filesystem.
```
7z e 2Z694-5428-3.dmg 2.hfs
```
The second step extracts boot.efi from the hfsplus filesystem
```
7z e 2.hfs OSBoot\System\Library\CoreServices\boot.efi
```
And out pops boot.efi. Kudos to the the 7-Zip authors for adding compressed DMG extraction and hfsplus file system handling.

# June 24, 2008 #
Tricky little sucker, I need to use x86emu to get the vbios to post and no one seems to claim ownership to x86emu so I have to figure it out from scratch. Coreboot uses it (they ignored my email to their list), Xorg uses it (not going there with questions about x86emu).

In answer to some questions about why worry about vbios. If I can get vbios POST'ed and working that means I can change the console video modes using standard Linux methods. If I can do it, then Linux can do it which means vgacon would work and most Linux distros expect vgacon to be present and working. Then all the work to force vesafb as a console frame buffer would not be required anymore. In addition, I need vgacon to support vbetool in order to save the video state for S3 (to ram) and S4 (to disk) hibernation.

And my home internet is down. Bummer.

# June 21, 2008 #
Still working on vbios. For giggles, here the output from POST
```
GeForce Go 7300 VGA BIOS (Apple M63)
Version 5.77.22.68.00
Engineering Release - Not for Production Use
Copyright (C) 1996-2006 NVIDIA corp.
```
Not for Production Use, that's funny.

# June 18, 2008 #
Got POST. Right now it's tricky. Running a modified atv-bootloader that does not reserve 0xA0000-0xBFFFF (Video RAM) and copies the nvidia bios from 0x21700000 to 0xC0000. Then boot a linux distro with X11 running the nv driver (which works fine). Then I can run testbios from the coreboot project and I get POST output to the screen. Still don't have console frame buffer (POST is too late) but I can change video modes. Now to move all this into atv-bootloader and create the INT10 hooks.

# June 15, 2008 #
Super woot! Finally found the nvidia vbios. Hardware details about the nvidia chipset is very hard to obtain. nvclock has some info but it's dump\_bios routine always returned garbage. Now thanks to the nouveau project, some more info was revealed about nvidia chipsets. The important part is where the vbios is located. For standard PCs, vbios can be found at 0xc0000 at runtime. It's the responsibility of the PC-bios to move it to that location. But the AppleTV is pure EFI and no one but Apple knows how the video bios is handled. I knew it was present because the nvidia binary could find it under X11, but it never showed up in any ram dumps. Well, it located off the nvidia chipset at 0x21000000 (first pci bar address) plus 0x00700000 (NV\_PRAMIN\_OFFSET) or at 0x21700000. Here's a dump of the first few pages.
```
00000000  55 aa 76 eb 4b 37 34 30  30 e9 4c 19 77 cc 56 49  |U.v.K7400.L.w.VI|
00000010  44 45 4f 20 0d 00 00 00  08 01 69 11 00 00 49 42  |DEO ......i...IB|
00000020  4d 20 56 47 41 20 43 6f  6d 70 61 74 69 62 6c 65  |M VGA Compatible|
00000030  01 00 00 00 c0 10 9e 8e  30 39 2f 30 36 2f 30 37  |........09/06/07|
00000040  00 00 00 00 00 00 00 00  01 10 00 00 00 00 00 00  |................|
00000050  e9 5d cd 00 00 00 de 10  ff ff ff 7f 00 00 00 00  |.]..............|
00000060  ff ff ff 7f 00 00 00 80  22 00 a5 c1 e9 c3 ab e9  |........".......|
00000070  ca ab 50 4d 49 44 6c 00  6f 00 00 00 00 a0 00 b0  |..PMIDl.o.......|
00000080  00 b8 00 c0 00 33 7e ca  9b 00 02 00 04 00 6a 21  |.....3~.......j!|
00000090  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
000000a0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
000000b0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
000000c0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
000000d0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
000000e0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
000000f0  ff ff ff ff ff ff ff ff  ff ff ff ff ff ff ff ff  |................|
00000100  ff ff ff ff 48 57 45 41  50 43 49 52 de 10 d7 01  |....HWEAPCIR....|
00000110  00 00 18 00 00 00 00 03  76 00 01 00 00 80 00 00  |........v.......|
00000120  47 65 46 6f 72 63 65 20  47 6f 20 37 33 30 30 20  |GeForce Go 7300 |
00000130  56 47 41 20 42 49 4f 53  20 28 41 70 70 6c 65 20  |VGA BIOS (Apple |
00000140  4d 36 33 29 0d 0a 00 00  00 00 00 00 00 00 00 00  |M63)............|
00000150  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000160  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
00000170  00 56 65 72 73 69 6f 6e  20 20 35 2e 37 32 2e 32  |.Version  5.72.2|
00000180  32 2e 36 38 2e 30 30 20  0d 0a 00 43 6f 70 79 72  |2.68.00 ...Copyr|
00000190  69 67 68 74 20 28 43 29  20 31 39 39 36 2d 32 30  |ight (C) 1996-20|
000001a0  30 36 20 4e 56 49 44 49  41 20 43 6f 72 70 2e 0d  |06 NVIDIA Corp..|
000001b0  0a 00 00 00 ba 91 98 96  91 9a 9a 8d 96 91 98 df  |................|
000001c0  ad 9a 93 9a 9e 8c 9a df  d2 df b1 90 8b df b9 90  |................|
000001d0  8d df af 8d 90 9b 8a 9c  8b 96 90 91 df aa 8c 9a  |................|
000001e0  f2 f5 ff ff ff ff ff 47  65 46 6f 72 63 65 20 47  |.......GeForce G|
000001f0  6f 20 37 33 30 30 00 20  20 20 20 00 00 00 00 00  |o 7300.    .....|
```

This is a very good result and might mean that the issues with video console frame buffers can be solved by dynamically extracting the vbios and locating it where Linux can find it. Tomorrow will be a busy day for me. If this works, then lots of possibilities open up.

# June 13, 2008 #
Action time. Warning on svn pulls, this is now the development version for recovery-0.7 which include kernel and initramfs construction. It's still a bit rough so if you have questions email or post to the discussion list.


# May 28, 2008 #
Still working on enhancements, and a really, really small test build of a mythbuntu frontend based distro that runs on a 512MB flash drive. Don't ask for it yet, it's a proof-of-concept that is still cooking.

# May 20, 2008 #
Now that was a really, really dumb error. Busybox is used in the initramfs and for those that don't know busybox replaces many standard Linux apps with much smaller applets that are built into busybox. There are two ways to reference the applet a) direct (busybox ls) or b) a sumlink (ls -> busybox). I use the symlink method, it's pretty standard. Well, the problem was that I added a Linux app (fsck) that just happened to already exist as a symlink to busybox. Teach me to use "cp -af". This cause "fsck" to replace busybox (the copy followed the symlink) and thus broke the boot because "/init" failed which the kernel reports as "can't find /init". Duh, as Homer says.

Now with that problem understood and fixed, I can press on with other issues and hopefully pop a new release in a few days.

# May 19, 2008 #
Still here, just busy breaking the boot. kernel is being cranky and does not want to run "/init", can't find it. Something bad is happening so I'm spending lots of time doing the old, touch code, build, test, buzzz - wrong, touch code, rinse-repeat-cycle.

# May 10, 2008 #
I've been putting this off for awhile and finally got some quiet time to solve it. I wanted to figure out which disk interface atv-bootloarder was booting from, the internal PATA or an external USB. This way I could skip the 5 second delay for USB device detection if booting off the internal PATA disk for fast booting. The boot device can be found as a node on the flat device-tree that EFI passes to a mach kernel.

First I need a copy of the flat device-tree so I can decode it outside the boot process. This turns out to be tricky since atv-bootloader has no file i/o capabilities before the embedded Linux kernel loads and after the embedded Linux kernel loads it's too late. So one must be sneaky and create a hole in the e820 memory map where the flat device tree is located. Now Linux will not touch this area, and we can extract it from ram once Linux is up and running.

Next is parsing the flat device-tree, this turns out to be easy as there is darwin source code that does this very thing early in the darwin boot process. The only tricky part is knowing the path which is "/chosen/boot-device-path". This gets me a device property, yea, now what to do it.

A little digging and this device property really the EFI device path so a fetch of the EFI EDK from https://edk.tianocore.org/ and a little poking around gets me the header files and some source code to use as an example for parsing the EFI device path for the boot disk. And presto, now I know the boot disk. Thanks to "David Elliott" (http://tgwbd.org/darwin/index.html) who suggested this method and no thanks to the Mac hackers out there who ignored my requests (why do I even ask). I'll be posting a [wiki page](DT_Decode.md) and [download of this sub-project](http://atv-bootloader.googlecode.com/files/DT_decode.zip) as it's a good example of how to parse a flat device-tree and how to parse an EFI device path.

# May 8, 2008 #
It's always bothered me that I could not find AppleSMC. There is a Linux kernel module for it but when I tried, applesmc reported no device nodes. I even found some code that could run under the AppleTV OS and dump device nodes -- nothing. WTF, how can that be. Apple hardware always uses AppleSMC.

Well, found it. It's actually easy to see from an ioreg dump once you know what to look for. I found it looking into front panel LED control. The AppleTV has a Cypress USB controller than handles the IR receiver. This is similar to other Apple hardware such as the MacMini. What was strange about the AppleTV USB controller is that there are more HID descriptors than required for just IR operation and there's a second interface with an interrupt endpoint. All other Apple hardware with internal USB IR receivers only have one interrupt endpoint.

That got the wheels turning and I took another look at an ioreg dump taken under the AppleTV OS. Well duh, there is is, using a USB interface and attached to the second interface. Typically, Apple uses AppleSMC to control led, fans and other accessory hardware device and this information shows that the internal USB controller IS handling these functions. Now to figure out how to talk to it without bricking my AppleTV. [Click this link to see information about the AppleTV internal USB controller](http://code.google.com/p/atv-bootloader/wiki/ATV_IR_Device).
# May 5, 2008 #
Ubuntu/Muthbuntu LiveCD boot from extracted iso fixed. Don't forget to copy the hidden ".disk" directory. Very important. Duh.

LiveXBMC working with mceusb IR remote. And speaking of IR, bikedude880 of the OSx86/awkwardtv fame suggested looking at bhollands "irkeyboardemu" for information on the IR controller. Thanks bikedude880, busy decoding the "Report Descriptor" as I speak. There seems to be more than what's required to describe a six button remote control. We will see where this goes.

DNS failing is solved, Grrr -- missing libs. That means a new release is pending to fix that and the missing fat/vfat support. And just when the download count has almost cleared 300 or 560 total for all versions.

# May 4, 2008 #
Ubuntu/Mythbuntu LiveCD booted direct from iso is being cranky. A USB cdrom based boot/install works fine but trying to boot from the iso or contents of the iso on a USB flash drive does not. It boots, but cannot seem to find it's root file system and drops to the initramfs prompt. Time to dig into the source code as Ubuntu specific kernel command-line params are not documented very well.

Someone posted info about using a JP1 IR remote with the AppleTV, has 30 functions (buttons) defined and **get this** works using the Apple IR receiver. Now that's neat, have to look around for an IR remote with a JP1 interface that suits my needs. remote rant on -- why do they always put the number keys low -- I use the number keys and they should be higher where you don't have to contort your fingers to use them -- remote rant off.

# May 3, 2008 #
Well that did not take long. I'm sold on the Microsoft MCE IR Remote. It's a pretty inexpensive solution and most all Linux apps already have support. A no-brainer if you ask me. Too bad one can't get raw codes from the Apple IR Controller but that's life, sometimes you win, sometimes not. I'll keep the Apple IR Controller in mind when I dip into "osputil". You never know, there might be possibility. Until then, just get the Microsoft MCE IR Remote and move on.

# May 2, 2008 #
Ok, I've finally gotten tired of the Apple IR remote. Need more buttons and I have a love/hate relationship with my Harmony 880 (that's another story). My [local Intex](http://www.intrex.com/) has the Microsoft A9O-00007 IR Remote with IR receiver for $27.99 and I'm on the way out to get one.

# May 1, 2008 #
Nice results with Mythbuntu Hardy installed to a USB flash drive. Little slow during initial MythTV setup. Hardy does not need any kernel patches as we can enabled the audio and IR devices using a different method. And Hardy  has 169.xx series nvidia binary driver.

Even with swap on the flash drive (yes, I know about the issues), it's surprisingly responsive. I think this configuration will be a keeper as I tend to trash the internal drive testing distro installs and AppleTV dev work.

# April 30, 2008 #
Grrr, seems to be a recent theme of users not getting boot when creating a USB flash drive with atv-bootloader. Don't panic, this method does work and numerous people have used it to boot Linux on the AppleTV. Something basic is incorrect and I've missed something in the massive editing of the wiki guides. This is quite annoying as web hits have really spiked in the last few days. I'll jump right on this once I get home from work tonight with my new 8GB fast flash drive (Corsair Voyager, sustained read speed of 19MB/sec, sustained write speed of 13MB/sec).

UPDATE - Clarified the wiki guides to solve some confusion.

On another note, Hardy has hfs tools via apt-get (http://packages.ubuntu.com/hardy/otherosfs/hfsprogs)

# April 29, 2008 #
Oh, the horror, Apple has purged ALL previous updates from mesa.apple.com. No more restore to 1.0.x unless you already have the updates squirreled away.

Some MythDora 5 action tonight. 1.2GB DVD download, ouch. "video=vesafb" works in kernel prams to get console framebuffer working. 169.12 nvidia driver version. Was not too hard to get X11 up with nvidia under-clocked. Of course, usbhid is a kernel built-in -- no module options tricks possible and ALSA need audio patch. MythDora 5 will need a kernel/ALSA rebuild to get IR/analog audio support working. I'm punting to someone experienced with building kernels under fedora, I need to move on and get back to atv-bootloader dev. Any takers?

I have an interesting idea to obtain the USB protocol that "osputil" uses in talking to the USB controller for fan and front panel LED control. Someone did a Qemu running OSX under Linux. If I can get that working -- Qemu running AppleTV OS under Linux on AppleTV hardware -- then I can use Linux USB snooping tools to trap the USB transfers. That beats trying to code/debug a custom "IOUSBDevice" under the AppleTV OS to snoop USB transfers.

# April 27, 2008 #
WooHoo, Ubuntu/Mythbuntu Gutsy AND Hardy boot from cdrom. Added new wiki page about cdrom booting. I really wish they would add vesafb to their initramfs.

# April 26, 2008 #
Updates abound.
  1. New version of "recovery.tar.gz" (aka atv-bootloader). Adds "mkfs.msdos", "mkfs.ext3", "mkfs.hfsplus", "gptsync" and "partprobe" to the initramfs. This completes the command-line tools required to build AppleTV and Linux partitions using only atv-bootloader.
  1. Slight mods to the internal scripts based on freeback from users.
  1. New version of dmg2img, this fixes the segmentation fault error.
  1. New backup/restore wiki page. You can use atv-bootloader to completely backup and restore the original contents of the internal PATA disk. This even has an "opps" section that can restore to factory even if you did not create a backup.

# April 23, 2008 #
Google's maintenance of the wiki pages is starting to get on my nerves. But I'm now editing via svn. Must be some serious maintenance going on. New page on [installing ndiswrapper for wireless](InstallWireless.md). It's pretty simple.

# April 17, 2008 #
Grrrr, last night was a bust, blocked at every turn. I was trying the get KnoppMyth to install onto a disk with GPT format. I think there is a way to shortcut the installer so I'm not forced to use cfdisk to partition, but I needed to build parted and hfs support. Of course, there was no room on the ramdisk that the installer uses. So tonight's exercise will be re-installing KnoppMyth the original way, building the apps I need, slipping them onto the atv-bootloader pen drive on a ext2 partition. Then try the shortcut again but this time I will have the partitioning support rebuilt and I should be able to slip them into the ramdisk filesytem. In theory -). More later tonight when I get home.

Think I have it. It's tricky but you have to do the KnoppMyth install using normal partitioning, but after getting the six choice dialog, atl-F2 into a console and re-partition the disk GPT with first three partitions like KnoppMyth wants and a fourth partition being "Recovery". AppleTV efi firmware does not care which partition is "Recovery" so make it the last one so KnoppMyth does not get confused.  You have to do it this way because the KnoppMyth install script does not understand GPT partitions and the only choice is to partition. Once you partition, then you get the six choice dialog.

Also KnoppMyth does not seem to pay attention to loading module options so we have to force our options. Edit "/etc/rc.local" and add
```
modprobe -r usbhid
modprobe usbhid quirks=0x05ac:0x8241:0x10
modprobe -r nvidia
modprobe nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222" NVreg_Mobile=0
```

This unloads usbhid and nvidia modules, then loads them with a) usbhid quirk to enable the AppleTV IR controller and b) nvidia with mobile and powermiser disabled so we can underclock the gpu. Notice that with a) now we don't have to patch and recompile the kernel, yea. This change will be propagated to the other distro installs once I get a chance to verify.

# April 15, 2008 #
Blog created, Doing real work right now so no updates until I get home later.

8:14pm - home again, now it's play time. 1st thing on the list is patch generation for IR and audio for an unreleased distro. Nothing new, same old kernel patches. For those unfamiliar with the process, you take the existing patch and try to apply using dry-run. If it works you're done. If not, then you need to figure out what's wrong and manually fix the patch. The target kernel is question is version 2.6.23. Not too old but the typical problem of the kernel not knowing about the AppleTV device IDs. The patches I'm generating here are going to be pushed upstream to the devs that handle the distro so they can include it in their next release.

The IR patch is dirt simple, This fixes "usbhid.ko" with four lines of code to add to "drivers/hid/usbhid/hid-quirks.c". The current patch applies without problem so done here.

Audio is also simple, this fixes "snd-hda-intel.ko" and gets applied to "sound/pci/hda/patch\_realtek.c". This patch is redundant as this distro uses ALSA for audio support and ALSA builds it's own "snd-hda-intel.ko" which will overwrite the kernel version.  So we will also have to fix "snd-hda-intel.ko" in ALSA which is at "alsa-kernel/pci/hda/patch\_realtek.c". It might seem silly to patch the kernel version but I do this just in case ALSA somehow did not get installed. The ALSA fix is more complicated as the current patch does not apply properly as the ALSA version of "patch\_realtek.c" is newer than that in 2.6.23 and it's internal structure has changed. Life in the patch/update lane.

We are patching alsa-driver-hg20080219 and we just need to test that the AppleTV audio device is detected and both analog and digital audio work properly. So just rebuild the driver, I'm not sure what configure parms the distro uses but we need "--with-cards=hda-intel".
```
cd alsa-driver-hg20080219 
./configure --with-cards=hda-intel 
make 
make install 
./snddevices 
```

To save the reboot, manually unload the old module, load the new one and check dmesg to see if the AppleTV audio device is detected.
```
root#modprobe -r snd-hda-intel
root#modprobe -v snd-hda-intel
root#dmesg | grep hda_codec
hda_codec: Detected AppleTV RealTek Subsystem ID
```

Bingo, there it is. Run "alsamixer and un-mute the digital output and I should hear audio over both analog and digital output. Note that digital audio over HDMI does not work yet but it is an ongoing project.

Once I test everything, and am confident that all is good, I generate diifs from the original code and the patched code,  then ship them upstream to the distro devs.

12:05am - done patching/building/testing and everything looks good so off the patches go swimming their way upstream. Memo to self, need to pull the dev source for ALSA and the kernel, create patches and submit them so they get into mainline source code.