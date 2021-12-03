---
layout: post
title:  "Best of “libinput on LWN”"
date:   2021-12-02 22:54:11 -0700
categories: news
---

Below, you'll find a curated collection of the best libinput-related reports from LWN (Linux Weekly News).

<div class="img-centered img-medium" style="float:left">
    <img src="/assets/images/touchpad-stack-simplified.png">
</div>

But first, some background:

One of the most important components of the modern touchpad experience on Linux is the `libinput` library ([source code](https://gitlab.freedesktop.org/libinput/libinput))---a project that was initiated in 2014 by Redhat and quickly grew to dominate input handling for X11 and Wayland based desktop environments (basically, any graphical Linux desktop).

It is currently maintained by Peter Hutterer, and is looking to be maintained by more developers.

In its earliest days, `libinput` started as a rallying cry for sane input expectations in a heterogenous input device world. In addition to each X11 device driver's unique quirks causing inconsistency, users also had to be fairly adept at configuring their system's X11 conf files, since device order mattered and manual settings were plentiful.

`libinput` has largely succeeded in easing and consolidating human device input on Linux. It has also been a key component in the input stack for other more complex input methods such as touchpads.

Throughout the transition (from pre-`libinput` to today's nearly ubiquitous `libinput` Linux world), LWN has summarized reports from various sources including XDC (X.Org Developers Conference) and the primary author's (Peter Hutterer's) blog. Here are the highlights:

<div style="clear:both"></div>

### Sep. 2014: ["Libinput - a common input stack"](https://lwn.net/Articles/612960/)
- Introduces the problem and explains why libinput is needed
- See also [the source](http://who-t.blogspot.com/2014/09/libinput-common-input-stack-for-wayland.html) at Peter Hutterer's blog

### Dec. 2014: ["Pointer acceleration in libinput"](https://lwn.net/Articles/624776/)
- Introduces the concept of a DPI database for mice
- See also [the source](http://who-t.blogspot.com/2014/12/building-a-dpi-database-for-mice.html) at Peter Hutterer's blog

### Nov. 2015: ["Version 1.0 released"](https://lwn.net/Articles/658052/)
- Announced at XDC
- See also [the presentation slides](https://www.x.org/wiki/Events/XDC2015/Program/hutterer_libinput.html)

![slides](/assets/images/2015-11-libinput-slides-sampler.png)

### Apr. 2016: ["Libinput doesn't have a lot of config"](https://lwn.net/Articles/682923/)
- Explains why fewer config options makes maintaining the library easier
- See also [the source](http://who-t.blogspot.com/2016/04/why-libinput-doesnt-have-lot-of-config.html) at Peter Hutterer's blog

### Jul. 2016: ["Libinput is done"](https://lwn.net/Articles/695607/)
- Peter Hutterer explains how the [original goals of libinput have now been achieved](http://who-t.blogspot.com/2016/07/libinput-is-done.html)

### Oct. 2016 ["An update on input"](https://lwn.net/Articles/702998/)
- Presentation at XDC
- See also [the presentation slides](https://www.x.org/wiki/Events/XDC2016/Program/xdc2016_input.html)
 
![slides](/assets/images/2016-10-libinput-slides-sampler.png)

### Oct. 2019 ["An update on the input stack"](https://lwn.net/Articles/801767/)
- Presentation at XDC
- Covers libinput testing utilities and input device testing infrastructure
- See also [the presentation slides](https://xdc2019.x.org/event/5/contributions/327/attachments/429/680/XDC2019.pdf)

![slides](/assets/images/2019-10-libinput-slides-sampler.png)

If you'd like to dive in to hacking on libinput, please see our [resources](/resources) and [docs](/docs) pages.

Happy hacking from the LinuxTouchpad.org team!
