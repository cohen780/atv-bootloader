# Update the audio driver #

# Introduction #

Patch the Realtek audio module to correctly identify the AppleTV audio hardware and enable analog (rca) output. Audio over HDMI is not supported at the current time.

# Details #

This involves patching the existing linux-ubuntu-modules-2.6.22-2.6.22 install, so remember that if you allows this to be updated in the future, you will have to reapply this patch. Thanks to Jason (SredniV in the [MythBuntu forums](http://ubuntuforums.org/showthread.php?t=720676&page=3)) for figuring out the deb part.
```
# get some build tools
sudo apt-get install fakeroot kernel-wedge

# move to the standard location for kernel source
cd /usr/src
# get the source code for the current ubuntu modules
sudo apt-get source linux-ubuntu-modules-2.6.22-14-generic
cd /usr/src/linux-ubuntu-modules-2.6.22-2.6.22/ubuntu/media/snd-hda-intel

# patch patch_realtek.c
# download the patch
sudo wget http://atv-bootloader.googlecode.com/files/atv-realtek-rca-audio-r2-2.6.22-14-generic.patch
# first, check the patch for errors
sudo patch --dry-run < atv-realtek-rca-audio-r2-2.6.22-14-generic.patch
# if no errors then apply the patch
sudo patch < atv-realtek-rca-audio-r2-2.6.22-14-generic.patch

# back to the level of the module deb source
cd ../../..

# rebuilt the modules
sudo fakeroot debian/rules binary-arch arch=i386 flavours="generic"

# the build will end with the following error, that's ok
dpkg-deb: --extract needs a target directory.
Perhaps you should be using dpkg --install ?
make: *** [binary-udebs] Error 2

# up one more level to where the deb is placed 
cd ..
sudo dpkg --install linux-ubuntu-modules-2.6.22-14-generic_2.6.22-14.37_i386.deb

# dpkg --install should rebuild the initramfs but some 
# users seem to require a manual rebuild of the initramfs
sudo update-initramfs -u

# the change takes effect on the next reboot and you should see
dmesg | grep hda_codec
hda_codec: Detected AppleTV RealTek Subsystem ID

```