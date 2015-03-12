# Update the boot disk MBR with gptsync #

# Introduction #

The installation of GRUB will also install a grub MBR (master boot record). The AppleTV does not use this MBR but the MBR has to be one that is compatible with GPT formatted partitions. The AppleTV EFI firmware and "boot.efi" are picky about the master boot record (MBR). While the AppleTV will eventually boot atv-bootloader with a "wrong" flavor MBR, it might take 30-60 seconds before it tries. We fix this by syncing the MBR with the GPT created formats.

# Details #

In order to sync the MBR with the gpt partitions we need to use "gptsync". Of course, there current apt-get version does not understand recovery partitions so we need to build the updated version.

```
# get some building tools
sudo apt-get install build-essential

# get the current source for "gptsync" which is in "refit" source
wget http://superb-west.dl.sourceforge.net/sourceforge/refit/refit-src-0.11.tar.gz
tar -xzf refit-src-0.11.tar.gz 

# make just gptsync
cd refit-src-0.11/gptsync
sudo make -f Makefile.unix
sudo cp gptsync /usr/bin/
sudo ldconfig

# sync the MBR to GPT partition for "/dev/sda"
sudo gptsync /dev/sda
```

The depreciated method to fix the slow boot time is to replace the grub MBR with a safe one
```
#------------------------------------------------
# fix up the MBR to make efi firmware happy and get quick boots
# remember to pick the correct device selection with dd.
wget http://atv-bootloader.googlecode.com/files/mbr_fast-1.0.bin
sudo dd if=mbr_fast-1.0.bin of=/dev/sda bs=512 count=1
```