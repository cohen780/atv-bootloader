# Update the Nvidia binary driver #

# Introduction #

There are two nvidia drivers used under Linux, the "nv" driver which comes from kernel sources and the "nvidia" driver which comes from nvidia. We need to use the "nvidia" driver to gain hardware decode acceleration support. The "nvidia" driver is not "open source" as it contains a binary blob of code. As such many Linux distributions place certain restrictions regarding the installation. Ubuntu (and Mythbuntu) resolve this by using a "restricted linux modules" package to control the installation. This seems to work fine unless the version of the nvidia driver one requires has not been updated or backported by the Ubuntu developers. This is the case here, so we need to do the update manually.

# Why #

The nvidia gpu needs to be under-clocked to prevent nvidia xvmc related hangs/video corruption when doing xvmc assisted mpeg2 decode/display. This simple step took me months to resolve. The 100.14.xx series driver does not allow gpu clock changes to 7300 chipsets so we need to update to the 169.xx series driver. These instructions assume that the 100.14.19 (default) is installed as it helps having a working xorg.conf.

# Details #

Ubuntu uses the "restricted linux modules" package to control the installation and startup of nvidia binary driver. The 169.xx series drivers are not available in the repositories for Gutsy nor it seems will they be backported so a simple at-get update is not possible. So we need to manually disable nvidia hardware from the "restricted linux modules" package and install the most recent version. Some people use [Envy](http://www.albertomilone.com/nvidia_scripts1.html) to automatically do this, I prefer the manual steps.

> Remove the installed nvidia driver and disable the use of the "restricted linux modules" package to control nvidia hardware.
```
#------------------------------------------------
# I do this from a remote ssh login, you really have to do this as you need
# to stop the X11 display driver or the nvidia install will not proceed.

# get build tools  
sudo apt-get install build-essential gcc gcc-3.4 xserver-xorg-dev
sudo apt-get install linux-headers-`uname -r`

# stop X11, remember do this from a ssh session.
sudo /etc/init.d/gdm stop

# remove and purge the current nvidia driver (100.14.19)
sudo apt-get --purge remove nvidia-glx nvidia-glx-new nvidia-settings nvidia-kernel-common

# edit linux-restricted-modules-common and change
# DISABLED_MODULES="" to DISABLED_MODULES="nv nvidia_new"
sudo nano /etc/default/linux-restricted-modules-common

# download the current nvidia binary
wget http://us.download.nvidia.com/XFree86/Linux-x86/169.12/NVIDIA-Linux-x86-169.12-pkg1.run

# run the nvidia installer, take the defaults (yes) and update the current xorg.conf (last step)
sudo sh NVIDIA-Linux-x86-169.12-pkg1.run

# run the nvidia xconfig tool again to enable coolbits so we can change the nvidia gpu clock
sudo nvidia-xconfig --cool-bits=1 --no-composite --no-logo

# restart X11
sudo /etc/init.d/gdm start

# run glxgears from a terminal window to check xvmc. I get about 2106.5 FPS
```

Now that a 169.xx series nvidia driver is installed, we need to under-clock it to prevent hangs/video corruption when doing xvmc decode. The AppleTV uses a 7300 mobile chipset, normally mobile chipsets are not allowed to have clock changes so we have to do some nvidia voodoo commands to alter this behavior and we also want to have this fix automatically applied every X11 starts running.

> Enabled gpu clock changes. The following forces the nvidia driver to treat the chipset as non-mobile and disabled "PowerMister" which provide dynamic clock changes depending on gpu load.
```
#------------------------------------------------
# Allow a nivida mobile chipset to be under-clocked by adding
# a module load option for the nvidia driver. 
sudo nano /etc/modprobe.d/options
# add
options nvidia NVreg_RegistryDwords="PerfLevelSrc=0x2222" NVreg_Mobile=0
```

> Create an autostart entry for X11. This is used to automatically apply our clock changes when X11 starts running. Clock changes using "nvidia-settings" are not saved and restored on restart so we have to do this manually. Ubuntu and Mythbuntu uses "/home/username/.config/autostart/" as the path to autostart apps. If you are running a different debian distribution then you will need to figure out where to place these additions.
```
# nvidia-settings is used to actually change the gpu clock but,
# we want it to always be applied after the startup of X11. Create an 
# autostart entry for the username that was setup during the install. 
# I'm using "username" here, you should replace "username" with your username.
#
sudo nano /home/username/.config/autostart/nvidia_fixes.desktop
[Desktop Entry]
Encoding=UTF-8
Version=0.9.4
Type=Application
Name=nvidia xvmc hang fixup
Comment=fixes nvidia problems with xvmc decode
Exec=/usr/sbin/nvidia_hang_fix.sh
StartupNotify=false
Terminal=false
Hidden=false

# now create the actual script
sudo nano /usr/sbin/nvidia_hang_fix.sh
#!/bin/bash
nvidia-settings -a GPUOverclockingState=1
nvidia-settings -a GPU2DClockFreqs=200,800

# change the file permission so it can execute
sudo chmod 755 /usr/sbin/nvidia_hang_fix.sh

# when X11 is up on the next reboot, you can verify the gpu clock change
# by doing the following in a terminal window and look for "GPU2DClockFreqs". 
# You should see 200 for gpu and 800 for vram and NOT 360 for gpu and 720 for vram.
#
sudo nvidia-settings -q all
```


# Problems #

There have been some reports of some remaining bits of the "restricted linux modules" package preventing the loading of the 169.xx nvidia binary driver. The fix was to edit "/etc/modprobe.d/lrm-video" and comment out the loading of the packaged nvidia module by changing the original
```
# Make nvidia/nvidia_legacy and fglrx use /sbin/lrm-video to load
install fglrx /sbin/lrm-video fglrx $CMDLINE_OPTS
install nvidia /sbin/lrm-video nvidia $CMDLINE_OPTS
install nvidia_legacy /sbin/lrm-video nvidia_legacy $CMDLINE_OPTS
install nvidia_new /sbin/lrm-video nvidia_new $CMDLINE_OPTS
```
to this
```
# Make nvidia/nvidia_legacy and fglrx use /sbin/lrm-video to load
install fglrx /sbin/lrm-video fglrx $CMDLINE_OPTS
#install nvidia /sbin/lrm-video nvidia $CMDLINE_OPTS
#install nvidia_legacy /sbin/lrm-video nvidia_legacy $CMDLINE_OPTS
#install nvidia_new /sbin/lrm-video nvidia_new $CMDLINE_OPTS
```