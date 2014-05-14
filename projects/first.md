---
layout: post
title: FIRST Team 3590
---

# {{ page.title }}

January - May 2011

In my final year of high school, our school received the funding for a second FIRST robotics team. My teachers at the time gave me the fantastic opportunity to run a very small team of students independent of the other school team.

At the competition we had a fantastic team of students that worked together incredibly well to coordinate everything and fix problems as they came up. We performed very well, winning Highest Rookie Seed, and getting knocked out by the formidable Team 2056 in the semi-finals.

Many of the parts of the robot were constructed on an old CNC machine that was designed for teaching I updated with [EMC][emc] and a new stepper drive system a few months prior.

![Overview 1]({{ site.url }}/images/first/full1.jpg)

![Overview 2]({{ site.url }}/images/first/full2.jpg)

![Overview 3]({{ site.url }}/images/first/full3.jpg)

![Overview 4]({{ site.url }}/images/first/full4.jpg)

Part of the competition included a "mini bot" which was deployed to climb a pole for extra points in the final seconds of the match. Ours worked somewhat well, making it up the pole successfully roughly half the time. When the main robot launched the minibot, contact with the post released a latch, clamping the wheels on and starting the motor.

![Minibot]({{ site.url }}/images/first/minibot.jpg)

The elbow gearbox was machined directly into the arm to save some weight and make everything more compact. Shafts were machined on a bench lathe from steel hex stock.

![Elbow drive gearbox (integrated into arm)]({{ site.url }}/images/first/elbow-gearbox.jpg)

The shoulder joint utilizes the only welding, as we had to take it to a shop to get it done.

![Front shoulder detail]({{ site.url }}/images/first/shoulder1.jpg)

![Back shoulder detail]({{ site.url }}/images/first/shoulder2.jpg)

The shoulder drive gearbox was a little more simple, but with a higher reduction ratio and more powerful motors. The gears were also machined out for weight savings. An encoder along with a limit switch on the shoulder itself provided position feedback.

![Shoulder drive gearbox (with feedback)]({{ site.url }}/images/first/shoulder-gearbox.jpg)

The main drive used two speed pneumatic transmissions (purchased than modified for weight) to maximize speed. The compressor, solenoids, and tank for the pneumatic system are pictured as well.

![Drive and pneumatic system]({{ site.url }}/images/first/powertrain1.jpg)

All six wheels were driven, using a simple chain setup. The center two wheels were slightly lower, giving the robot a point to pivot on.

![Drive chains and wheels]({{ site.url }}/images/first/powertrain2.jpg)

[emc]: "http://www.linuxcnc.org/"
