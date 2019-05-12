---
title: Winlink and the TH-D74, now with more Bluetooth!
date: 2019-05-11T20:35:14-0700

---

As stated in my [post about getting winlink going with a Raspberry Pi and my TH-D74a]({{< ref "winlink-on-the-th-d74-with-raspberry-pi">}}), I wasn't having success getting my radio connected to the Pi using bluetooth.

Well, tonight, that's changed. I managed to do it. It's glorious.

{{< pureimg src="look-ma-no-wires.jpg" caption="TH-D74a bluetooth KISSing" >}}

The first thing I figured out that I was doing wrong the other day, was in `bluetoothctl` I was trying to `connect` to my radio. What I needed to do was `pair`. Then I ran into all sorts of other confusing things that I'm still not 100% sure on, but I'm going to try to explain them here.

Tools you'll need:

* `bluetoothctl`
* `sdptool`
* `rfcomm`
* `kissattach`
* `kissparms`

And of course, your text editor of choice.

In this case, these instructions are going to be specifically for Raspbian Stretch, but they might just work with other things? I'm also going to try to link to where I found instructions or assistance that helped me along the way, but I had so many tabs open and searching for stuff it's hard to remember where I first saw things.

Anywho, here goes!

# Radio setup

The first thing you'll need to do is configure your radio for KISS over bluetooth.

* Turn Bluetooth on (menu 930)
* Turn KISS to Bluetooth (menu 983)
* Turn other Bluetooth things off (menu 98*) -- This may not be strictly necessary, but I did it as a way to isolate just KISS in case there were some strange conflicts happening. I'd love to hear success/failure if other things are enabled!

And that should be that for radio configuration.

# Raspberry Pi setup

First you need to modify the bluetooth service to enable the Serial Port Profile. [via](https://www.raspberrypi.org/forums/viewtopic.php?p=947185#p947185)

* `sudo $EDITOR /etc/systemd/system/dbus-org.bluez.service`
* At the end of the `StartCommand=` line, add ` -C` -- This seems to enable `sdptool` to actually work, apparently [it has been deprecated in Bluez 5](https://raspberrypi.stackexchange.com/questions/41776/failed-to-connect-to-sdp-server-on-ffffff000000-no-such-file-or-directory)?
* Add an `ExecStartPost=` line to enable the SP profile: `ExecStartPost=/usr/bin/sdptool add SP`

Save the file and reboot. You might only need to restart bluetoothd. YMMV.

# Pair the radio with the Pi

Now you need to pair the radio with the Pi. This is the bit where I messed up before, hopefully these instructions will help you not make the same mistakes I did!

* Turn on pairing mode on the radio (menu 934)
* On the Pi: `bluetoothctl`
* `agent on` (within the bluetoothctl shell) -- This seems to be [required to enable pairing](https://askubuntu.com/questions/763939/bluetoothctl-what-is-a-bluetooth-agent).
* `scan on` (within the bluetoothctl shell) -- This scans for the radio.

Note that if your pairing mode on your radio has timed out you may need to start it over. It's no big deal, and once scan is on your radio will eventually show up.

The radio should show up in the list:
```
[bluetooth]# scan on
Discovery started
[CHG] Controller B8:27:EB:A8:5A:4F Discovering: yes
[...]
[CHG] Device 4C:2C:65:EF:94:74 RSSI: -74
[CHG] Device 59:C6:33:9F:30:63 RSSI: -77
[CHG] Device 5B:1F:D6:7C:AE:6D RSSI: -91
[NEW] Device 56:D4:9A:25:4E:D8 56-D4-9A-25-4E-D8
[NEW] Device 24:71:89:A2:4A:68 TH-D74
```

The `24:71:89:A2:4A:68` there is the mac address of the device, and also the ID you'll need going forward. Note that it'll be different for your radio as MAC addresses are meant to be mostly globally unique.

Then you `pair 24:71:89:A2:4A:68` (use your radio's mac address, not mine ;-) and it will show a code on the radio and ask you if that's correct. Don't do like I did and type in the code, that doesn't work. Just confirm whether or not they match!

```
[bluetooth]# pair 24:71:89:A2:4A:68
Attempting to pair with 24:71:89:A2:4A:68
[CHG] Device 24:71:89:A2:4A:68 Connected: yes
Request confirmation
[agent] Confirm passkey 345344 (yes/no):
[agent] Confirm passkey 345344 (yes/no): no
Failed to pair: org.bluez.Error.AuthenticationFailed
[CHG] Device 24:71:89:A2:4A:68 Connected: no
[bluetooth]# pair 24:71:89:A2:4A:68
Attempting to pair with 24:71:89:A2:4A:68
[CHG] Device 24:71:89:A2:4A:68 Connected: yes
Request confirmation
[agent] Confirm passkey 187740 (yes/no): yes
[CHG] Device 24:71:89:A2:4A:68 UUIDs: 00001101-0000-1000-8000-00805f9b34fb
[CHG] Device 24:71:89:A2:4A:68 UUIDs: 00001112-0000-1000-8000-00805f9b34fb
[CHG] Device 24:71:89:A2:4A:68 ServicesResolved: yes
[CHG] Device 24:71:89:A2:4A:68 Paired: yes
Pairing successful
[CHG] Device 24:71:89:A2:4A:68 ServicesResolved: no
[CHG] Device 24:71:89:A2:4A:68 Connected: no
```

And finally, tell the bluetooth system to trust the radio so it will automatically connect when available:

```
trust 24:71:89:A2:4A:68
```


Now your radio should be paired to the Raspberry Pi.

Note that if you've previously paired your radio and Pi you might run into `Failed to pair: org.bluez.Error.AuthenticationFailed`. This is because your radio still thinks it's paired to the Pi, even though the Pi doesn't think so anymore. Go into the connect menu (menu 931), highlight the Pi, and then press Menu. This will bring up a submenu where you can "clear" the device, which makes the radio forget about it. Then try the above steps again! [via TH-D74a manual](https://www.manualslib.com/manual/1170023/Kenwood-Th-D74a.html?page=99)

# rfcomm

Ok, so this is where I got *really* confused. The instructions I've found online say to do this:

```
rfcomm connect 0 24:71:89:A2:4A:68 2
```

What does any of that even mean?

So, my understanding of it is that `rfcomm` is what actually goes about creating the `/dev/rfcommN` devices that you can then connect to as serial port devices. The `0` in the above command is which one you'd like to configure. As far as I can tell, this is arbitrary, just needs to not already be in use by another `rfcomm`. The `24:71:89:A2:4A:68` is, of course, the mac address of the radio. And the `2` is the channel. I'm still pretty foggy on what the channel is, but I know that, [at least on my radio, it needs to be 2](https://lieselotte-sky.tumblr.com/post/164194743525/kenwood-th-d74-and-linux-w-bluetooth-packet). 1 "works", but it doesn't actually work. 2, however, works. YMMV. And I'd love to know more about how to even discover what this should be.

This command, if it works will stay running in the foreground:

```
root@raspberrypi:~# rfcomm connect 0 24:71:89:A2:4A:68 2
Connected /dev/rfcomm0 to 24:71:89:A2:4A:68 on channel 2
Press CTRL-C for hangup
```

Once you've gone through all of this and have it up and running, you can use `rfcomm bind` instead of `rfcomm connect` and it will launch into the background and connecting to the `/dev/rfcomm0` device will automatically bring up the connection to the radio (i.e. `kissattach`).

# axports file

The rest of this pretty much follows [Pat's excellent documentation for setting up ax25](https://github.com/la5nta/pat/wiki/AX25-Linux), but I'll reproduce it here.

In the `/etc/ax25/axports` file, add a line like:

```
wl2k MYCALL 9600 255 7 Winlink
```

According to [`axports(5)`](https://manpages.ubuntu.com/manpages/precise/en/man5/axports.5.html) this means:

* `wl2k` -- the unique identifier of the port. You'll use this in the pat config file.
* `MYCALL` -- replace this with your callsign.
* `9600` -- the speed of the interface. 9600 seems to be fine. I feel like it could be faster, maybe 115200, but it doesn't really matter as you can't actually send that fast over the air anyways.
* `paclen` -- the default maximum packet size. This value comes straight out of Pat's docs.
* `7` -- the default window size. This value also comes straight out of Pat's docs.
* `Winlink` -- a description of the interface. Doesn't need to be Winlink. It can be literally anything.

# kissattach / kissparms

Unlike in my previous post, we won't be using `/dev/ttyACM0` this time, since that's the USB version of this. We'll be using the device `rfcomm` created above: `/dev/rfcomm0` (if your first argument to `rfcomm` was not 0, use that number instead). The `kissparms` command will be the same.

```
sudo kissattach /dev/rfcomm0 wl2k
sudo kissparms -p wl2k -t 300 -l 10 -s 12 -r 80 -f n
```

And then you should be able to use Pat like normal!

# slow first connection

Note that I've found the first connection to take a bit of extra time, which has many times led me to believe it wasn't working correctly. Looking in `/var/log/syslog` I see some things:

```
May 11 22:03:12 raspberrypi pat[1012]: 2019/05/11 22:03:12 Connecting to W7LT-10 (ax25)...
May 11 22:03:12 raspberrypi kernel: [ 1923.347019] mkiss: ax0: Trying crc-smack
May 11 22:03:22 raspberrypi kernel: [ 1933.918389] mkiss: ax0: Trying crc-flexnet
May 11 22:03:44 raspberrypi pat[1012]: 2019/05/11 22:03:44 Connected to W7LT-10 (AX.25)
```

(I'm running Pat under systemd which is why the logs are showing up in syslog)

Note that it takes about 30 seconds to make that first connection. Subsequent connections are much faster, I'm assuming the KISS/AX.25 bits have figured out how to talk to the radio and remember that for later:

```
May 11 22:05:48 raspberrypi pat[1012]: 2019/05/11 22:05:48 Connecting to W7LT-10 (ax25)...
May 11 22:05:50 raspberrypi pat[1012]: 2019/05/11 22:05:50 Connected to W7LT-10 (AX.25)
May 11 22:05:52 raspberrypi pat[1012]: WELCOME TO PORTLAND AMATEUR RADIO CLUB'S WINLINK PACKET GATEWAY - CN85RK
```

[`kissparms(8)`](https://manpages.ubuntu.com/manpages/xenial/man8/kissparms.8.html) suggests that you can specify the crc mode using it, but I'm not sure what it need to be. The default seems to be `auto` and that matches what I'm seeing the radio actually doing.

# tearing down

This is all rather finicky to set up, but it's also pretty finicky to tear down. And you need to tear it down or things will be in weird states when you try to set it up again the next time.

* `kissattach` launches a background process and keeps the connection to the radio active. Kill the process to exit it.
* `rfcomm` stays in the foreground if using `connect`. Ctrl-c to exit it out. If you've used `bind` instead, you'll need to `rfcomm release 0` to release the device.

This can be done in really any order. `rfcomm release 0` will error if `kissattach` is still running, `rfcomm connect` probably won't exit until `kissattach` is killed, etc. Also, if you power off the radio or get out of range, you can still clean it all up. And you'll want to do this or the next time you'll get some `device already in use` or whatever errors.

# what's next?

So, what kinda set me down this road today is I was playing around with making it so when I plug my radio in via usb, kissattach comes up, kissparms runs, and it's ready to roll. When I unplug the radio, it tears everything down. I actually got the tearing down bits working, but I had to manually start the systemd service that launched the `kissattach` and such when plugging the radio in. Still better than doing it all manually every time. I'd still like to complete that work, if for no other reason than it provides a learning opportunity for how to use systemd. But I'd also like to do this for the bluetooth side of things. That would mean I can grab my laptop, turn my radio on, open the pat web interface, check my winlink email, and tear it all down without logging into the pi at all. That would be *perfect*. So, I'm going to play around with trying to do that. If/when I am successful, I will of course post about it here :)

If you've made it this far, thanks for reading! If this helped you, I'd love to hear about it! If you're having trouble, I'd definitely love to hear about it and try to get you working!
