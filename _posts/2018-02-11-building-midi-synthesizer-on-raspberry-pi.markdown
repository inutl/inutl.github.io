---
layout: post
title:  "Building a MIDI synthesizer on the Raspberry Pi"
date:   2018-02-11 20:06:52 -0200
categories: posts
---
I got a [Samson Graphite 25][graphite-25] MIDI controller for myself and have been
playing with it on Windows using the [OB-Xd VST][obxd-home] hosted on [SAVIHost][savihost]
and it's been quite fun! But at some point I know that I'll want to use it during band
rehearsals and it got me wondering how will I manage to do so.

# Motivation

The simplest way is just setup a Windows notebook with something like my current setup,
maybe automate it a little bit more, allowing for quicker changes of presets and stuff
like that.

That would have been perfectly fine, but I've got a Raspberry Pi 2 for quite some time and it
has not seen any use until now, so I thought: "Hey, maybe I finally found something
that's a good use for this thing!".

So I started researching and have been able to hack something that resembles a MIDI synthesizer.
Just to be clear, I want to connect my MIDI controller to the Raspberry Pi, and, without use
of any screen whatsoever, be able to perform some songs on it.

Disclaimer: I don't have much experience dealing with MIDI, VSTs and all this stuff. If I end up doing
something that's complete unusual and stupid, I'm sorry, but that's how I managed to accomplish
what I wanted.

Also, note that this is more of a guideline than a proper guide.

# Raspbian

For this project, I'm using the [Raspbian][raspbian] `stretch` distribution.

I don't want to have to plug a keyboard, mouse and display into the Raspberry Pi to use it,
so I set it up with SSH access. Just follow this [article][pi-ssh] and everything should be fine.

Even though I wasn't smart enought for this, it might be useful to do the [WiFi setup][pi-wifi]
before hand as well. I just connected the first time using an Ethernet cable and configured it
directly, but it is nice to know that you can do it even if you don't have access to a physical
cable.

Connect to your Pi with `ssh -XC <user>@<pi-address>`, the `-XC` option will enable graphical
output through SSH (`-X`) and compress the data when transmitting it (`-C`), making it easier
to set everything up.

To improve the sound quality on the Pi, you need to [update your firmware][pi-audio].

# JACK

When using Linux and trying to do something with audio, you end up having to use JACK. Fortunately,
it seems to work fine on the Pi, so just installing it (`sudo apt-get install qjackctl`) and
running (`qjackctl &`) doesn't gave me any problems.

Even though it is far from ideal, I am running it with 1024 frames/period and 4 periods/buffer, giving
me 85.3 ms of latency. Low enough for testing purposes, but needs to be optimized. It looks like it's
already stressing the system, because I am getting some buffer underruns even with the latency this
high. I will look into it in the future.

# OB-Xd

I enjoy this plugin a lot, so I wanted to build it for the Pi. I didn't have much trouble with it
thanks to the [DISTHRO project][disthro].

Just cloned their repository, installed its pre-requisites, set the `LINUX_EMBED` environment
variable and it compiled a lot of its plugins. Had some issues compiling the VST version, but the
LV2 version compiled just fine, which was enough for my purposes.

Here's a [link][pi-obxd-lv2] to a compiled version of it. I have no idea if it will work with your
environment. Use at your own risk.

# Carla

I wanted to use Carla to host the OB-Xd plugin and you can find a compiled version of it on the
[Autostatic][autostatic] repository. It gave me some errors since I was using `stretch` instead of
`wheezy` or `jessie` but it seemed to work just fine.

But the version of Carla there is a little bit outdated and I wanted to be able to use it without
the GUI, so I looked into compiling it also. It wasn't that difficult, but I don't think the version
I compiled is feature complete, since it is missing some dependencies.

Anyways, just clone its [repository][carla-github], take a look at the `INSTALL.md` file, install
all dependencies and try to build it.

It will give you an error because of some compiler flags, just patch it and it will compile.


```
diff --git source/Makefile.mk source/Makefile.mk
index 7b06e2df..9af26485 100644
--- source/Makefile.mk
+++ source/Makefile.mk
@@ -101,7 +101,7 @@ endif
 # Set build and link flags

 BASE_FLAGS = -Wall -Wextra -pipe -DBUILDING_CARLA -DREAL_BUILD -MD -MP
-BASE_OPTS  = -O3 -ffast-math -mtune=generic -msse -msse2 -mfpmath=sse -fdata-sections -ffunction-sections
+BASE_OPTS  = -O3 -ffast-math -mtune=native -fdata-sections -ffunction-sections

 ifeq ($(MACOS),true)
 # MacOS linker flags
```

Note: I don't know if there are better optimized flags. I think there are, but I didn't look too much into it.

With this updated version, now I can run it without the GUI with `carla -n <carla_patch>`.

# Conclusion

After gathering all these components now I could play some sounds on the Pi, yay! Just had to run everything
I had until now together with `a2jmidid` to expose my MIDI controller to JACK (more info [here][a2jmidid]),
connect them all and I could hear sounds coming out of its headphone port. Success!

Now I could set it up to boot with everything connected using [aj-snapshot][aj-snapshot-sf] and make it a
standalone synth, which is what I set up to do!

This has been more of a proof-of-concept than the finished project, but it's enough for me to keep
investigating. Some stuff that I am currently looking into: porting my `.fxp`/`.fxb` presets from the
Windows OB-Xd into LV2 presets, automating stuff from the MIDI controller (changing presets, playing
samples / chords instead of a single note, etc.) and optimizing the audio latency.


[graphite-25]: http://www.samsontech.com/samson/products/usb-midi/keyboard-controllers/graphite25/
[obxd-home]: https://obxd.wordpress.com
[savihost]: http://www.hermannseib.com/english/savihost.htm
[raspbian]: https://www.raspberrypi.org/downloads/raspbian/
[pi-ssh]: https://hackernoon.com/raspberry-pi-headless-install-462ccabd75d0
[pi-wifi]: https://core-electronics.com.au/tutorials/raspberry-pi-zerow-headless-wifi-setup.html
[pi-audio]: https://www.raspberrypi.org/forums/viewtopic.php?f=29&t=195178
[disthro]: https://github.com/DISTRHO/DISTRHO-Ports
[pi-obxd-lv2]: {{ "/downloads/Obxd.lv2.zip" | absolute_url }}
[autostatic]: http://rpi.autostatic.com/
[carla-github]: https://github.com/falkTX/Carla
[a2jmidid]: http://manual.ardour.org/setting-up-your-system/setting-up-midi/midi-on-linux/
[aj-snapshot-sf]: https://sourceforge.net/projects/aj-snapshot/