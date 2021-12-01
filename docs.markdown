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
2. Firmware embedded in the device at the time of manufacture interprets electrical signals.
3. The linux kernel combines hardware-specific custom drivers and general Human Interface Device drivers ([hid](https://github.com/torvalds/linux/tree/master/drivers/hid), [hid-multitouch](https://github.com/torvalds/linux/blob/master/drivers/hid/hid-multitouch.c))
4. `ABS_MT_*` [multitouch events](https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h) are emitted by the kernel and made available to userspace via one of the `/dev/input/event*` file descriptors. Older touchpads may emit relative touch events or multitouch events without contact tracking.
5. [libinput](https://gitlab.freedesktop.org/libinput/libinput) normalizes DPI, converts absolute position to relative (mouse pointer) position, handles device-specific quirks, and detects gestures such as two-finger scrolling and others. [mtdev](http://bitmath.org/code/mtdev/) is also empoyed to convert to "slotted" contact tracking events.
6. The Wayland compositor listens direction to libinput and fulfills all parts of the device and gesture stack.

    (a) Gesture remappers (e.g. [Touchégg](https://github.com/JoseExposito/touchegg), [libinput-gestures](https://github.com/bulletmark/libinput-gestures), [fusuma](https://github.com/iberianpig/fusuma), [gebaar](https://github.com/Coffee2CodeNL/gebaar-libinput)) accept gesture events from libinput and custom emit key, button, or position events.

    (b) Xserver employs a "driver" such as [xf86-input-libinput](https://gitlab.freedesktop.org/xorg/driver/xf86-input-libinput) to effectively mirror what `libinput` produces. Xserver used to have many device drivers for input, but libinput has consolidated them and there is usually no need for them any more.

    (c) Xserver interprets and re-emits events.
7. Application responds to touch events (position, tap, scroll, gestures etc.)

For most touchpad hackers, the 3 most important layers are "kernel", "libinput", and "application":

- The `kernel` must be aware of and compatible with the touchpad hardware & firmware. If your touchpad is not working at all and it's a new model, perhaps you'll need to work at the kernel level.
- The `libinput` layer is the userland layer that unifies all of the differences in capabilities, DPI, and quirks so that subsequent layers don't need to know about particular hardware. If your touchpad is "working" but you're experiencing some annoyance with regard to reliable heuristics, it's likely at the `libinput` level that you'd like to focus your attention.
- The `application` need only concern itself with wrapped events, e.g. GTK or QT library events.

## Finding Your Touchpad Device

### Kernel

    $ lsmod | grep touch

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



## Quick Hack Steps on Debian/Ubuntu

If you're on a Debian-based system like Ubuntu, you can quite easily modify libinput source code, compile, and test your changes with the following commands:

    sudo apt build-dep libinput
    sudo apt install fakeroot
    apt source libinput
    cd libinput-1.12.6/
    # edit code! e.g. `vim src/evdev-mt-touchpad.c`
    dpkg-buildpackage -b
    sudo dpkg -i -O ../libinput*.deb

## Resources

See <a href="{% link resources.markdown %}">resources</a>.