**About**

Firmware for a Teensy device that interfaces with some old Xbox 360 parts that I've integrated into a PC.

What the firmware does is:

* receives IR signals from an Xbox 360 remote and maps them to USB HID commands
* interfaces with the Xbox 360 RF module to initiate controller sync, and control the LED ring
* monitors the front side buttons (sync and eject) and handles them as necessary

Here's a WIP pic of how the Teensy is connected. Each of the components labelled in RED are salvaged from an old (and broken) Xbox 360 motherboard. Not shown in the image is that the Teensy is connected to the motherboard via its USB port, and it also connects to the front panel jumers power_switch (to turn on/off the PC) and power_led (to detect when the PC is on/off).

![board_layout_labelled.jpg](docs/images/board_layout_labelled.jpg?raw=true)

**Teensy Model**

The model I used for this was the Teensy 2.0, but it should work on any Teensy
model without much modification (I think only the I/O pins may need to be
changed).

**Teensy Library Modifications**

The Teensy is seen by the PC as a USB HID device and uses a custom HID report, so that some IR commands get mapped to keyboard, and some to a consumer control device (this is useful for the media buttons). This requires a slight modification to a couple of files in the Teensy USB RawHID library (arduino/hardware/teensy/cores/usb_rawhid):

In file usb_private.h, change the transmitted report size from 64 to 9:

```
#!c

#define RAWHID_TX_SIZE		9
```


And in file usb.c, replace the rawhid_hid_report_desc array with:

```
#!c

static uint8_t PROGMEM rawhid_hid_report_desc[] = {
    0x05, 0x01, /* Usage Page (Generic Desktop)             */
    0x09, 0x06, /* Usage (Keyboard)                         */
    0xA1, 0x01, /* Collection (Application)                 */
    0x85, 0x01, //   REPORT_ID (1)
    0x05, 0x07, /*      Usage page (Key Codes)              */
    0x19, 0xE0, /*      Usage minimum (224)                 */
    0x29, 0xE7, /*      Usage maximum (231)                 */
    0x15, 0x00, /*      Logical minimum (0)                 */
    0x25, 0x01, /*      Logical maximum (1)                 */
    0x75, 0x01, /*      Report size (1)                     */
    0x95, 0x08, /*      Report count (8)                    */
    0x81, 0x02, /*      Input (data, variable, absolute)    */
    0x95, 0x01, /*      Report count (1)                    */
    0x75, 0x08, /*      Report size (8)                     */
    0x81, 0x01, /*      Input (constant)                    */
    0x95, 0x06, /*      Report count (6)  */
    0x75, 0x08, /*      Report size (8)                     */
    0x15, 0x00, /*      Logical minimum (0)                 */
    0x25, 0xFF, /*      Logical maximum (255)               */
    0x05, 0x07, /*      Usage page (key codes)*/
    0x19, 0x00, /*      Usage minimum (0)                   */
    0x29, 0xFF, /*      Usage maximum (255)                 */
    0x81, 0x00, /*      Input (data, array)                 */
    0xC0,       /*      End Collection               */
    0x05, 0x0c,                    // USAGE_PAGE (Consumer Devices)
    0x09, 0x01,                    // USAGE (Consumer Control)
    0xa1, 0x01,                    //   COLLECTION (Application)
    0x05, 0x0c,                    // USAGE_PAGE (Consumer Devices)
    0x85, 0x02,                    //   REPORT_ID (2)
    0x15, 0x00,                    //   LOGICAL_MINIMUM (0)
    0x25, 0x01,                    //   LOGICAL_MAXIMUM (1)
    0x75, 0x01,                    //   REPORT_SIZE (1)
    0x95, 0x09,                    //   REPORT_COUNT (9)
    0x09, 0x30,                    //   USAGE (Power)
    0x09, 0xb8,                    //   USAGE (Eject)
    0x09, 0xcd,                    //   USAGE (Play/Pause)
    0x09, 0xb7,                    //   USAGE (Stop)
    0x09, 0xcd,                    //   USAGE (Play/Pause)
    0x09, 0xb4,                    //   USAGE (Rewind)
    0x09, 0xb3,                    //   USAGE (Fast Forward)
    0x09, 0xb6,                    //   USAGE (Scan Previous Track)
    0x09, 0xb5,                    //   USAGE (Scan Next Track)
    0x81, 0x02,                    //   INPUT (Data,Var,Abs)
    0x75, 0x01,                    // REPORT_SIZE (1)
    0x95, 0x07,                    // REPORT_COUNT (7)
    0x81, 0x01,                    // INPUT (Cnst,Ary,Abs)
    0x75, 0x08,                    // REPORT_SIZE (8)
    0x95, 0x06,                    // REPORT_COUNT (6)
    0x81, 0x01,                    // INPUT (Cnst,Ary,Abs)
    0xc0                           //               END_COLLECTION
};
```

**References**

* [XBOX 360 Motherboard Headers and Connector V1.4 documentation](http://dwl.xbox-scene.com/tutorial/Xbox_360-HandC-V1_4.pdf) for how to wire up the Xbox 360 components
* [RF Module serial protocol](http://forums.xbox-experts.com/viewtopic.php?f=13&t=4029&sid=d67fad5f1eb2e7254ee59c3233302624)  (see the linked zip for some useful code and explanation), also [here](http://tkkrlab.nl/wiki/XBOX_360_RF_Module) and [here](http://techocd.blogspot.com.au/2014/01/xbox-360-wireless-controller-to-pc-via.html)
* [Tutorial on USB HID report descriptors](http://eleccelerator.com/tutorial-about-usb-hid-report-descriptors/), very useful to understand the USB HID spec which can be quite tricky, see also the official [class definition](http://www.usb.org/developers/hidpage/HID1_11.pdf) and [usage tables](http://www.usb.org/developers/hidpage/Hut1_12v2.pdf)
* [Using an Xbox 360 remote with arduino](http://www.righto.com/2010/12/64-bit-rc6-codes-arduino-and-xbox.html), also [this](http://www.remotecentral.com/cgi-bin/mboard/rc-discrete/thread.cgi?5489) might be useful but I didn't end up using it
