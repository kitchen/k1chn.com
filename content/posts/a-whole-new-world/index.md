---
title: "A Whole New World"
date: 2019-05-08T08:44:42-07:00
---

I recently got Winlink up and running using a raspberry pi. It was pretty cool. I also love the fact that this little device is probably as powerful if not more powerful than my old linux server I ran in 2001 when I was in college. And it's super tiny and awesome. I'm probably going to get at least one more to turn into a "home server" of sorts, maybe even a VPN endpoint or something. I dunno. At any rate, I'm enjoying it.

I've been using linux for something like 18 years now, and working with it professionally for most of that time. I'd seen ham radio stuff in kernel configs before but never thought too much about it. However, the winlink project opened a door for me. A door I didn't know existed. This morning I've been looking into how to automate setting up the ax25 bits when I connect my radio to my pi and have been looking at systemd docs and all sorts of stuff.

However, the most eye opening thing was when I started looking through the packages available on Raspbian and accidentally finding so much more than I thought even existed:

```
pi@raspberrypi:~ $ apt-cache search rigctl
gpredict - Satellite tracking program
xdx - DX-cluster tcp/ip client for amateur radio

pi@raspberrypi:~ $ apt-cache search hamlib
cqrlog - Advanced logging program for hamradio operators
gpredict - Satellite tracking program
grig - graphical user interface to the Ham Radio Control Libraries
libhamlib++-dev - Development C++ library to control radio transceivers and receivers
libhamlib-dev - Development library to control radio transceivers and receivers
libhamlib-doc - Documentation for the hamlib radio control library
libhamlib-utils - Utilities to support the hamlib radio control library
libhamlib2 - Run-time library to control radio transceivers and receivers
libhamlib2++c2 - Run-time C++ library to control radio transceivers and receivers
libhamlib2-perl - Run-time perl library to control radio transceivers and receivers
libhamlib2-tcl - Run-time Tcl library to control radio transceivers and receivers
pyqso - logging tool for amateur radio operators
python-libhamlib2 - Run-time Python library to control radio transceivers and receivers
soapysdr-module-audio - Audio device support for SoapySDR (default version)
soapysdr0.5-2-module-audio - Audio device support for SoapySDR
xdx - DX-cluster tcp/ip client for amateur radio
xlog - GTK+ Logging program for Hamradio Operators
xlog-data - data for xlog, a GTK+ Logging program for Hamradio Operators
```

Amazing! There's SO MUCH software just in the officially packaged stuff for Raspbian! Along with the Winlink forms for Pat project, I'm already coming up with so many other project ideas just looking at this list! And I haven't even really started down the road of SDR! Fun!
