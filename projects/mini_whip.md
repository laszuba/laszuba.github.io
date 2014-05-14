---
layout: post
title: Mini Whip Antenna
---

# {{ page.title }}

I currently have two different antennas for testing. A 20 m band dipole at home in Toronto, and a small, active [PA0RDT][miniwhip] mini-whip that travels to wherever I am currently living.

The mini-whip is an active antenna, originally designed for long wave and medium wave reception, but that works well all the way up through the shortwave bands. Mine is built in a piece of PVC pipe. A couple part substitutions had to be made, as one of the transistors is a TO39 metal can device (2N5109) that is challenging to find. It was replaced by a SOT223 package device with similar, if not better performance.

![Mini-whip in its plastic pipe]({{ site.url }}/images/antennas/mini-whip-full.jpg)

![Detail]({{ site.url }}/images/antennas/mini-whip-detail.jpg)

The antenna is powered by +12V injected through the coax by a small power supply module (a few filter capacitors and an inductor). It is built in a small box constructed from copperclad scraps.

![Power supply]({{ site.url }}/images/antennas/mini-whip-pwr.jpg)

## Advantages and Disadvantages

The biggest disadvantage to using an active antenna is the inability to transmit with it.

A benefit and drawback of the mini-whip is the incredibly wide usable bandwidth (0-30 MHz). This means only one antenna will receive almost any signal the current version of SDR is capable of processing, and still provide a meaningful output. Unfortunately, the mini-whip can easily overload the receiver's input, causing intermodulation distortion. The bandpass filter described earlier will take out most of this problem, but it has a small but significant insertion loss, and only covers one band.

The non-directionality of the antenna can also be an advantage or disadvantage. It is excellent when you do not know from which direction the signal is coming, but it also means it picks up all signals equally. In Toronto, a dipole can be oriented such that one of the nulls at the ends of the wires is facing the downtown. This greatly reduces the amount of AM/FM radio interference. Using the mini-whip, all the broadcast interference gets through.

The biggest advantage of the mini-whip is it is tiny and very easy to set up. Living away from home for school, in a small apartment, it is almost impossible to set up an antenna. The mini-whip can be mounted very simply on a balcony railing with recieve performance very similar to a dipole.

[miniwhip]: "http://www.radiopassioni.it/pdf/pa0rdt-Mini-Whip.PDF"
