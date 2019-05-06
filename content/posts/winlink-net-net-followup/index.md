---
title: "Winlink NET net followup"
date: 2019-05-05T21:58:57-07:00

---

Tonight on the NET net I talked about my [experiences with setting up Winlink]({{< ref "winlink-on-the-th-d74-with-raspberry-pi">}}). There were some great questions and discussion items, so I thought I'd try to capture some of that here for future reference.

There were a couple of people who asked for clarification on how this is all actually connected together. So here's a picture.

{{< pureimg src="th-d74-raspberry-pi.jpg" caption="TH-D74a and Raspberry Pi wired up" >}}

On the right hand side is the [raspberry pi](https://www.raspberrypi.org/) itself. I don't have a project box or anything for it. I might make something at some point, but probably not. It works fine without a case. On the left is my [Kenwood TH-D74a](https://www.kenwood.com/usa/com/amateur/th-d74a/). Between the Raspberry Pi and the radio is a standard microusb cable. Then I have the raspberry pi connected to a USB power supply using microusb. In the pi itself there's a 32GB microsd card with [raspbian](https://www.raspberrypi.org/downloads/raspbian/) on it. I may swap that out at some point for a different distro, but everything I've needed so far has worked without any trouble, so I'll probably keep it that for now.

On the display you see 2 frequencies. One is band A, and one is band B. This radio has the ability to receive on 2 frequencies at once. When it's in KISS mode, there isn't really much reason for the B band to be selected, because it's being controlled automatically, but I wanted to show the display for the B band. I can turn the A band off entirely, but then I get a GPS display and, ugh. This is fine :) The red bar and STA along with the red light on the top indicate that it's currently transmitting. The `W7LT-10 winlink` on the display is just the name I gave that memory, the radio itself doesn't need to know about the destination node.

I'd also mentioned that you don't need a $500 radio to do winlink, there are cheaper options including options that work even with a little Baofeng radio. Getting this working with a standard FM voice radio is a goal of mine, if for no other reason than to try to make it more accessible to folks. Not everyone wants to drop $500 on a handheld. Kevin (KU0L) mentioned that there are youtube videos that show how to get it all up and running and sent [a link to one he recommends](https://www.youtube.com/watch?v=GjYLhDKrNeE).

My next step is going to be heading down the path of using [Dire Wolf](https://github.com/wb2osz/direwolf) as a software TNC, using Pat with that, and connecting the radio (probably my [Yaesu FT-60](https://www.yaesu.com/indexVS.cfm?cmd=DisplayProducts&encProdID=6EC43B29CEF0EC2B4E19BB7371688B7F) to start with) using audio cables from the raspberry pi's sound output.

Another option is to use a [mobilinkd](http://www.mobilinkd.com/) TNC as suggested by Ian (N7IMB). It's a little battery powered TNC with bluetooth connectivity that you can basically duct tape to the back of your handheld and turn it into a TNC enabled radio.  Additionally, since it has bluetooth, you can use software like [APRSdroid](https://aprsdroid.org/) for doing APRS things. In fact, a friend of mine did this very thing at the Southern California Linux Expo a couple of years ago and was walking around the conference just beaconing out his position for grins. It doesn't seem like there's any android software for winlink, but I'd love to be wrong on that front! I wasn't successful getting my raspberry pi and my radio connected with bluetooth, but perhaps mobilinkd is easier to connect? I'm not sure. I am fairly certain I was experiencing a [layer 8 problem](https://en.wikipedia.org/wiki/Layer_8) with my bluetooth attempts, but maybe the mobilinkd is easier out of the box.

Ian also followed up in an email with me some other options:

> Also, there is TNC-Pi, which is Raspberry Pi shield TNC. The TNC-Pi 2 is $40 and 3 that does 9600 baud is $65. It is based on TNC-X.
https://tnc-x.com/TNCPi.htm

> While Googling, I found PiGate. It looks like distribution for Winlink on Raspberry Pi with TNC-X.
http://www.pigate.net/

> This might inspire me to actually setup Raspberry Pi.

I sincerely hope it does!

Michael (AE7XP) had also previously let me know about the TNC-X. To be honest, I think if I have a raspberry pi in the mix already, I might as well use a software TNC. But maybe there are good reasons to go with a hardware TNC. I'll have to do some experimenting on it!

And one final thing. Ralph (AG7FE) wrote to ask me about Pat and Winlink forms. I was not aware that such things existed, but it turns out that there are some html forms and templates which come with Winlink Express which allow for standardized input and display of various common forms like ICS-213 forms and whatnot. Apparently this is a hard requirement for ARES operation. I [managed to find the templates and such](https://winlink.org/content/how_manually_update_standard_templates_version_10910) and it looks like there are 3 things here: first being a form to actually fill in. Second, the output of that form, sometimes it's an XML document, sometimes it's JSON, sometimes it's CSV. And the third thing being a way to turn that output back into a human friendly format. Pat doesn't currently have any of these things, but the API it exposes is bog simple, so creating something like that wouldn't be terribly difficult if it had to be done outside of Pat. I've reached out to the developer to see if there would be any interest in having native support for these forms directly in Pat. The only real difficulty I see with making this happen is detecting what type of form the file actually is to be able to choose which template to use for rendering. There doesn't seem to be any standardization at all on the actual transport format. Anywho, that's its own rabbit hole, methinks!

I do plan to do another followup post about getting things going with a normal radio, but I wanted to get this out here tonight!

Thanks to everyone who participated in the net this evening and brought suggestions and info and questions and all of that! I hope I didn't miss anything or mis-credit anyone, if I did, let me know, I'll fix it!
