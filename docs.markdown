---
layout: page
title: Docs
permalink: /docs/
---

## The Input Stack

Touchpad input takes a long journey from physical finger movements to the application layer:

1. Finger movements affect capacitive touch circuitry
2. Firmware
3. Linux kernel (hid, hid-multitouch)
4. /dev/input/event* Multitouch Events
5. libinput
6. Xserver / Wayland
7. Gesture detection (Touchegg, libinput-gestures, fusuma, gebaar)
8. Application responds to touch events (position, tap, scroll, gestures etc.)

For most touchpad hackers, the 2 most important layers are "kernel" and "libinput":

- The `kernel` must be aware of and compatible with the touchpad hardware & firmware. If your touchpad is not working at all and it's a new model, perhaps you'll need to work at the kernel level.
- The `libinput` layer is the userland layer that unifies all of the differences in capabilities, DPI, and quirks so that subsequent layers don't need to know about particular hardware. If your touchpad is "working" but you're experiencing some annoyance with regard to reliable heuristics, it's likely at the `libinput` level that you'd like to focus your attention.

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