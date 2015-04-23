---
layout: post
title: Reverse Engineering an RGB LED Controller
---

# {{ page.title }}

On a recent trip to Shenzhen, China, I had the fantastic opportunity to visit the electronics market at Hua Qiang Bei. Among other things, it is perhaps the best place in the world to buy loads of cheap LEDs. My co-worker found several vendors selling RGB LED strips, with about 1 LED per 2 cm. The best part? They cost $1/m. The LEDs are wired in series/parallel, with 3 of the same colour in series with a current limiting resistor, than these groups of 3 in parallel with all others of the same colour. He also found controllers that have a little IR remote and the current capacity for 2 A, or 10 m of LEDs.

The controllers are small and cheap, but they don't work particularly well. The remote has fairly limited colour options and doesn't work all the time. But before replacing the controller with my own electronics, I figured it would be fun to reverse engineer them.

## Component Breakdown

The controller consists of a single one-sided PCB with no jumpers. It has very few components. The first part is some simple voltage regulation, composed of D2 and R1. D2 is a 4.7 V zener diode, which creates the logic supply for the rest of the circuit. This explains the controller's 5-12 V rating. Above 12 V the power dissipation through the series resistor will be ~80 mW, quite high for a resistor in that package, and ~50 mW in the zener diode, not terrible, but a little high. The controller could potentially operate up over 12 V, but it would be wise to replace the 4.7 V power supply with something a little more robust at that point. The lower 5 V limit is to give the zener diode a little bit of headroom so it has current through it to maintain regulation. (This was probably set in the opposite order, the engineer decided on a lower limit of 5 V and picked the next lowest common diode voltage.)

Next, C2 acts to smooth the power supply and bypass the serial EEPROM supply, U1. To find the value I measured the capacitance on the whole rail with a DMM. C2 is about the right size for a 6.3 or 10 V rated 10 uF ceramic capacitor, so it makes sense that it provides most of this capacitance. In the same vein, C3 bypasses the IR receiver and U2. I was not able to measure it directly, but based on the size and usage I would guess 100 nF.

Third is the serial EEPROM. It seems to be similar to an Atmel AT24C02 2 K bit I2C EEPROM based on the part number and footprint, although with slightly different markings. The first clue that gave it away were the 4 grounded pins down the left side. This is common and represents ground and three address pins. In most applications with a single I2C device, the address is tied at 0x0 for simplicity. After that with some googling I was able to find a pin compatible part from a known manufacturer. It wouldn't be surprising if this was a gray market replacement for an Atmel EEPROM to save some cost.

The IR receiver sits off-board, attached to a short ribbon pigtail. It's a common 3 pin 38 kHz photodiode + demodulator, used to receive TV remotes everywhere. This particular one seems to be equivalent to the Vishay TSOP381x, based on the shape and pinout.

The actual controller chip was a lot more tricky to identify. It has power and ground on pins 1 and 8, consistent with a lot of 8 pin PIC microcontrollers. It probably doesn't need much program space or RAM, especially based on the external EEPROM. It could even be OTP (One Time Programmable). I don't have a PIC programmer to try and connect to it with, so there isn't much further to be done. The best guess it probably a PIC12F508 because of the low cost, although it would have to be a more expensive part if they use more than the limited memory in that part. At some point using the Bus Pirate to read the contents of the EEPROM could give some more clues. An interesting note is the low-end PICs tend not to have hardware I2C peripherals, so the interface would have to be bit-banged. It could also be a PIC compatible knock-off part, particularly if it is OTP.
