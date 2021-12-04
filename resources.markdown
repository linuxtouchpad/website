---
layout: page
title: Resources
permalink: /resources/
---

## Core Projects

- [https://code.launchpad.net/mtdev](https://code.launchpad.net/mtdev) - converts kernel events to modern slotted B type events
- [https://gitlab.freedesktop.org/libinput/libinput](https://gitlab.freedesktop.org/libinput/libinput) - the workhorse of touchpad hardware unification
- [https://gitlab.freedesktop.org/xorg/xserver](https://gitlab.freedesktop.org/xorg/xserver) - the most well-established graphical environment on Linux


## Project Documentation

Some good overview docs:

- [What is libinput?](https://wayland.freedesktop.org/libinput/doc/latest/what-is-libinput.html) (libinput docs)
- [libinput - touchpads](https://wayland.freedesktop.org/libinput/doc/latest/touchpads.html#touchpads) (libinput docs)

Help is also available for end users:

- [Arch Linux - libinput](https://wiki.archlinux.org/title/Libinput) (troubleshooting & customization)

## Tools

- [evemu](https://gitlab.freedesktop.org/libevdev/evemu) (C, Python)
  - Record input events and play them back on a virtual device

- [cleartouch](https://github.com/canadaduane/cleartouch) (Zig, 2021)
  - Visualize /dev/input/event* kernel events via GL (works on KDE, Gnome, etc.)
<div style="text-align:center">
    <iframe width="280" height="157" src="https://www.youtube.com/embed/Cpn_lILPhEM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
</div>

- [mtview](https://github.com/whot/mtview) (C, 2010-2016)
  - Visualize /dev/input/event* kernel events with GTK

- [mtdiag-qt](https://github.com/bentiss/mtdiag-qt) (C++, 2012-2017)
  - Visualize and diagnose kernel multitouch events


## Blogs

- [who-t](https://who-t.blogspot.com/) Peter Hutterer's tech blog, focused mainly on libinput and related development over the years


## Presentations

[XDC2014: Peter Hutterer - Consolidating the input stacks with libinput](https://www.youtube.com/watch?v=vxhdba4RS8s)
(YouTube) Peter introduces `libinput`
