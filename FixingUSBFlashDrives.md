# USB Flash Drive Games #

Sometimes a USB flash drive will be strange under Linux. The re-partitioning gives strange errors. It will not mount or there are "extra" partitions that you can't get rid of. Welcome to the wonderful world of Windows causing problems with basic devices. What you are seeing is either geometry differences or "special" partitions designed to aid Windows user. Once such mechanism is [U3 Smart Drive](http://www.u3.com/smart/default.aspx). These "features" will get in the way and prevent using the USB flash drive in a normal manor under Linux. They must be removed and unfortunatly this normally needs to be performed under Windows.

Other times, the flash geometry is just wrong or very odd-ball. USB flash drives are mass produced and can have different flash components even if the drives are identical in outside appearance.

Here is a guide to change the geometry to something more useful. We use a large amount of USB flash drives at work as the boot drive for an embedded Linux system and this is what we use to "fix" them.

The current USB "Super Flash Drives" are labeled 256MB but are showing 512MB, the real problem is that they have an odd-ball drive geometry together with an strange partition table layout and they don't boot with syslinux.

You can change the above to match your requirements. I use it as is, then re-partition again with parted. This guide might fail with very large flash drives. It would need to be changed from "USB-ZIP" geometry to "USB-HDD" as the zip geometry has a fixed max size.