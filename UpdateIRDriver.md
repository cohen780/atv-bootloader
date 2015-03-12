# Update the IR driver #

The Apple IR controller is a USB device similar to that of the MacMini but the device ID is different. So to enable IR device support we have to patch and rebuild a kernel module. Apple IR remote can actually generate any of 256 6 button sets. The IR receiver on the AppleTV is physcially able to recognize and distingusih all 6x256 codes, and no others.

The sequences are all of the form 0x87EEXXYY, where XX is the ID code (UID) and YY is the key code. This info comes from the [JP1 remote project](http://www.hifi-remote.com/forums/viewtopic.php?p=59792#59792) The UID is used for paring IR remotes to IR controllers under OS X. When you pare a IR remote to the computer what you are doing is telling OS X to only listen to an Apple IR remote with this UID. A certain key press on the Apple IR remote will increment the UID. For example, the "menu" button on my remote has this for the IR code
```
0x87EE4A03
```

The pre-data is "0x87EE", the UID is "0x4A" and the key is "0x03". If I increment the UID code, now I get this for the "menu" button
```
0x87EE4B03
```

There are 256 possible UID codes. So 256 x 6 possible combinations that the AppleTV IR controller (actually all Apple IR receivers) will decode. This means that you can use a learning remote instead of the Apple IR remote to gain access to more than six buttons. To do this, use the learning remote to learn six buttons from the Apple IR remote. Then increment the UID by pressing "Menu" and "Play/Pause", then learn another set of six buttons. rinse, repeat until you have enough buttons defined.

Now use irrecord to create your lirc.conf from the learning remote. Then you have the AppleTV IR controller responding to the learning remote which has more than six buttons.


---

For a raw analysis of the [Apple IR remote protocol](http://www.barchard.net/files/apple_ir_remote_analysis.html). And [web page](http://cweiske.de/tagebuch/Getting%20Apple%20Remote,%20Macbook%20and%20LIRC%20work%20together.htm) that mentions handling the repeat in a different method (I do not think this is relevant anymore due to the lirc repeat patch).


---

There are other possibilities than using the AppleTV IR remote. As mentioned above, the AppleTV IR receiver is only capable of receiving IR sequences from an Apple IR remote. The IR receiver pre-processes the raw IR data and ignores any other. That leaves three methods for gaining more IR commands. a) use a learning remote and exploit the mutiple UID (as mentioned previously), b) replace the IR controller with a USB based IR receiver or c) use a programmable remote to emulate an Apple IR remote with multiple UIDs. This link describes [using other IR remotes with the AppleTV](Other_IR_Remotes.md).


---

Contributed by Ben Firshman, [here's an alternate](http://atv-bootloader.googlecode.com/files/appleremote.py-0.1.tar.gz) to using "~/.mythtv/lircrc". This is a python script that enables greater functionality using the Apple IR Remote. It completely replaces using lircrc for control of a MythFrontEnd so make sure "~/.mythtv/lircrc" is removed before installing. Thanks Ben.


---

# Enable usbhid #

There are two methods for fixing this, an older method which involves patching appleir.c and the preferred method which involves patching usbhid.c. While patching appleir.c is easier, I choose the preferred method as it moves IR control out of the kernel and into userspace. This update is in two parts, first patch/rebuild "drivers/hid/usbhid/usbhid.c", then patch/rebuild lirc. Just to remind users of lirc, lircd.conf assigns symbols to the signals from the remote and ~/.lircrc associates those symbols to actions in programs.
```
Note: there is way to alter usbhid without rebuilding the module but it only works with 2.6.22 and above kernel versions.
This has the form;

modprobe usbhid quirks=0x<vendor_id>:0x<product_id>:<quirk value>

So for the Apple IR receiver IDs and HID_QUIRK_HIDDEV it would be;

options usbhid quirks=0x05AC:0x8241:0x10

This would go into modprobe.conf or modules.d/usbhid, or modprobe.d/usbhid then
do a "modules-update".

another method is to add this to the kernel boot params

usbhid.quirks=0x05AC:0x8241:0x10
```

Patch/rebuild usbhid.ko. The IR controller is a USB HID device but the usbhid kernel module does not know about the AppleTV IR device, so we need to add the device ID. We are also going to shortcut the normal dpkg build as it takes forever to build and just build what we need to make a new usbhid.ko. Then we copy this into "/lib/modules/2.6.22-14-generic/kernel/drivers/hid/usbhid/" and rebuild the initramfs.
```
sudo apt-get install  kernel-package libncurses-dev ncurses-dev  unzip
sudo apt-get source linux-image-2.6.22-14-generic

cd /usr/src/linux-source-2.6.22-2.6.22/drivers/hid/usbhid/

sudo wget http://atv-bootloader.googlecode.com/files/atv-hid-quirks-enable-ir-r2-2.6.22-14-generic.patch

# test the patch (this should complete without errors)
sudo patch --dry-run < atv-hid-quirks-enable-ir-r2-2.6.22-14-generic.patch

# apply the patch
sudo patch  < atv-hid-quirks-enable-ir-r2-2.6.22-14-generic.patch

# cd back to the root of linux source
cd ../../../

# copy the current kernel config into the kernel source tree
sudo cp /boot/config-2.6.22-14-generic ./config
sudo make oldconfig
# this will take some time
sudo make modules

# copy the rebuild usbhid module into runtime location
sudo cp drivers/hid/usbhid/usbhid.ko /lib/modules/2.6.22-14-generic/kernel/drivers/hid/usbhid/usbhid.ko

# rebuild the initramfs
sudo update-initramfs -u

reboot

# /dev/usb/hiddev0 should now be present
```


---

# Setup LIRC #

Patch/rebuild lirc. lirc versions <= 0.8.2 do not flag repeat events for an apple hiddev device. This part will fix this and rebuild lirc.
```
# install lirc package and choose Apple Mac Mini if asked
sudo apt-get install lirc

# get the source and source dependencies
sudo apt-get build-dep lirc
apt-get source lirc
# get the lirc patch
wget http://atv-bootloader.googlecode.com/files/lirc-0.8.2-macmini-repeat.patch

cd lirc-0.8.2
# test the patch
patch --dry-run -p1 < ../lirc-0.8.2-macmini-repeat.patch
# apply the patch
patch -p1 < ../lirc-0.8.2-macmini-repeat.patch

# rebuild the lirc package
sudo dpkg-buildpackage

# install the package and choose Apple Mac Mini if asked
sudo dpkg -i ../lirc*deb
```

Create the lirc files to support the Apple IR controller. This is going to be highly dependent on install distribution so you might have to do some google search to find the correct conf. This procedure outlines creating the lirc files from scratch. Remember lircd.conf assigns symbols to the signals from the remote, and ~/.lircrc associates those symbols to actions in programs
```
sudo /etc/init.d/lirc stop

sudo nano /etc/lirc/hardware.conf
#replace DEVICE="" with
#DEVICE="/dev/usb/hiddev0"
```

You can generate an /etc/lirc/lircd.conf from scratch
```
#use PLUS, MINUS, PREV, NEXT, MENU and PLAY for the button
#names for the Apple remote and follow the instructions from irrecord
sudo irrecord -H macmini -d /dev/usb/hiddev0 /etc/lirc/lircd.conf
```

Or use this one, but watch out, the AppleIR "pre\_data" is different when the remote is paired with the AppleTV so you might have to change "pre\_data" to match your Apple IR remote.
```
begin remote
  name  Apple_IR
  bits            8
  eps            30
  aeps          100

  one             0     0
  zero            0     0
  pre_data_bits   24
  pre_data       0x87EE4A
  gap          211995

      begin codes
          PREV                     0x09
          NEXT                     0x06
          PLUS                     0x0A
          MINUS                    0x0C
          PLAY                     0x05
          MENU                     0x03
      end codes
end remote
```

Here's another variant
```
begin remote
 name  Apple_IR
 bits            8
 eps            30
 aeps          100

 one             0     0
 zero            0     0
 pre_data_bits   24
 pre_data       0x87EEA3
 gap          211995

     begin codes
         PLUS                     0x0B
         PREV                     0x08
         PLAY                     0x04
         NEXT                     0x07
         MINUS                    0x0D
         MENU                     0x02
     end codes
end remote
```

Append this to your ~/.mythtv/lircrc or use  [Ben's python script](http://atv-bootloader.googlecode.com/files/appleremote.py-0.1.tar.gz)
```
#nano ~/.mythtv/lircrc and append
begin
    remote = Apple_IR
    prog = mythtv
    button = PLUS
    config = Up
    repeat = 2
    delay = 0
end

begin
    remote = Apple_IR
    prog = mythtv
    button = MINUS
    config = Down
    repeat = 2
    delay = 0
end

begin
    remote = Apple_IR
    prog = mythtv
    button = MENU
    config = Escape
    repeat = 2
    delay = 0
end

begin
    remote = Apple_IR
    prog = mythtv
    button = PLAY
    config = Space
    repeat = 2
    delay = 0
end

begin
    remote = Apple_IR
    prog = mythtv
    button = NEXT
    config = Right
    repeat = 2
    delay = 0
end

begin
    remote = Apple_IR
    prog = mythtv
    button = PREV
    config = Left
    repeat = 2
    delay = 0
end
```

Restart lirc and test
```
# restart lirc
sudo /etc/init.d/lirc restart

# test your configuration by running the following app and pressing the IR buttons
irw /dev/lircd
```