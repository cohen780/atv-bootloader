# Partition for Bootstrap Linux Install #

This is the easiest partition method. There are no prep steps. You boot a [CD installer using atv-bootloader](BootingLiveCD.md), then install to the internal PATA disk over-writing everything on it. Make sure you [backup your AppleTV](ATVBackup.md) so you can return to the factory AppleTV OS if desired.

You will need to make sure "com.apple.Boot.plist" on the recovery partition of the USB flash disk has the the atv-boot parameter of "auto"
```
<string>atv-boot=auto video=vesafb</string>
```

When "auto" is selected, atv-bootloader will search for a GRUB menu.lst and kexec boot the default kernel.

# Usage #

When the AppleTV is powered-on, Apple EFI firmware will look for a "Recovery" partition. It will find the one on the USB flash drive and boot atv-bootloader which then searches for valid GRUB menu.lst boot settings. Atv-bootloader will in turn use these boot settings to kexec boot that default kernel.