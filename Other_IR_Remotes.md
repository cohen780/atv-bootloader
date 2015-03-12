# Using other IR Remotes #





---

# Microsoft MCE IR Remote #
The Microsoft MCE IR remote include a USB receiver/transmitter. It is used to completely replace the AppleTV IR receiver. Since it is a USB device, that means if you are already using the single USB port for an external USB disk you will need to deploy a USB Hub for more USB ports. The Microsoft MCE IR remote is well supported under LIRC and as there are already plenty of web references, I'm not going to repeat them here.


---

# JP1 IR Remotes #
The JP1 IR remotes are made by Universal Electronics (UEIC), which include the brand names "One For All" and "Radio Shack". A good starting place to get information about JP1 IR remtoes is http://www.hifi-remote.com/.

The JP1 Interface enables you to perform all sorts of customizations to many Radio Shack and One For All brand universal remotes, that are not possible just using the remote itself, such as adding new device codes, re-programming existing device codes. This mean that you can define more AppleTV IR controller compatible IR sequences (buttons) than are emitted by the six button Apple TR remote.

Contributed by vvadim -- He has managed to get 30 functions made using a single UID and programmed them into his JP1 capable remote (URC-8811). He now controls his appletv (running mythtv) with this remote and says it's perfect, no extra dongles, everything just there. plug in the power and video and everything is there. Grab the [JP1 files in the download section](http://atv-bootloader.googlecode.com/files/JP1-ATV-Remote-v1.zip).