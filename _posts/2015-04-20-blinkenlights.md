---
layout: post
title: Experiements with LED Strips
---

# {{ page.title }}

(in progress)

I somewhat recently ended up with a little [ARM development board][lpc_dev] for the LPC824---a Cortex-M0 part with a rich peripheral set. Obviously the only logical use for a new dev board is blinking LEDs. In this case, lots of LEDs. One of the fruits of a recent trip to Shenzhen, China was meters and meters of RGB LED strips. They are essentially just a row of RGB LEDs in series-parallel that you can cut at incremental lengths or attach together. I also wanted to play around with some DSP code, so I figured why not make some sort of audio-LED controller.

The end result is a module that detects sound, particularly the kick drum, and flashes RGB LEDs to the beat. It can also be controlled by a tablet running [TouchOSC][touchosc] through a Python-based server, which relays commands over a serial link to the LED controller.

## Hardware

I whipped up a little prototype board with: a microphone with an anti-alias filter; a programmable gain amplifier; and three MOSFETs with associated level shifting. Check out the [schematic (pdf)][pdf_schematic] and the picture below:

![Prototype circuit]({{ site.url }}/images/blinkenlights/prototype_board.jpg)

The first amplifier does a little bit of amplification and incorporates a simple 20dB/decade low pass anti-aliasing filter. The corner frequency is set at a relatively low 5kHz, because most of the 'interesting' content for LED control in music is below this frequency. It also drastically lowers the required sample rate, letting the ADC oversample more. The DC offset is added to set the signal at the midpoint of the ADC's range as well.

The second amplification stage has programmable gain. This allows the microcontroller to adjust the ADC input level dynamically, making the best use of the available dynamic range. Inexpensive 2N7002 FETs are used to switch gain resistors in and out of the circuit. The available gains are:

| Setting | System Gain |
| ------- | ----------- |
|    0b00 |          49 |
|    0b01 |         130 |
|    0b10 |         528 |
|    0b11 |         609 |

Capacitors C2 and C5 are used to set the DC gain of each op amp to 1, preventing the input offset errors from being amplified, particularly the offset of the first stage. This lets cheap amplifiers with poor offset characteristics do the job, taking advantage of the fact there is no interesting DC content in the audio signal.

Automatic gain control has not yet been implemented in software. For now it's operating at the lowest gain of 49, and seems to be working fine. After the gain stages, the signal goes directly into the internal ADC of the LPC824. A small series resistance is included to isolate the output of the amplifier from the ADC input, which should help with stability (although that is hardly a problem in the first place, it doesn't hurt).

## Software

The server-side software and Cortex-M0 firmware are on Github:

[Server][github_server]

[Client][github_client]

Both are currently a huge mess. I'm working on cleaning them up a tad but for a quick project it isn't high on my to do list. The firmware relies heavily on NXP's provided library functions for IO manipulation. Setting up the peripherals was one of the more difficult parts. I realized pretty early on that if the ADC called an interrupt after every conversion, a ton of CPU time would be wasted just copying out ADC values. To get around this, I set up the DMA, triggered on an ADC conversion complete. It alternately fills two buffers with the ADC results. After a buffer is finished, the DMA signals an interrupt which tells the processor new data is available. The DMA controller is quite smart, and takes a big struct of addresses and configuration:

```
__attribute__ (aligned(16)) static DMA_CHDESC_T dmaDescA = {
	.xfercfg = DMA_XFER_CONFIG,
	.source = DMA_ADDR(&(LPC_ADC->DR[INPUT_ADC_CH])),
	.dest = DMA_ADDR(&dmaBuffA[DMA_BUFFER_SIZE-1]),
	.next = DMA_ADDR(&dmaDescB)
};
```

The DMA will use the `.next` pointer to load another configuration, which will start the next transfer, then 'ping-pong' (using the `.next` pointer again) back to this one. With all this setup, the processor doesn't need to interfere at all, other than processing the notification that new data is available. 


[lpc_dev]: http://www.embeddedartists.com/products/lpcxpresso/lpc824_xpr.php
[touchosc]: 
[pdf_schematic]: {{ site.url }}/assets/lpc_led_controller.pdf
[github_server]:
[github_client]:

