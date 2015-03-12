# AppleTV Internal USB Controller Info #

The AppleTV IR receiver appears as a USB-based human interface device (HID). Similar to a USB mouse or keyboard it uses an interrupt IN endpoint to send information to the host computer when some action occurs such as pressing a button on the Apple IR remote.

USB HID devices have the following requirements
  * A HID must have an interrupt IN endpoint for sending periodic data to the host. An interrupt OUT endpoint for receiving periodic data from the host is optional
  * A HID must contain a class descriptor and one or more report descriptors
  * A HID must support the HID-specific control request Get\_Report and may support the optional request Set\_Report

For interrupt IN transfers, the device must place the report data in the interrupt endpoint's buffer and enable the endpoint.

---

The AppleTV IR HID device is defined by this report descriptor. This is taken from "irkeyboardemu" (bholland/bikedude880) and corrected. The descriptor in "irkeyboardemu" is garbled near the end and does not decode correctly but this will not effect IR functions. Here is the full and corrected and decoded HID report descriptor.
```
unsigned char atv_hid_report_descriptor[] = {

0x05, 0x0c,	/* USAGE PAGE (Consumer)         	*/
0x09, 0x01,	/* USAGE (Pointer)                      */
0xa1, 0x01, 	/* COLLECTION (Application)             */

0x75, 0x08, 	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x01, 	/*  LOGICAL MINIMUM (1) 		*/
0x25, 0x02, 	/*  LOGIXAL MAXIMUM (2) 		*/
0x85, 0x01, 	/*  ReportID (1)			*/
0x09, 0xff, 	/*  Reserved				*/
0x81, 0x02, 	/*  INPUT (Data, Variable, Absolute)	*/
 
0x75, 0x08, 	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x02, 	/*  ReportID (2)			*/
0x09, 0xb4, 	/*  Rewind				*/
0x81, 0x02, 	/*  INPUT (Data, Variable, Absolute)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x03, 	/*  ReportID (3)			*/
0x09, 0xb3, 	/*  Fast Forward			*/
0x81, 0x02, 	/*  INPUT (Data, Variable, Absolute)	*/

0x75, 0x08, 	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x04, 	/*  ReportID (4)			*/
0x09, 0x40, 	/*  Menu				*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x05, 0x01, 	/* USAGE PAGE (Generic Desktop)         */

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x01, 	/*  LOGICAL MINIMUM (1) 		*/
0x25, 0x04, 	/*  LOGIXAL MAXIMUM (4) 		*/
0x85, 0x05, 	/*  ReportID (5)			*/
0x09, 0xff, 	/*  Reserved				*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x06, 	/*  ReportID (6)			*/
0x09, 0x86, 	/*  System App Menu			*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x07, 	/*  ReportID (7)			*/
0x09, 0x89, 	/*  System Menu Select			*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x08, 	/*  ReportID (8)			*/
0x09, 0x8a, 	/*  System Menu Right			*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x09, 	/*  ReportID (9)			*/
0x09, 0x8b, 	/*  System Menu Left			*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x01, 	/*  LOGICAL MINIMUM (1) 		*/
0x25, 0x02, 	/*  LOGIXAL MAXIMUM (2) 		*/
0x85, 0x0a, 	/*  ReportID (10)			*/
0x09, 0xff, 	/*  Reserved				*/
0x81, 0x02, 	/*  INPUT (Data, Variable, Absolute)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x0b, 	/*  ReportID (11)			*/
0x09, 0x8c, 	/*  System Menu Up			*/
0x81, 0x02, 	/*  INPUT (Data, Variable, Absolute)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x0c, 	/*  ReportID (12)			*/
0x09, 0x8d, 	/*  System Menu Down			*/
0x81, 0x02, 	/*  INPUT (Data, Variable, Absolute)	*/

0x05, 0xff, 	/* USAGE PAGE (Vender)			*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x01, 	/*  LOGICAL MINIMUM (1) 		*/
0x25, 0x04, 	/*  LOGIXAL MAXIMUM (4) 		*/
0x85, 0x0d, 	/*  ReportID (13)			*/
0x09, 0xff,	/*  Reserved				*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x0e, 	/*  ReportID (14)			*/
0x09, 0x20, 	/*  Vender 0x20				*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x0f, 	/*  ReportID (15)			*/
0x09, 0x21, 	/*  Vender 0x21				*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x10, 	/*  ReportID (16)			*/
0x09, 0x22, 	/*  Vender 0x22				*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0x01, 	/*  LOGIXAL MAXIMUM (1) 		*/
0x85, 0x11, 	/*  ReportID (17)			*/
0x09, 0x23, 	/*  Vender 0x23				*/
0x81, 0x06, 	/*  INPUT (Data, Variable, Relative)	*/

0x05, 0x06, 	/* USAGE PAGE (Generic Device Controls)	*/

0x75, 0x08,  	/*  REPORT SIZE (8) 			*/
0x95, 0x01, 	/*  REPORT COUNT (1)			*/
0x15, 0x00, 	/*  LOGICAL MINIMUM (0) 		*/
0x25, 0xff, 	/*  LOGIXAL MAXIMUM (255) 		*/
0x85, 0x12, 	/*  ReportID (18)			*/
0x09, 0x22, 	/*  Wireless ID				*/
0x81, 0x02, 	/*  INPUT (Data, Variable, Absolute)	*/ 14

0xc0		/* END					*/ 1

};
```

[Pulled from www.osxbook.com](http://www.osxbook.com/software/iremoted/) The usage table information for this device includes usages in the Generic Desktop Page (page ID 0x01), the Generic Device Controls Page (page ID 0x06), the Consumer Page (page ID 0x0C), and a vendor-defined page (page ID 0xff). In detail, there are;
  * Consumer -- includes on/off (up/down in this case) controls with the usage IDs 0x40 (Menu), 0xb3 (Fast Forward), and 0xb4 (Rewind).
  * Vendor -- includes usage IDs 0x20, 0x21, 0x22, and 0x23.
  * Generic Desktop -- includes one-shot controls (OSC) with usage IDs 0x86 (System App Menu), 0x89 (System Menu Select), 0x8a (System Menu Right), 0x8b (System Menu Left), 0x8c (System Menu Up), and 0x8d (System Menu Down).
  * Generic Device Controls -- includes a dynamic value with the usage ID ID 0x22 (Wireless ID), which identifies a wireless device in a wireless subsystem.


---

Info from Linux, "lsusb -v", if "Report Descriptor" shows as " UNAVAILABLE ", then "sudo modprobe -r usbhid" to expose it. Note that Linux only reports the "consumer" page.
```
Bus 001 Device 002: ID 05ac:8241 Apple Computer, Inc. 
Device Descriptor:
  bLength                18
  bDescriptorType         1
  bcdUSB               2.00
  bDeviceClass            0 (Defined at Interface level)
  bDeviceSubClass         0 
  bDeviceProtocol         0 
  bMaxPacketSize0         8
  idVendor           0x05ac Apple Computer, Inc.
  idProduct          0x8241 
  bcdDevice            2.42
  iManufacturer           1 Apple Computer, Inc.
  iProduct                2 IR Receiver
  iSerial                 0 
  bNumConfigurations      1
  Configuration Descriptor:
    bLength                 9
    bDescriptorType         2
    wTotalLength           59
    bNumInterfaces          2
    bConfigurationValue     1
    iConfiguration          1 Apple Computer, Inc.
    bmAttributes         0xa0
      (Bus Powered)
      Remote Wakeup
    MaxPower              100mA
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        0
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass         3 Human Interface Device
      bInterfaceSubClass      0 No Subclass
      bInterfaceProtocol      0 None
      iInterface              0 
        HID Device Descriptor:
          bLength                 9
          bDescriptorType        33
          bcdHID               1.11
          bCountryCode            0 Not supported
          bNumDescriptors         1
          bDescriptorType        34 Report
          wDescriptorLength      42
          Report Descriptor: (length is 42)
            Item(Global): Usage Page, data= [ 0x0c ] 12
                            Consumer
            Item(Local ): Usage, data= [ 0x01 ] 1
                            Consumer Control
            Item(Main  ): Collection, data= [ 0x01 ] 1
                            Application
            Item(Global): Logical Minimum, data= [ 0x00 ] 0
            Item(Global): Logical Maximum, data= [ 0xff 0x00 ] 255
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Global): Report Count, data= [ 0x04 ] 4
            Item(Global): Report ID, data= [ 0x24 ] 36
            Item(Local ): Usage, data= [ 0x00 ] 0
                            Unassigned
            Item(Main  ): Input, data= [ 0x22 ] 34
                            Data Variable Absolute No_Wrap Linear
                            No_Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Global): Report Count, data= [ 0x04 ] 4
            Item(Global): Report ID, data= [ 0x25 ] 37
            Item(Local ): Usage, data= [ 0x00 ] 0
                            Unassigned
            Item(Main  ): Input, data= [ 0x22 ] 34
                            Data Variable Absolute No_Wrap Linear
                            No_Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Global): Report Size, data= [ 0x08 ] 8
            Item(Global): Report Count, data= [ 0x04 ] 4
            Item(Global): Report ID, data= [ 0x26 ] 38
            Item(Local ): Usage, data= [ 0x00 ] 0
                            Unassigned
            Item(Main  ): Input, data= [ 0x22 ] 34
                            Data Variable Absolute No_Wrap Linear
                            No_Preferred_State No_Null_Position Non_Volatile Bitfield
            Item(Main  ): End Collection, data=none
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x82  EP 2 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval              10
    Interface Descriptor:
      bLength                 9
      bDescriptorType         4
      bInterfaceNumber        1
      bAlternateSetting       0
      bNumEndpoints           1
      bInterfaceClass       255 Vendor Specific Class
      bInterfaceSubClass      0 
      bInterfaceProtocol      0 
      iInterface              0 
      ** UNRECOGNIZED:  09 21 11 01 00 01 22 2a 00
      Endpoint Descriptor:
        bLength                 7
        bDescriptorType         5
        bEndpointAddress     0x81  EP 1 IN
        bmAttributes            3
          Transfer Type            Interrupt
          Synch Type               None
          Usage Type               Data
        wMaxPacketSize     0x0008  1x 8 bytes
        bInterval              10
Device Status:     0x0000
  (Bus Powered)
```



---

The interesting thing is there are two interfaces, the first is the HID which includes the IR receiver, the second interface also has an interrupt IN endpoint. What could this be? Well, examining the ioref dump gives us a good hint.
```
+-o USB1@1D  <class IOPCIDevice, registered, matched, active, busy 0, retain count 10>
| +-o AppleUSBUHCI  <class AppleUSBUHCI, !registered, !matched, active, busy 0, retain count 10>
|   |
|   +-o UHCI Root Hub Simulation@1D  <class IOUSBRootHubDevice, registered, matched, active, busy 0, retain count 13>
|   | +-o AppleUSBHub  <class AppleUSBHub, !registered, !matched, active, busy 0, retain count 4>
|   | +-o IOUSBInterface@0  <class IOUSBInterface, !registered, !matched, active, busy 0, retain count 6>
|   | +-o IOUSBUserClientInit  <class IOUSBUserClientInit, !registered, !matched, active, busy 0, retain count 4>
|   +-o Hub in Apple Extended USB Keyboard@1d100000  <class IOUSBDevice, !registered, !matched, active, busy 0, retain count 6>
|   |
|   +-o IR Receiver@1d200000  <class IOUSBDevice, registered, matched, active, busy 0, retain count 10>
|     +-o IOUSBCompositeDriver  <class IOUSBCompositeDriver, !registered, !matched, active, busy 0, retain count 4>
|     |
|     +-o IOUSBInterface@0  <class IOUSBInterface, registered, matched, active, busy 0, retain count 7>
|     | +-o AppleIRController  <class AppleIRController, registered, matched, active, busy 0, retain count 9>
|     | | +-o IOHIDInterface  <class IOHIDInterface, registered, matched, active, busy 0, retain count 6>
|     | | | +-o IOHIDEventDriver  <class IOHIDEventDriver, registered, matched, active, busy 0, retain count 8>
|     | | |   +-o IOHIDConsumer  <class IOHIDConsumer, registered, matched, active, busy 0, retain count 8>
|     | | |   | +-o IOHIDSystem  <class IOHIDSystem, registered, matched, active, busy 0, retain count 11>
|     | | |   | | +-o IOHIDUserClient  <class IOHIDUserClient, !registered, !matched, active, busy 0, retain count 5>
|     | | |   | | +-o IOHIDParamUserClient  <class IOHIDParamUserClient, !registered, !matched, active, busy 0, retain count 5>
|     | | |   | +-o IOBSDConsole  <class IOBSDConsole, !registered, !matched, active, busy 0, retain count 5>
|     | | |   +-o IOHIDSystem  <class IOHIDSystem, registered, matched, active, busy 0, retain count 10>
|     | | |     +-o IOHIDUserClient  <class IOHIDUserClient, !registered, !matched, active, busy 0, retain count 5>
|     | | |     +-o IOHIDParamUserClient  <class IOHIDParamUserClient, !registered, !matched, active, busy 0, retain count 5>
|     | | +-o IOUSBUserClientInit  <class IOUSBUserClientInit, !registered, !matched, active, busy 0, retain count 4>
|     | | +-o IOHIDLibUserClient  <class IOHIDLibUserClient, !registered, !matched, active, busy 0, retain count 6>
|     | +-o IOUSBUserClientInit  <class IOUSBUserClientInit, !registered, !matched, active, busy 0, retain count 4>
|     |
|     +-o IOUSBInterface@1  <class IOUSBInterface, registered, matched, active, busy 0, retain count 7>
|     | +-o AppleSMC  <class AppleSMC, registered, matched, active, busy 0, retain count 4>
|     | +-o IOUSBUserClientInit  <class IOUSBUserClientInit, !registered, !matched, active, busy 0, retain count 4>
|     |
|     +-o IOUSBUserClientInit  <class IOUSBUserClientInit, !registered, !matched, active, busy 0, retain count 4>
```

"IOUSBInterface@0" uses endpoint 2 IN, and "IOUSBInterface@1" uses endpoint 1 IN. Well what do you know, that's where AppleSMC is hiding.
```
+-o IOUSBInterface@1  <class IOUSBInterface, registered, matched, active, busy 0, retain count 7>
| | {
| |   "IOUserClientClass" = "IOUSBInterfaceUserClient"
| |   "idProduct" = 33345
| |   "IOCFPlugInTypes" = {"2d9786c6-9ef3-11d4-ad51-000a27052861"="IOUSBFamily.kext/Contents/PlugIns/IOUSBLib.bundle"}
| |   "iInterface" = 0
| |   "bAlternateSetting" = 0
| |   "bConfigurationValue" = 1
| |   "bInterfaceProtocol" = 0
| |   "bInterfaceNumber" = 1
| |   "bInterfaceSubClass" = 0
| |   "idVendor" = 1452
| |   "bInterfaceClass" = 255
| |   "locationID" = 488636416
| |   "bNumEndpoints" = 1
| |   "bcdDevice" = 578
| | }
| | 
| +-o AppleSMC  <class AppleSMC, registered, matched, active, busy 0, retain count 4>
| |   {
| |     "idProduct" = 33345
| |     "bConfigurationValue" = 1
| |     "CFBundleIdentifier" = "com.apple.driver.AppleSMC"
| |     "IOClass" = "AppleSMC"
| |     "IOProbeScore" = 20000
| |     "IOMatchCategory" = "IODefaultMatchCategory"
| |     "bInterfaceNumber" = 1
| |     "IOUserClientClass" = "AppleSMCClient"
| |     "idVendor" = 1452
| |     "IOProviderClass" = "IOUSBInterface"
| |   }
| |   
| +-o IOUSBUserClientInit  <class IOUSBUserClientInit, !registered, !matched, active, busy 0, retain count 4>
|     {
|       "IOMatchCategory" = "IOUSBUserClientInit"
|       "IOProbeScore" = 9000
|       "IOClass" = "IOUSBUserClientInit"
|       "IOProviderClass" = "IOUSBInterface"
|       "CFBundleIdentifier" = "com.apple.iokit.IOUSBUserClient"
|       "IOProviderMergeProperties" = {"IOUserClientClass"="IOUSBInterfaceUserClient","IOCFPlugInTypes"={"2d9786c6-9ef3-11d4-ad51-000a27052861"="IOUSBFamily.kext/Contents/PlugIns/IOUSBLib.bundle"}}
|     }
```