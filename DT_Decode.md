# Finding the boot device #

I needed to figure out which interface atv-bootloarder was booting from, the internal PATA or an external USB. This way I could skip the 5 second delay for USB device detection if booting off the internal PATA disk and make internal PATA booting faster. The boot device can be found as a node on the flat device-tree that the AppleTV EFI firmware passes to a mach kernel.


---

First I need a copy of the flat device-tree so I can decode it outside the boot process. This turns out to be tricky since atv-bootloader has no file i/o capabilities before the embedded Linux kernel loads and after the embedded Linux kernel loads it's too late. So one must be sneaky and create a hole in the e820 memory map where the flat device tree is located. Now Linux will not touch this area, and we can extract it from ram once Linux is up and running. This required modifying atv-bootloader. Basically, I first added a printout of the size and ram location of the flat-device tree. This gave me;
```
dt_size = 0x13d8, dt_ptr=0x025ae000
```

"cat /proc/iomem" gives me the Linux memory organization and I use this to find the above ram location.
```
00000100-0008efff : System RAM
0008f000-0008ffff : ACPI Non-volatile Storage
00090000-0009ffff : System RAM
000a0000-000bffff : reserved
000f0000-000fffff : System ROM
00100000-025affff : System RAM
  00100000-0031b5a3 : Kernel code
  0031b5a4-00414dc3 : Kernel data
  00476000-004eba7f : Kernel bss
025b0000-02600fff : reserved
02601000-0f170fff : System RAM
0f171000-0f371fff : ACPI Non-volatile Storage
0f372000-0febcfff : System RAM
0febd000-0febefff : ACPI Non-volatile Storage
0febf000-0febffff : ACPI Tables
0fec0000-0feeefff : ACPI Non-volatile Storage
0feef000-0fef0fff : System RAM
0fef1000-0fefefff : ACPI Tables
0feff000-0fefffff : System RAM
0ff00000-0fffffff : reserved
10000000-1fffffff : PCI Bus #01
  10000000-1fffffff : 0000:01:00.0
    10028000-10737fff : vesafb
20000000-21ffffff : PCI Bus #01
  20000000-20ffffff : 0000:01:00.0
  21000000-21ffffff : 0000:01:00.0
    21000000-21ffffff : nvidia
22000000-220fffff : PCI Bus #02
  22000000-220fffff : 0000:02:00.0
22100000-221fffff : PCI Bus #03
  22100000-221000ff : 0000:03:03.0
    22100000-221000ff : 8139too
22200000-222fffff : PCI Bus #02
  22200000-22203fff : 0000:02:00.0
22300000-22303fff : 0000:00:1b.0
  22300000-22303fff : ICH HD audio
22304000-22304fff : 0000:00:07.0
22305000-223053ff : 0000:00:1d.7
  22305000-223053ff : ehci_hcd
f00f8000-f00f8fff : reserved
fed00000-fed003ff : HPET 0
  fed00000-fed003ff : pnp 00:04
fed14000-fed17fff : pnp 00:01
fed18000-fed18fff : pnp 00:01
fed19000-fed19fff : pnp 00:01
fed1c000-fed1ffff : reserved
fed20000-fed8ffff : pnp 00:01
fffa0000-fffcffff : reserved
```

The interesting parts are
```
00100000-025affff : System RAM
025b0000-02600fff : reserved
```

So System RAM overlaps the device-tree ram location which over-writes the values. Bummer, now I have to change the size of the System RAM memmap so that it does not overlap. This needs to be done in "linux\_code.c" where the EFI memmap is converted to an e820 memmap. I already have a routine that fixes an existing memory overlap case so I can just add what I need in the routine "quirk\_fixup\_efi\_memmap", here's a snipit.
```
for (i = 0, p = (efi_memory_desc_t*)bp->s.efi_mem_map; i < num_maps; i++) {
	UINT64   target;

	target = 0x025AE000;
	md = p;

	bgn = md->phys_addr;
	end = md->phys_addr + (md->num_pages << EFI_PAGE_SHIFT);

	if ( (bgn < target) & (end > target) ) {
		UINT64          new_bgn, new_end, new_pages;

		new_bgn = bgn;
		new_pages = (target - new_bgn) / (1 << EFI_PAGE_SHIFT);

		new_end = new_bgn + (new_pages << EFI_PAGE_SHIFT);
		printk("ATV:   fixing memory target\n");

		md->phys_addr = new_bgn;
		md->num_pages = new_pages;

		md->num_pages = new_pages;
	}

	p = NextEFIMemoryDescriptor(p, bp->s.efi_mem_desc_size);
}
```

This finds the EFI memmap entry that contains 0x025AE000 and adjusts that memmap entry to have 0x025AE000 as the new ending. Normally, I would not hardcode address location, especially with the AppleTV as they do seem to move around, but this was just a hack to get at the flat device tree.

A re-compile and copy the new "mach\_kernel" to a USB flash drive. Boot using "atv-boot=none" then telnet in and pull out the section using "dd".
```
dd if=/dev/mem of=atv_flat_device_tree.bin bs=1 skip=39510016 count=5080
```

Move that to the USB flash drive and there you go, the extracted flat device-tree from the AppleTV under Linux with the help of atv-bootloader. I could have done this under the AppleTV OS using ioreg but that would be cheating and not much fun.

---

Next is parsing the flat device-tree, this turns out to be easy as there is darwin source code (device\_tree.h and device\_tree.c) that does this very thing early in the darwin boot process. The only tricky part is knowing the path which is "/chosen/boot-device-path".
```
// device-tree init
DTInit(flat_device_tree);

// Get the "boot-device-path" property from the device tree.
if (DTLookupEntry(0, "/chosen", &entry) == kSuccess) {
	if (DTGetProperty(entry, "boot-device-path", (void **)&value, &size) == kSuccess) {
	}
}
```

Wow, that was easy, This gets me a device property, now what to do it.


---

A little digging and some help reveals that this device property really the EFI device path so a fetch of the EFI EDK from https://edk.tianocore.org/ and a little poking around gets me the header files and some source code to use as an example for parsing the EFI device path for the boot disk. And presto, now I know the boot disk. Here is a manually decoded "boot-device-path" property from a boot using a USB flash drive connected to the AppleTV using a USB hub.
```
Type 0x01 - Hardware Device Path 
Type 0x02 - ACPI Device Path 
Type 0x03 - Messaging Device Path 
Type 0x04 - Media Device Path 
Type 0x05 - BIOS Boot Specification Device Path 
Type 0xFF - End of Hardware Device Path 


0x02	Type ACPI Device Path
0x01	Sub type ACPI Device Path
0x0C	LSB Length
0x00	MSB Length
0xD0	_HID PNP0A03 (0x41D0 == "PNP")
0x41
0x03
0x0A
0x00	_UID
0x00
0x00
0x00

0x01	Type Hardware Device Path
0x01	Sub type PCI Device Path
0x06	LSB Length
0x00	MSB Length
0x00	PCI Function
0x1D	PCI Device

0x03	Type Messaging Device Path
0x05	Sub type USB Device Path
0x06	LSB Length
0x00	MSB Length
0x00	ParentPortNumber
0x00	InterfaceNumber

0x03	Type Messaging Device Path
0x05	Sub type USB Device Path
0x06	LSB Length
0x00	MSB Length
0x03	ParentPortNumber
0x00	InterfaceNumber

0x03	Type Messaging Device Path
0x05	Sub type USB Device Path
0x06	LSB Length
0x00	MSB Length
0x00	ParentPortNumber
0x00	InterfaceNumber

0x04	Type Media Device Path
0x01	Sub type Media Hard Drive Device Path
0x2A	LSB Length (42)
0x00	MSB Length
0x01	PartitionNumber
0x00
0x00
0x00
0x28	PartitionStart
0x00
0x00
0x00
0x00
0x00
0x00
0x00
0x00	PartitionSize
0x10
0x01
0x00
0x00
0x00
0x00
0x00
0x50	Signature[16]
0xAB
0xA4
0x1D
0xA6
0xDD
0xBD
0x4D
0xA4
0x90
0xF1
0xA4
0xD4
0xE6
0x76
0x5F
0x02	MBR_TYPE_EFI_PARTITION_TABLE_HEADER
0x02	SIGNATURE_TYPE_GUID

0x7F	END_DEVICE_PATH_TYPE
0xFF	END_ENTIRE_DEVICE_PATH_SUBTYPE
0x04
0x00
```


---

A big thanks to "David Elliott" (http://tgwbd.org/darwin/index.html) who suggested this method and no thanks to the Mac hackers out there who ignored my requests (why do I even ask?).

This sub-project turned out to be very interesting as source code examples for parsing darwin flat device-trees and the resulting EFI device paths are next to non-existing on the web. The apps that do this are closed source OS X hacker tools and guess they don't want their secret sauce discovered. So [grab the Xcode project](http://atv-bootloader.googlecode.com/files/DT_decode.zip) if you want to play around.

The extracted flat device tree is included as a binary file. Run it under the debugger and step through to see it work. Free free to use this as you see fit. The only Licensing is Apple Public Source License Version 2.0 on the two files from darwin source and BSD Licensing on the EFI EDK files included for reference. If you pick an older xnu source you might find Apple Public Source License Version 1.0 on the darwin files. Since I'm not hacking an Apple Operating System, the Version 2.0 License is fine for my usage.




