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

## Quick Hack Guide on Debian/Ubuntu

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