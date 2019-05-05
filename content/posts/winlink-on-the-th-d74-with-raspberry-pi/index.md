---
title: "Winlink on the TH-D74 using a Raspberry Pi"
date: 2019-05-04T17:39:51-07:00
---

I'm a Mac user. If you're a Mac user in the ham world, you're probably as familiar with how difficult it is to find good software for ham radio for the Mac.

With [Portland NET](https://www.portlandoregon.gov/pbem/31667), I'm being encouraged to get [Winlink](https://winlink.org/) up and running if at all possible. Being on a Mac, this is not the easiest proposition. I'm not a fan of running things under WINE or running unsigned binaries or whatever. I'm fine with using a VM and running Linux inside it, but I made a bit of a mistake when I bought this laptop and thought 128GB of storage would be plenty. Pro tip: it's not. But that's ok. Because I have other options.

Fortunately, there's a piece of software that looks actually quite good called [Pat](https://getpat.io/). It's written in [golang](https://golang.org/). It has a nice command line interface (my day job is working on linux servers and has been forever, so command line is life). It has a web based gui. And it says it supports KISS. I have a [Kenwood TH-D74a](https://www.kenwood.com/usa/com/amateur/th-d74a/) which has a KISS TNC built in. Of course, I didn't see the "linux only" part before I started playing around with it. Oops. And it seems the direct serial interface it has both doesn't work with my setup and is very much not supported anymore.

It does seem that someone has gotten this working in the past [using a proxy that Pat can talk to](https://github.com/la5nta/pat/issues/109) via some sort of TCP based protocol and the proxy translates that to KISS/AX25, but it's quite old and I can't seem to get it working. The KISS/AX25 projects it's based on are self-described as "do not use" and the rewritten versions don't seem to be fully done, so it's kind of in a state of limbo. I considered trying to rewrite the proxy itself using a different language that has supported or at least functioning kiss/ax25 libraries, but without really knowing what I'm doing at all, I figure this is probably the wrong route to take at the moment.

Pat's suggestion is to use Linux, which has native support for AX25, the protocol that winlink uses for transmitting packets over the air. Given my limited amount of storage on my laptop, I'm not really able to run a full VM just for that at the moment, so other options are required. One of those options is the ever awesome [Raspberry Pi](https://www.raspberrypi.org/). The latest version, 3B+, has bluetooth, wifi, 4 usb ports, is plenty fast enough for my needs, etc. I picked one up on Amazon for about $35, along with a 32GB microsd card for about $8. Given that it has bluetooth, I'm thinking I should be able to connect it to my radio wirelessly, which allows me to stuff the rpi in the back of my tv cabinet somewhere after I get it up and running and not have to think about it too much anymore.

After hooking it up to my tv and a usb keyboard, I enabled ssh and hooked it up to wifi and went back to my couch with my laptop instead of sitting 3 feet in front of my TV on my ottoman. And then of course I couldn't get the pi and the radio to see each other, no matter which one I set to discoverable and which one I set to scan. I was able to get the pi to see the radio, but establishing a connection wasn't working. Before I gave up on the bluetooth route, I searched for a message I saw `Failed to connect: org.bluez.Error.NotAvailable` and found [a forum post](https://www.raspberrypi.org/forums/viewtopic.php?p=947185#p947185) which talks about what I suspected that error meant, that the radio and the pi couldn't establish a common profile, in this case the serial port profile. Sadly, after much mucking about with it, I was still unable to get it working, so I bailed on that and am just going to use USB for now.

USB worked just fine out of the box. On the TH-D74a you need to set the KISS mode to USB (menu 983) and the USB Function to COM+AF/IF Output (menu 980) and plug it in. A lot of other types of radios have a USB adapter you can buy which is almost always just a USB to serial adapter with an FTDI chip and a the correct wiring to get it to talk to the radio (either UART or I2C). RT systems sells these as well, except they are "branded" and don't work with standard FTDI drivers and, at least on the mac, the drivers they provide don't work because they aren't signed. I don't reccomend any RT systems cables to anyone. Anywho. The TH-D74a basically just has one built in and it's FTDI compatible!

If I `screen /dev/ttyACM0 9600` and type `ID` I get back `ID TH-D74`, so I know it's working! Hooray!

Ok, so on to trying to set up Pat. First, to install it. Which has some dependencies of its own. First, I need golang to be installed. I used `apt-get install golang` on the pi to grab go, which is version 1.7. Then I needed `git` because `go get` uses git under the hood, so I installed that. I set my `GOPATH` environment variable to `$HOME/go` and once all of that was done, `go get github.com/la5nta/pat` finally worked correctly.

From there it was following along on the [AX.25 Linux instructions](https://github.com/la5nta/pat/wiki/AX25-Linux) which were easy to follow along with. When I got to the `axlisten` part, I got a bit sad, as the local frequency I'm trying to use isn't very active, so I don't know if it's working or not. But onward I press!

I got to configuring pat and setting up the ax25 interface and such and all seemed well. Then I got this error: `2019/05/05 03:22:29 Unable to establish connection to remote: AX.25 support not included in this build`. I thought to myself... WHAT?! Isn't that the whole point of this? Perhaps it's a raspberry pi issue, maybe the code doesn't work on ARM or something. No. Not so much. According to [some other docs](https://github.com/la5nta/pat/wiki/Building-from-source#build-tags) I needed to add build tags to get ax25 support. Which also required me to install `libax25-dev`.

Connect!

```
pi@raspberrypi:~ $ pat connect ax25://wl2k/KF7LJH-10
2019/05/05 03:27:30 Connecting to KF7LJH-10 (ax25)...
```

Nothing happened.

I forgot to mention that in the middle here I had to relocate the pi. I'd gotten a couple of messages about under voltage and thought maybe I needed a beefier USB power supply, so I shut the pi down and moved the whole setup to the kitchen. When I got logged back into the pi everything seemed fine, but I couldn't use screen to connect anymore, for some reason. Strange. So I think that's what's happening right now. I'm going to reboot everything and see if that fixes it. "Have you tried turning it off and on again?" :)

Success! At least, success using `screen` to connect and send an ID command. Let's try the rest of it again. Nope. I didn't hear anything from my radio. I didn't see any activity indication, and no output. When I finally gave up and hit ctrl-c, I got `2019/05/05 03:36:42 Unable to establish connection to remote: Dial timeout` as output, so something isn't quite right.

Google to the rescue! According to [The Fine Manual™](https://www.manualslib.com/manual/1170023/Kenwood-Th-D74a.html?page=82), I need to set the TNC into KISS mode. Oops.

Ok, so I got that set up. Then I was trying some winlink nodes I found on [winlink.org](https://www.winlink.org/RMSChannels) but not really having any luck. A quick google search for "portland packet repeater" turned up the [Portland Amateur Radio Club repeater list](http://www.w7lt.org/repeaters), which mentions a packet repeater that had been replaced with a winlink node. 144.910 / W7LT-10. So I tuned my radio to that frequency and tried to connect. Suddenly I was deafened by my radio. I'd turned the volume way up trying to hear if there was anything, and hadn't turned it down. So when that winlink node started talking to me I heard it. It was glorious.

```
pi@raspberrypi:~ $ pat connect ax25://wl2k/W7LT-10
2019/05/05 04:59:42 Connecting to W7LT-10 (ax25)...
2019/05/05 04:59:53 Connected to W7LT-10 (AX.25)
WELCOME TO PORTLAND AMATEUR RADIO CLUB'S WINLINK PACKET GATEWAY - CN85RK
[WL2K-5.0-B2FWIHJM$]
CMS via W7LT >
>FF
;PM: K1CHN K1CHN71CYAUW 711 SERVICE@winlink.org Your New Winlink Account
FC EM K1CHN71CYAUW 1177 711 0
F> 59
1 proposal(s) received
Accepting K1CHN71CYAUW
Receiving [Your New Winlink Account] [offset 0]
Your New Winlink Account: 100%
>FF
FQ
2019/05/05 05:00:51 Disconnected.
```

So, it worked? There was lots of back and forth between my radio and the remote system, but judging by the output, everything worked!

So, what now? Well, Pat has a web interface. So I fired it up: `pat http -a 0.0.0.0:8080` (the 0.0.0.0 makes it listen on all interfaces, not just localhost, and since I'm connecting from my laptop to the raspberry pi I need to do this). Opening my browser to the web interface, and I'm greeted with what looks like a webmail view!

{{< pureimg src="webface-index.png" caption="Pat web interface" >}}

Success!

And it looks like I have mail!

{{< pureimg src="your-new-winlink-account.png" caption="Your new winlink account message" >}}

Great!

Then I updated my password and sent myself a test message!

{{< pureimg src="compose-test-message.png" caption="test message composition" >}}

A couple of round trips connecting to the winlink system later and I have a new piece of mail!

{{< pureimg src="test-message-received.png" caption="receiving test message" >}}

Awesome!

Now, since AE7XP has been one of the folks hounding me about getting winlink going, I dropped him a message just to see if it's all working. As far as I know winlink is also able to send and receive email from the internet, so one final test:

{{< pureimg src="internet-compose.png" caption="composing message to internet" >}}

And a short time later...

{{< pureimg src="winlink-to-gmail.png" caption="receiving message to internet" >}}

Excellent! Now I have something to talk about on the NET net tomorrow night! :)
