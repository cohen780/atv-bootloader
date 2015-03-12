Instructions on how to get a copy of the AppleTV boot.efi file

# Introduction #

During power-on boot, AppleTV EFI firmware finds and executes the file "boot.efi" on a "Recovery" or "OSBoot" partition. This "boot.efi" file is responsible for loading the mach kernel and kexc drivers using "com.apple.Boot.plist". It must be an original AppleTV "boot.efi"and not one from an Intel Mac as it is signed and efi firmware checks the certificate and will rejected a "bad" "boot.efi" file. So for atv-bootlaoder to work, you need a copy of "boot.efi". This raises the chicken-or-egg question of how do you get "boot.efi" without opening the AppleTV and removing the hard drive. If you have already enabled "ssh" using a "patchstick" you can just extract it directly from your AppleTV using "scp" using something like this
```
scp frontrow@YOUR.IP.ADDRESS.HERE:/System/Library/CoreServices/boot.efi ./
```

If you don't have ssh enabled and don't want to open up the AppleTV, the solution is simple, extract it from the AppleTV update. The following instructions detail how to download the AppleTV 2.0.2 update and extract "boot.efi" using "dmg2img". "boot.efi" is the only file required to enable atv-bootloader to boot linux so unless you are interested in building a [patchstick](BuildingPatchstick.md), you can delete the downloaded/converted files.

**This needs to be done using an installed/working Linux distro and not a LiveCD. Some LiveCD distros use unionfs and loopfs does not work under a unionfs mount.**

**Update - April 29, 2008 - Apple seems to have purged their previous updates. The following are now missing from mesu.apple.com**.
```
1.0.1 - at http://mesu.apple.com/data/OS/061-2988.20070620.bHy75/2Z694-5248-45.dmg 
2.0.0 - at http://mesu.apple.com/data/OS/061-3561.20080212.ScoH6/2Z694-5274-109.dmg
2.0.1 - at http://mesu.apple.com/data/OS/061-4375.20080328.gt5er/2Z694-5387-25.dmg
```

Still present is the current 2.0.2 update.
```
2.0.2 - at http://mesu.apple.com/data/OS/061-4632.2080414.gt5rW/2Z694-5428-3.dmg
```

# Details #

**download the update and extract boot.efi
```
# install some required tools to build dmg2img
#
sudo apt-get install build-essential zlib1g-dev

# download and built dmg2img
wget http://atv-bootloader.googlecode.com/files/atv-dmg2img-1.0.tar.gz
tar -xzf atv-dmg2img-1.0.tar.gz
cd dmg2img
sudo ./install_dmg2img.sh

# download the AppleTV 2.2 update
wget http://mesu.apple.com/data/OS/061-4632.2080414.gt5rW/2Z694-5428-3.dmg
#
# convert it to an img format
dmg2img 2Z694-5428-3.dmg atv.img

# create a mount point
mkdir atv-update

# mount the converted img
sudo mount -o loop -t hfsplus atv.img atv-update

# extract boot.efi
sudo cp -ap atv-update/System/Library/CoreServices/boot.efi ./

# check that boot.efi byte count is the same.
# the time stamp might be different and that does not matter
# It is the permissions and byte count that are important.
ls -l boot.efi
# Mine reports "-rw-r--r-- 1 root    root       298800 2007-06-19 00:47 boot.efi"

# and the md5 checksum should match below.
md5sum boot.efi
280323d8700e4cfef15116f7e50590e3  boot.efi
```**
