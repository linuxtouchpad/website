---
layout: page
title: Docs
permalink: /docs/
---

## The Input Stack

Touchpad input takes a long journey from physical finger movements to the application layer:

<div class="img-centered img-large">
    <img src="/assets/images/touchpad-stack.png">
</div>

1. Finger movements affect capacitive touch circuitry.
2. Firmware embedded in the device at the time of manufacture interprets electrical signals. Some convey proximity, pressure, orientation, or simply absolute X/Y coordinates of fingers. A modern specification for touchpad firmware<->OS integration is the [Windows Precision Touchpad](https://docs.microsoft.com/en-us/windows-hardware/design/component-guidelines/windows-precision-touchpad-implementation-guide) spec.
3. The linux kernel combines hardware-specific custom drivers and general Human Interface Device drivers ([hid](https://github.com/torvalds/linux/tree/master/drivers/hid), [hid-multitouch](https://github.com/torvalds/linux/blob/master/drivers/hid/hid-multitouch.c)) to provide a stream of data that is in relative or absolute coordinates (usually absolute), and context-free or slotted (usually slotted, meaning each "finger" corresponds to a data slot).
4. All touchpad events, including `ABS_MT_*` [multitouch events](https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h) are emitted by the kernel and made available to userspace via one of the `/dev/input/event*` file descriptors. Older touchpads may emit relative touch events or multitouch events without contact tracking.
5. Gesture remappers take `libinput` events such as gestures as input, and remap them to user-defined or desktop environment norms. Their output is then looped back into `libinput` at the next layer as well. Sometimes gesture remappers also extend into the User Interface domain by providing customization GUIs or showing graphics and animations (but not always).
6. The Linux graphical desktop world is currently divided into X.Org and Wayland, with the former being the "stable classic" and the latter being the "eventual replacement" for X.  That said, both systems now depend on `libinput` as part of their stack (see #8 below).

    (a) Gesture remappers (e.g. [Touchégg](https://github.com/JoseExposito/touchegg), [libinput-gestures](https://github.com/bulletmark/libinput-gestures), [fusuma](https://github.com/iberianpig/fusuma), [gebaar](https://github.com/Coffee2CodeNL/gebaar-libinput)) accept gesture events from libinput and custom emit key, button, or position events.

    (b) Xserver employs a "driver" such as [xf86-input-libinput](https://gitlab.freedesktop.org/xorg/driver/xf86-input-libinput) to effectively mirror what `libinput` produces. Xserver used to have many device drivers for input, but libinput has consolidated them and there is usually no need for them any more.

    (c) Xserver interprets and re-emits events.
7. The application responds to touch events (position, tap, scroll, gestures etc.)
8. Finally we can talk about [libinput](https://gitlab.freedesktop.org/libinput/libinput)! `libinput` is a cornerstone library for the touchpad input stack that can be used by several layers of the stack at any given time. It contains a hardware database to provide physical measurements, normalize DPI, and handle device-specific quirks. In addition, it converts absolute position touchpad coordinates to relative mouse pointer movements, detects gestures such as two-finger scrolling and others, and improves quality of life through palm detection and DWT ("disable [the touchpad] while typing"). [mtdev](http://bitmath.org/code/mtdev/) is also empoyed to convert to "slotted" contact tracking events if needed. Because `libinput`

For most touchpad hackers, the 3 most important layers are "kernel", "libinput", and "application":

- The `kernel` must be aware of and compatible with the touchpad hardware & firmware. If your touchpad is not working at all and it's a new model, perhaps you'll need to work at the kernel level.
- The `libinput` layer is the userland layer that unifies all of the differences in capabilities, DPI, and quirks so that subsequent layers don't need to know about particular hardware. If your touchpad is "working" but you're experiencing some annoyance with regard to reliable heuristics, it's likely at the `libinput` level that you'd like to focus your attention.
- The `application` need only concern itself with wrapped events, e.g. GTK or QT library events.

## Finding Your Touchpad Device

### Kernel

The kernel uses its `udev` database system to track attributes and properties of various devices as they are initialized or added/removed.

```
$ find /dev/input/event* -maxdepth 1 \
  | sudo xargs udevadm info \
  | grep -20 -i touchpad

P: /devices/pci0000:00/0000:00:15.3/i2c_designware.2/i2c-12/i2c-PIXA3854:00/0018:093A:0274.0002/input/input10/event8
N: input/event8
L: 0
S: input/by-path/pci-0000:00:15.3-platform-i2c_designware.2-event-mouse
E: DEVPATH=/devices/pci0000:00/0000:00:15.3/i2c_designware.2/i2c-12\
     /i2c-PIXA3854:00/0018:093A:0274.0002/input/input10/event8
E: DEVNAME=/dev/input/event8
E: MAJOR=13
E: MINOR=72
E: SUBSYSTEM=input
E: USEC_INITIALIZED=4797212
E: ID_INPUT=1
E: ID_INPUT_TOUCHPAD=1
E: ID_INPUT_WIDTH_MM=111
E: ID_INPUT_HEIGHT_MM=73
E: ID_SERIAL=noserial
E: ID_PATH=pci-0000:00:15.3-platform-i2c_designware.2
E: ID_PATH_TAG=pci-0000_00_15_3-platform-i2c_designware_2
E: LIBINPUT_DEVICE_GROUP=18/93a/274:i2c-PIXA3854:00
E: DEVLINKS=/dev/input/by-path\
     /pci-0000:00:15.3-platform-i2c_designware.2-event-mouse
```

The `DEVNAME` property above shows `/dev/input/event8` as the touchpad device.

You can also see what kernel module(s) may be involved in making this touchpad device work. The "MAJOR" and "MINOR" properties above map to 13, 72. Or, using `ls` (or `exa`), you can aslo see the major, minor numbers of the device:

```
$ ls -l /dev/input/event8
crw-rw---- 1 root input 13, 72 Nov 28 10:42 /dev/input/event8
```

The `input 13, 72` tells us this is an input device (i.e. character device, not a block device) with major number 13 and minor number 72. These can be found in the `/sys/dev/char` (character) directory:

```
$ ls /sys/dev/char/13\:72/
dev        device/    power/     subsystem/ uevent 
```

Within the `device/` folder, you should be able to see what kernel module is driving:

```
$ ls -l /sys/dev/char/13\:72/device/device/driver
lrwxrwxrwx 1 root root 0 Dec  2 09:28 \
  /sys/dev/char/13:72/device/device/driver -> \
  ../../../../../../../bus/hid/drivers/hid-multitouch
```

More info can be found at [How to find the driver associated with a device on Linux](https://unix.stackexchange.com/questions/97676/how-to-find-the-driver-module-associated-with-a-device-on-linux).

Another tool you can use to find what's driving your device at the kernel level is `lsmod`:

```
$ lsmod | grep touch
hid_multitouch         28672  0
hid                   139264  6 i2c_hid,usbhid,hid_multitouch,hid_sensor_hub,intel_ishtp_hid,hid_generic
```

The comma-separated list indicates what modules *depend on* this module. So `hid_multitouch` depends on `hid`.

### Xorg

If you're using Xorg, you can introspect your input devices like so:

    $ xinput list

    ⎡ Virtual core pointer                    	id=2	[master pointer  (3)]
    ⎜   ↳ Virtual core XTEST pointer              	id=4	[slave  pointer  (2)]
    ⎜   ↳ FRMW0001:00 32AC:0006 Consumer Control  	id=10	[slave  pointer  (2)]
    ⎜   ↳ PIXA3854:00 093A:0274 Touchpad          	id=11	[slave  pointer  (2)]
    ⎜   ↳ PIXA3854:00 093A:0274 Mouse             	id=12	[slave  pointer  (2)]
    ⎜   ↳ keyd virtual pointer                    	id=15	[slave  pointer  (2)]
    ⎣ Virtual core keyboard                   	id=3	[master keyboard (2)]
        ↳ Virtual core XTEST keyboard             	id=5	[slave  keyboard (3)]
        [...snip...]


To see what `libinput` settings are enabled and disabled for your touchpad, e.g.:

    $ xinput --list-props "PIXA3854:00 093A:0274 Touchpad"

    Device 'PIXA3854:00 093A:0274 Touchpad':
        Device Enabled (177):	1
        Coordinate Transformation Matrix (179):	1.0, 0.0, 0.0, 0.0, 1.0, 0.0, 0.0, 0.0, 1.0
        libinput Tapping Enabled (316):	1
        libinput Tapping Enabled Default (317):	0
        libinput Tapping Drag Enabled (318):	1
        libinput Tapping Drag Enabled Default (319):	1
        libinput Tapping Drag Lock Enabled (320):	0
        [...snip...]

### Hardware-reported Capabilities: Report Descriptors

Each HID device has a hardware "report descriptor" that is sent from the device itself to the kernel to describe its capabilities. The data is stored in a binary format called a *HID report descriptor*.

Find your device's `report_descriptor` file, and then dump as hex:

```
cat /sys/devices/pci0000\:00/0000\:00\:15.3/\
    i2c_designware.2/i2c-12/i2c-PIXA3854\:00/0018\:093A\:0274.0002\
    /report_descriptor \
| hexdump -e '16/1 "%02x " "\n"'

05 01 09 02 a1 01 85 02 05 01 09 01 a1 00 05 09
19 01 29 02 15 00 25 01 75 01 95 02 81 02 05 0d
[...snip...]
```

Copy-paste the entire hex byte sequence to [Frank's USB Descriptor and Request Parser](http://eleccelerator.com/usbdescreqparser/). Click "USB HID Report Descriptor" and you'll get output something like this:

```
[...snip mouse collection...]
0x05, 0x0D,        // Usage Page (Digitizer)
0x09, 0x05,        // Usage (Touch Pad)
0xA1, 0x01,        // Collection (Application)
0x85, 0x01,        //   Report ID (1)
0x05, 0x0D,        //   Usage Page (Digitizer)
0x09, 0x22,        //   Usage (Finger)
0xA1, 0x02,        //   Collection (Logical)
0x09, 0x47,        //     Usage (0x47)
0x09, 0x42,        //     Usage (Tip Switch)
0x15, 0x00,        //     Logical Minimum (0)
0x25, 0x01,        //     Logical Maximum (1)
0x75, 0x01,        //     Report Size (1)
0x95, 0x02,        //     Report Count (2)
0x81, 0x02,        //     Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x95, 0x06,        //     Report Count (6)
0x81, 0x03,        //     Input (Const,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x09, 0x51,        //     Usage (0x51)
0x25, 0x0F,        //     Logical Maximum (15)
0x75, 0x08,        //     Report Size (8)
0x95, 0x01,        //     Report Count (1)
0x81, 0x02,        //     Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x05, 0x01,        //     Usage Page (Generic Desktop Ctrls)
0x09, 0x30,        //     Usage (X)
0x75, 0x10,        //     Report Size (16)
0x55, 0x0E,        //     Unit Exponent (-2)
0x65, 0x11,        //     Unit (System: SI Linear, Length: Centimeter)
0x35, 0x00,        //     Physical Minimum (0)
0x46, 0x5A, 0x04,  //     Physical Maximum (1114)
0x27, 0x39, 0x05, 0x00, 0x00,  //     Logical Maximum (1336)
0x81, 0x02,        //     Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0x09, 0x31,        //     Usage (Y)
0x46, 0xDA, 0x02,  //     Physical Maximum (730)
0x27, 0x6C, 0x03, 0x00, 0x00,  //     Logical Maximum (875)
0x81, 0x02,        //     Input (Data,Var,Abs,No Wrap,Linear,Preferred State,No Null Position)
0xC0,              //   End Collection
0x05, 0x0D,        //   Usage Page (Digitizer)
0x09, 0x22,        //   Usage (Finger)
0xA1, 0x02,        //   Collection (Logical)
0x09, 0x47,        //     Usage (0x47)
0x09, 0x42,        //     Usage (Tip Switch)
[...snip additional finger collections...]
```

### Kernel-reported Capabilities

While the hardware may report one thing, the kernel may report another, depending on the compatibility and state of the driver. Here is a method to inspect what the kernel is saying the device is capable of.

```
# Note: `cat *` would work, but we want to see the filenames next to their
# contents, so we use `tail -n +1`. We cd to the capabilities dir so that 
# filenames aren't so long:

$ (cd /sys/devices/pci0000\:00/0000\:00\:15.3/\
    i2c_designware.2/i2c-12/i2c-PIXA3854\:00/0018\:093A\:0274.0002/\
    input/input10/capabilities/;
  tail -n +1 *)

==> abs <==
2e0800000000003

==> ev <==
1b

==> ff <==
0

==> key <==
e520 30000 0 0 0 0

==> led <==
0

==> msc <==
20

==> rel <==
0

==> snd <==
0

==> sw <==
0
```

The above files and their numeric contents represent HID "capability bits". The Linux Humain Interface Devices spec has several categories of devices that correspond to the short names above. See the [kernel.org input event-codes documentation](https://www.kernel.org/doc/html/latest/input/event-codes.html#event-types).

For our touchpad hacking purposes, we're mostly interested in the non-zero capabilities:

- abs: ABS_ prefix, indicates a device capable of sending absolute coordinates (like a touchpad!)
- ev: EV_ prefix, indicates a device capable of sending generic events
- key: KEY_ prefix, indicates a device capable of sending key events (key down, key up, key repeat).
- msc: MSC_ prefix, indicates a device capable of sending "misc" events, such as MSC_TIMESTAMP (a piece of information that can be sent as a kind of heartbeat, as in the case of fingers not moving but still touching the touchpad--for example when two fingers are touching but not moving it can be a signal that means "put on the brakes" for inertial scrolling).

How do you decode the exact capability bits in each non-zero category of HID input even types? See this [stackexchange contribution](https://unix.stackexchange.com/questions/74903/explain-ev-in-proc-bus-input-devices-data/74907#74907) by [Runium](https://unix.stackexchange.com/users/28489/runium).

In my case, the above bits work out to:

```
ev: 1b
   7654 3210
   0001 1011
   Set Bits: 0, 1, 3, 4 => EV_SYN, EV_KEY, EV_ABS, EV_MSC

key: e520 30000 0 0 0 0
   #TODO: work out which keys/buttons this represents
 
abs: 2e0800000000003
   0010 1110 0000 1000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0000 0011
   Set Bits: 0, 1, 47, 53, 54, 55, 57
   => ABS_X, ABS_Y, ABS_MT_SLOT, ABS_MT_POSITION_X, ABS_MT_POSITION_Y, ABS_MT_TOOL_TYPE, ABS_MT_TRACKING_ID

msc: 20
   0010 0000
   Set Bits: 5
   => MSC_TIMESTAMP
```

## Quick Hack Steps on Debian/Ubuntu

If you're on a Debian-based system like Ubuntu, you can quite easily modify libinput source code, compile, and test your changes with the following commands:

    sudo apt build-dep libinput
    sudo apt install fakeroot
    apt source libinput
    cd libinput-1.19.2/
    # edit code! e.g. `vim src/evdev-mt-touchpad.c`
    dpkg-buildpackage -b
    sudo dpkg -i -O ../libinput*.deb

## Resources

See <a href="{% link resources.markdown %}">resources</a>.