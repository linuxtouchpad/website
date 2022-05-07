---
layout: post
title:  "How to Diagnose Disable-While-Typing Issues"
date:   2022-05-07 08:14:11 -0700
categories: libinput
---

One of the more annoying things about typing with a large touchpad on a laptop, is that the palms or thumbs can occasionally brush past the touchpad and cause the mouse cursor to move. This disrupts your typing flow, and occasionally even worse--for example, clicking the submit button before you're ready, or deleting text you just typed.

Unfortunately, the Linux experience is not quite optimal yet. Some touchpads are to blame--they may not properly report "palm" events--but there may also be some improvements to be made on the software stack (TBD).

In the meantime, there is a fairly simple "hack" called DWT ("Disable While Typing") that has been built by the [libinput](https://gitlab.freedesktop.org/libinput/libinput) team that allows you to "disable" the touchpad while you are typing. See the [libinput docs](https://wayland.freedesktop.org/libinput/doc/latest/palm-detection.html#disable-while-typing).

Note: DWT works well in most cases, but fails for things like gaming where you want to be able to use the touchpad while controlling a character (for example), or web apps like Miro that require you to hold the spacebar while moving the canvas around with the pointer (touchpad).

## Turn on Disable While Typing (DWT)

On most Linux systems, DWT is enabled through a settings panel.

### Gnome

In the Gnome desktop environment, you can visit "Settings -> Mouse & Touchpad -> Touchpad -> Disable while typing":

<img src="/assets/images/2022-05-07-gnome-dwt.png">

If you use Gnome, but this settings panel is not available, you can use `gsettings` command-line tool to enable or disable DWT:

`gsettings set org.gnome.desktop.peripherals.touchpad disable-while-typing true`

### KDE Plasma

In the KDE desktop environment, you can visit "System Settings -> Touchpad settings -> Enable/Disable Touchpad -> Disable touchpad when typing".

## What if DWT Doesn't Work?

For "Disable While Typing" to work, one of a pair of conditions must be met. Consider that the most common scenario where DWT is needed is in the "touchpad below the keyboard" laptop scenario. This is the scenario that `libinput` is trying to detect, using the following heuristics:

1. The keyboard and touchpad are both "internal" devices (i.e. not plugged in via USB), or
2. The keyboard and touchpad IDs match (i.e. their vendor ID, product ID are the same) and can be treated as a pair.

See [tp_want_dwt](https://github.com/wayland-project/libinput/blob/10124797b502f3dd308919b7bab80752483d0f6b/src/evdev-mt-touchpad.c#L2336) in `libinput`'s source if you'd like to understand more about this logic.

If `libinput` detects neither of the above conditions, then the "Disable While Typing" setting will do nothing.

But don't despair! If you're in this situation, you can still likely fix this with a [quirks file](https://wayland.freedesktop.org/libinput/doc/latest/device-quirks.html#device-quirks-local) for your hardware, but hinting to `libinput` that your keyboard or touchpad is "internal".

Although `libinput` is most commonly known as just a library, there is actually a corresponding command-line tool--also called `libinput`--that you can install via the `libinput-tools` package to diagnose and debug issues. For example, on Debian/Ubuntu-based systems: `sudo apt install libinput-tools`.

With the `libinput` command freshly installed, you can now diagnose your hardware with `sudo libinput list-devices`:

```
...

Device:           PIXA3854:00 093A:0274 Touchpad
Kernel:           /dev/input/event7
Group:            6
Seat:             seat0, default
Size:             111x73mm
Capabilities:     pointer gesture
Tap-to-click:     disabled
Tap-and-drag:     enabled
Tap drag lock:    disabled
Left-handed:      disabled
Nat.scrolling:    disabled
Middle emulation: disabled
Calibration:      n/a
Scroll methods:   *two-finger edge 
Click methods:    *button-areas clickfinger 
Disable-w-typing: enabled
Accel profiles:   flat *adaptive
Rotation:         n/a

...

Device:           AT Translated Set 2 keyboard
Kernel:           /dev/input/event2
Group:            7
Seat:             seat0, default
Capabilities:     keyboard 
Tap-to-click:     n/a
Tap-and-drag:     n/a
Tap drag lock:    n/a
Left-handed:      n/a
Nat.scrolling:    n/a
Middle emulation: n/a
Calibration:      n/a
Scroll methods:   none
Click methods:    none
Disable-w-typing: n/a
Accel profiles:   n/a
Rotation:         n/a

...

Device:           keyd virtual device
Kernel:           /dev/input/event9
Group:            9
Seat:             seat0, default
Capabilities:     keyboard 
Tap-to-click:     n/a
Tap-and-drag:     n/a
Tap drag lock:    n/a
Left-handed:      n/a
Nat.scrolling:    n/a
Middle emulation: n/a
Calibration:      n/a
Scroll methods:   none
Click methods:    none
Disable-w-typing: n/a
Accel profiles:   n/a
Rotation:         n/a
```

Note that on my system (where Disable While Typing is working) I have a "virtual keyboard" created by [keyd](https://github.com/rvaiya/keyd/). I want this to pair with my touchpad for DWT.

Using the "Kernel" devices from the list above, I can also check for 'quirks':

```
$ sudo libinput quirks list /dev/input/event7
AttrEventCodeDisable=BTN_RIGHT;

$ sudo libinput quirks list /dev/input/event2
AttrKeyboardIntegration=internal

$ sudo libinput quirks list /dev/input/event9
AttrKeyboardIntegration=internal
```

If your keyboard and touchpad do not share the same vid/pid, then you can inform `libinput` that a keyboard and a touchpad are `internal` via a quirks file (see below). There should be only 1 internal keyboard and 1 internal touchpad. If there are more than 3 matching keyboards, [pairing will not work](https://github.com/wayland-project/libinput/blob/10124797b502f3dd308919b7bab80752483d0f6b/src/evdev-mt-touchpad.c#L2373).

### Creating a Quirks File

Let's suppose that your keyboard is not being recognized as "internal". Add the following file, with `MatchName` adjusted to match the name of your keyboard, to `/etc/libinput/local-overrides.quirks`:

```
[Serial Keyboards]
MatchUdevType=keyboard
MatchName=keyd virtual keyboard
AttrKeyboardIntegration=internal
```

Voila! Now your keyboard is "internal", and as long as your touchpad is also "internal", the Disable While Typing option will work as expected.

