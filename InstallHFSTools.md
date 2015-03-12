# Build/Install HFS Tools #

HFS (hfs) and HFSPlus (hfsplus) are the names of Apple formatted partitions. These are similar to EXT2 and EXT3 in the Linux world, just different flavor. The original AppleTV disk has four GPT partitions, three of which are formatted hfsplus. Two of those are actually hfsplus journaled.

In order to boot atv-bootloader, one needs to be able to create a GPT partition with a "Recovery" GUID and format that partition hfsplus. Linux does not have proper disk tools for formatting hfsplus so one needs to build and install them. These tools actually come from "darwin", the Apple open source project and they have an addition patch in oder to compile and function under a Linux OS.

**Note**
If you are running Ubuntu Hardy then this step is not required. Hardy has hfs tools via apt-get (http://packages.ubuntu.com/hardy/otherosfs/hfsprogs)
```
sudo apt-get install hfsprogs
```


**Warning**
If your file path name has spaces "/home/davilla/atv dev/work", then hfs support tools will not build properly. Make sure your file path does not have spaces in it.

**build and install hfs support tools
```
# fetch hfs_support
wget http://atv-bootloader.googlecode.com/files/hfs_support-1.0.tar.gz
tar -xzf hfs_support-1.0.tar.gz

# build and install
cd hfs_support/
sudo ./build_diskdev_cmds.sh
```**

Done.