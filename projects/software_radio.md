---
layout: post
title: Software Defined Radio
---

# {{ page.title }}

## Introduction

Lately I have been very interested in RF. I decided the perfect project to delve into radio was a software defined radio receiver (SDR). I was inspired by Jeri Ellsworth's [videos][jeri_vids] on the topic.

From Jeri's videos, I discovered Gerald Youngblood's seminal [articles][qex] and used these as a reference during my development. During development, I have been using [HDSDR][hdsdr] as a quick and easy way to demodulate input signals, as well as display a spectrum waterfall. I have also been experimenting with [GNU Radio][gnuradio], eventually hoping to write drivers for my own custom hardware, to run on the Raspberry Pi. The current FPGA code is available on [GitHub][gitsdr].

The end goal of the project is to integrate a 130 Msps ADC (LTC2208), FPGA (Altera Cyclone IV), and Raspberry Pi into an integrated radio receiver, running GNU radio. The FPGA will downsample the signal, create the waterfall display, and pipe the data to the Pi through 10/100 Ethernet. The Pi will present the pre-processed waterfall to the user, as well as demodulate a narrower band signal (probably around 2 MHz bandwidth). Keeping the demodulation in the Pi will allow huge amounts of flexibility, but will take a significant amount of processing power. The Pi cannot possibly be presented with the full 60 MHz baseband, so this is where the FPGA comes in. This is a fairly lofty goal, but I am very excited to work on it. I expect it to be quite ongoing.

## First Version

The first version of the SDR is a general coverage receiver, covering 100 kHz - ~50 MHz, using a computer sound card to digitize the baseband. It was very exciting to see the first radio signals come through! First AM radio, then shortwave SSB voice, and then CW (Morse Code). Watching the waterfall display in HDSDR is mesmerizing, and it gives one a great deal of insight into the radio spectrum.

It is built around a quadrature sampling mixer, using a 40XX analog switch, and LM324 op amp, built on proto-board. The clock source is a DE0 Nano FPGA development board, using the on board PLL.

### Problems and Improvements

On this version, the tuning frequency was hard coded, so the FPGA needs to be re-flashed to change the tuning. This was obviously less than ideal. Unfortunately, the performance left a lot to be desired as the noise performance of the op amp is very very poor. This version is just a proof of concept, to try and understand the hardware and how the sampling detector works. No real measurements were performed, as I did not have any of the required test equipment at the time. The hardware later failed, and I decided because of the poor performance it was not worth fixing. Work on an improved version began.

## Second Version

The second version of the SDR is again built around a quadrature sampling detector, with a similar coverage, but using a higher performance op amp (LT6231) and analog switch (ADG712). Tuning is accomplished using a rotary encoder, which dynamically adjusts the Cyclone IV's PLL.

![The complete SDR set up (without computer)]({{ site.url }}/images/software_radio/sdr-v2-full.jpg)

### Test Results
During the development of this version, I acquired a frequency counter and RF signal generator. This allowed me to perform further tests on the receiver.

#### Carrier Measurement
The first test was to inject a CW signal into the receiver. This checked basic functionality and distortion performance.

### Problems and Improvements

#### Tuning

The tuning frequency steps are quite big (~0.5 MHz). This makes it tough to use in the shortwave band, as it is very easy to skip over the frequency you are interested in. The solution to this is to use an external PLL with better performance, or to implement an NCO/DDS oscillator. Both of these methods are fairly complicated and difficult to implement. I may experiment with different types of all digital DDS or PLL solutions, but I am not yet sure of the direction I want to take. I may just wait for the direct sampling version and forget about the tuning altogether.

#### Overloading and Gain Compression

The presence of strong AM and FM broadcast stations has a tendency to overload the input of the receiver. This can cause the op amp after the detector to clip, creating intermodulation distortion and generally ruining the signal quality.

It also affects the input to the sound card, and requires lowering the gain, losing weaker signals in the process. This problem is fairly inherent in a wide bandwidth receiver. To mediate this problem, a bandpass filter was designed and built for the 20 meter (14.1 MHz) band. The response before and after tuning is was measured. Before tuning, the different poles of the filter were at different frequencies creating a rather wide passband with a 'lumpy' response.

![20 m band bandpass filter]({{ site.url }}/images/software_radio/20m-bandpass.jpg)

![Filter response before tuning]({{ site.url }}/images/software_radio/20m-bandpass-insertion-loss.png)

[jeri_vids]: "http://www.youtube.com/watch?v=vY9Hi1Jiadg"
[qex]: "http://www.flex-radio.com/Data/Doc/qex1.pdf"
[hdsdr]: "http://www.hdsdr.de/"
[gnuradio]: "http://gnuradio.org/"
[gitsdr]: "https://github.com/laszuba/Pulsar_SDR"
