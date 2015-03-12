# Apple IR Remote with LIRC #

The Apple IR controller is a USB device similar to that of the MacMini but the USB device ID is different. So to enable IR device support we have to either force the usbhid kernel module to ignore this device or patch and rebuild the usbhid kernel module. A kernel version 2.6.22 or greater and use a kernel module option to ignore the AppleTV IR device. Kernel versions less than 2.6.22 will need to patch and rebuild the usbhid module.

Apple IR remote can actually generate any of 256 6 button sets. The IR receiver on the AppleTV is physcially able to recognize and distingusih all 6x256 codes, and no others.

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

For a raw analysis of the [Apple IR remote protocol](http://www.barchard.net/files/apple_ir_remote_analysis.html). And [web page](http://cweiske.de/tagebuch/Getting%20Apple%20Remote,%20Macbook%20and%20LIRC%20work%20together.htm) that mentions handling the repeat in a different method (I do not think this is relevant anymore due to the lirc repeat patch).

Contributed by Ben Firshman, [here's an alternate](http://atv-bootloader.googlecode.com/files/appleremote.py-0.1.tar.gz) to using "~/.mythtv/lircrc". This is a python script that enables greater functionality using the Apple IR Remote. It completely replaces using lircrc for control of a MythFrontEnd so make sure "~/.mythtv/lircrc" is removed before installing. Thanks Ben.


---

# Enable usbhid #
The Apple IR controller is a USB device that depends on the kernel module "usbhid.ko". This kernel module must be patched or have options added during load, see the specific Linux distribuntion install guides in this wiki for information about enabling "usbhid".


---

# Patch LIRC #
LIRC versions <= 0.8.2 do not flag repeat events for an apple hiddev device. This section will fix this and rebuild lirc.
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

# Setup LIRC #
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