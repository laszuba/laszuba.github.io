---
layout: post
title: RF Power Meter
---

# {{ page.title }}

In testing the SDR filter, I determined an RF power meter would work well with my signal generator to make a poor-man's network analyzer. The typical HP offerings are quite precise and resultantly very expensive. After stumbling across Analog Devices logarithmic amplifiers, it figured building one would be not too difficult.

The design is based around an AD8307 logarithmic amplifier, alongside a simple linear power supply and an op-amp used for output amplification and conditioning. It is flat from DC - 500 MHz, then suffers roll off in the amplifier. The output is DC that can be easily converted into the equivalent input, in dBm. Testing used an HP signal generator to check flatness over frequency and linearity.

Everything is assembled in a case made of copper clad scraps, which does a fantastic job at preventing interference and noise. Power is provided by a 9 V battery to prevent noise from coupling in on the supply, and the output passes through a feedthrough capacitor for the same reason.

![Overview]({{ site.url }}/images/pwr_meter/pwr-meter-full1.jpg)

The circuitry is built on a piece of SMD prototype board, that fits SOIC ICs very nicely. It also has a solid ground plane on the underside and an array of unplated vias. This makes for quite robust RF construction.

![Closeup of the circuitry]({{ site.url }}/images/pwr_meter/pwr-meter-detail.jpg)
