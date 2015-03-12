# Mythbuntu Hardy Post-Install Fixes #

Mythbuntu Hardy is pretty easy to fix up.


---

# NVIDIA #
Mythbuntu Hardy uses the 169.12 nvidia binary kernel (selected during the install). This mean we don't have to update it, just add the under-clock. The nvidia gpu needs to be under-clocked to prevent nvidia xvmc related hangs/video corruption when doing xvmc assisted mpeg2 decode/display.

This enables gpu clock changes. Note that it's "nvidia\_new" not "nvidia" as we are running under the linux restricted modules install of the nvidia driver. Edit "/etc/modprobe.d/options" and add following which forces the nvidia driver to treat the chipset as non-mobile and disabled "PowerMister" which provide dynamic clock changes depending on gpu load.
```
options nvidia_new NVreg_RegistryDwords="PerfLevelSrc=0x2222" NVreg_Mobile=0
```

Create an autostart entry for X11. This is used to automatically apply our clock changes when X11 starts running. Clock changes using "nvidia-settings" are not saved and restored on restart so we have to do this manually. Ubuntu and Mythbuntu uses "/home/username/.config/autostart/" as the path to autostart apps. If you are running a different debian distribution then you will need to figure out where to place these additions.
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
```

Finally
```
# add nvidia-setting from apt-get
sudo apt-get install nvidia-settings

# run the nvidia xconfig tool to enable coolbits so we can actually change the nvidia gpu clock
sudo nvidia-xconfig --cool-bits=1 --no-composite --no-logo
```

if "nvidia-xconfig" complains about "cool-bits" then just do "sudo nvidia-xconfig --no-composite --no-logo" and add this option manually to 'Section "Screen"' in "/etc/X11/xorg.conf"
```
    Option         "Coolbits" "1"
```

if "nvidia-xconfig" complains about "--no-composite", manually add that to "/etc/X11/xorg.conf"
```
Section "Extensions"
    Option         "Composite" "Disable"
EndSection
```

To test, stop X11, unload/load the nvidia module then restart X11 and check the gpu clock settings. Note this might be "nvidia-new" instead of "nvidia".
```
# stop X11
sudo /etc/init.d/gdm stop

sudo modprobe -rv nvidia
sudo modprobe -v nvidia
```

You should see the following
```
insmod /lib/modules/2.6.24-16-generic/kernel/drivers/i2c/i2c-core.ko 
install /sbin/lrm-video nvidia 
insmod /lib/modules/2.6.24-16-generic/volatile/nvidia_new.ko NVreg_RegistryDwords="PerfLevelSrc=0x2222" NVreg_Mobile=0
```

Start X11 then check the gpu clock settings
```
# start X11
sudo /etc/init.d/gdm start

sudo nvidia-settings -q all
```



---

# AUDIO #
Dirt simple, edit "/etc/modprobe.d/options" and add
```
options snd-hda-intel model=imac24
```

That's it, remember to run "alsamixer" and un-mute for analog and digital audio output. The iMac with 24 inch LCD has very similar ALC885 ALSA configuration and we can use it for the AppleTV.


---

# IR Controller #
Edit "/etc/modprobe.d/options" and add
```
options usbhid quirks=0x05ac:0x8241:0x10
```

Rebuild the initramfs so that the usbhid module options get propagated to the initramfs
```
sudo update-initramfs -u
```

Now follow the "Setup LIRC" section (half way down the page) [in the old IR Update wiki page](UpdateIRDriver.md). Skip the "Enable usbhid" part as it was already enabled by the modprobe quirk above.





