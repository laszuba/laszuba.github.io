---
layout: post
title: "Dynamic Flowers - A robotic sculpture"
date: 2024-08-01
categories: sculpture electronics mechanics
---

# Introduction

"Dynamic Flowers" was conceived leading up to my wedding. Instead of a typical altar, myself and my partner wanted to showcase artwork that incorporated robotically-driven movement. The long-term goal is to create 4 more flowers and stands to make the artwork more immersive. I am currently manufacturing the remaining flowers.

The sculpture consists of 12 petals controlled by 4 motors with complete position feedback. The underside is illuminated by a full-colour LED array designed for smooth dimming and natural colour reproduction through an additional white channel and software spectral optimization. The petal position, and light colour and brightness can be commanded over a standard theatrical lighting control protocol (DMX512). For the wedding, a controller based on a Raspberry Pi running [OLA](https://www.openlighting.org/ola/) was provided to the DJ. The controller played back recorded sequences or loops created in [TouchDesigner](https://derivative.ca/) throughout the ceremony and evening.

# Photos

{% include image.html url="/images/dynamic_flowers/wedding-overall.jpg" description="Complete flower at the wedding venue" %}

{% include image.html url="/images/dynamic_flowers/wedding-underneath.jpg" description="Underside showing translucent petals" %}

{% include image.html url="/images/dynamic_flowers/wedding-ceremony.jpg" description="Flower at rest during ceremony" %}

{% include image.html url="/images/dynamic_flowers/wedding-nighttime.jpg" description="Nighttime, with soft purple light and gentle ambient motion" %}

# Design

Myself and my partner wanted to explore an interplay of materials that are of great interest to us. The design incorporates translucent porcelain ceramic for the top of the flower, a molded glass light diffuser, white tulle petals, and an aluminum, steel and bronze structure and mechanics.

Ceramics development is exceptionally time consuming, so I decided to re-use part of an existing project, the shades from my [Ceramic Lamp]({% post_url 2020-06-01-ceramic_lamp %}). I created several prototypes to develop a suitable mechanism to fit in the space under the shade and allow moving all 12 petals. After a series of conceptual sketches, I modelled the design in CAD and started developing CAM toolpaths and fixturing for manufacture.

{% include image.html url="/images/dynamic_flowers/design-complete.png" description="Complete CAD with stand" %}

{% include image.html url="/images/dynamic_flowers/complete-light_and_mechanics.jpeg" description="Motion assembly" %}

{% include image.html url="/images/dynamic_flowers/design-overall_integration.png" description="Mechanical integration" %}

{% include image.html url="/images/dynamic_flowers/design-electronics_integration.png" description="Electronics under flower" %}

# Light

The light consists of 6 LED modules with 4 channels, RGB and white, on a custom metal-core PCB. The LEDs are the LumiLEDs Luxeon series, selected for optimum colour mixing near the blackbody curve. I wrote a Python script to select the best LEDs, relying on the fantastic [Colour](https://github.com/colour-science/colour) library to calculate the IES TM-30 Rf and Rg from the LED power spectral densities. The script also determines ideal mixing ratios for the application software, using numeric optimization constrained by the aforementioned light quality metrics. The design is able to achieve a CRI of >85 across a range of 2800K to 6500K with only one white, and no additional amber or lime channel.

The diffuser is made from glass slumped in a custom ceramic mold designed to reflect the shape of the upper ceramic shade.

{% include image.html url="/images/dynamic_flowers/electronics-led_module.jpeg" description="4 channel LED module" %}

# Motion

The motion is driven by 4 motors, each connected to 3 petals through universal joints. I machined most of the parts on my home-built [CNC Mill]({% post_url 2023-08-01-cnc_mill %}). Design for manufacture using my small, home-built mill was very difficult. I was not able to rely on any tight tolerances, especially accurate running fits for shafts and pins. The design heavily relies on pre-ground rod and reamed holes for the close-tolerance motion components. Most of the critical components are bronze or 304 stainless to avoid corrosion and provide wear-resistant sliding surfaces. 304 stainless in particular required careful machining strategies to avoid excessive chatter on the small mill.

{% include image.html url="/images/dynamic_flowers/design-motor_module.jpeg" description="Test assembling 1 of 4 motor modules" %}

{% include image.html url="/images/dynamic_flowers/design-u_joint_detail.png" description="Detail showing belt driving U-joint with bearings" %}

{% include image.html url="/images/dynamic_flowers/machining-test_cuts.jpeg" description="Test cuts to determine if machining the U-joints in 304 stainless is viable" %}

{% include image.html url="/images/dynamic_flowers/machining-u_joint-setup.jpeg" description="Final U-joint setup" %}

{% include video.html url="/images/dynamic_flowers/machining-u_joint-cnc.webm" description="U-joint machining" %}

{% include image.html url="/images/dynamic_flowers/machining-u_joint-coolant.jpeg" description="Hard to see under the coolant while the parts are running" %}

{% include image.html url="/images/dynamic_flowers/machining-u_joint-complete.jpeg" description="After the CNC, waiting on deburring and manual operations to add the pivot holes" %}

{% include image.html url="/images/dynamic_flowers/machining-u_joint-deburring.jpeg" description="All parts are carefully deburred by hand to remove sharp edges from machining" %}

{% include image.html url="/images/dynamic_flowers/machining-bearing_toolpath.png" description="CAM toolpaths for secondary bearings" %}

{% include image.html url="/images/dynamic_flowers/machining-u_joint-inner_spacer.jpeg" description="Some parts can be machined in groups, like these U-joint pivot spacers" %}

{% include image.html url="/images/dynamic_flowers/machining-petal_clamp.jpeg" description="Adding a tapped hole to the petal clamp on the manual mill" %}

# Electronics

[Complete Schematic (PDF)](/documents/flower_controller-1.pdf)

Each flower contains a custom control MLB based on a Teensy 4.1 (Cortex-M7) microcontroller module. It receives commands over DMX512 / RS485 and handles control of the motors and lights. The motors are standard DC gearmotors, driven by DRV8847 integrated H-bridges. Position feedback uses magnetic encoders, one positioned on the rear shaft of the motor to measure relative motion without backlash, and one at the output of the gearbox to measure absolute position.

{% include image.html url="/images/dynamic_flowers/electronics-mlb_render.png" description="Render of Flower control MLB" %}

{% include image.html url="/images/dynamic_flowers/electronics-mlb_layout.png" description="Layout of controller, bottom side" %}

{% include image.html url="/images/dynamic_flowers/electronics-mlb.jpeg" description="Control MLB before installation of through-hole parts" %}

{% include image.html url="/images/dynamic_flowers/design-motor_position.png" description="Detail of rear-shaft motor position feedback" %}

# Firmware

The firmware architecture is a simple main loop, written in C++. I tried to use this as an opportunity to learn modern C++ features like type-safety (eg. `std::optional`), templating, memory management, and standard library allocators (eg. `std::array`).

The main loop decodes the incoming DMX512 packets, filters the input to smooth abrupt changes and improve the effective frame-rate, and updates the light and motion controller:

``` cpp
void controller_main() {
    // Get and process latest DMX packet if it exists
    dmx_manager.run();
    
    // ...

    // LED controller updates
    led_controller.set_global_brightness(global_brightness);
    for (size_t i = 0; i < HardwareConfig::num_leds; i++) {
        led_controller.set(i, led_setpoints[i]);
    }

    // Motion controller updates
    for (size_t i = 0; i < motion_controllers.size(); i++) {
        motion_controllers[i].set(motor_setpoints[i]);
    }

    // Run motion control loops
    for (auto& motion_ctrl : motion_controllers) {
        motion_ctrl.run();
    }
}
```
(everything other than core functionality removed for clarity)

The motion controller is a simple PID architecture with a gravity feed-forward term:

``` cpp
void MotionController::_update_control_loop() {
    // Feedforward
    float output_ff_gravity = _config.ff_gravity_constant * sin(_shaft_position);

    // Run PID
    float output_pid = _pid.compute(_shaft_position, _setpoint);
    _output = output_pid + output_ff_gravity;

    // Output result and measure current
    _motor.set(_output);
    _motor_current = _motor.get_current();
}
```

{% include image.html url="/images/dynamic_flowers/testing-pid_tuning.jpeg" description="PID tuning with test electronics and a bit of extra weight" %}

The firmware handles typical errors gracefully, monitoring temperature, DMX512 status, position sensor bus errors, and motor overcurrent. It attempts to keep remaining parts of the system functional even under error conditions, while preventing long-term damage to subsystems.

# Software

Motion and light choreography was programmed in a framework built in TouchDesigner. I was quite short on time at this point in the project, with all of the other wedding preparations ongoing, so only one sequence was completed, along with some gentle ambient motion that ran on a loop throughout the evening.

{% include image.html url="/images/dynamic_flowers/software-td_project.png" description="Animating in TouchDesigner, and trying to avoid too much spaghetti code" %}

# Petals

The petals are made from 3/32" stainless welding wire, bent and welded to an attachment plate. They are covered in white tulle fabric, with two layers to provide an interesting interplay of light reminiscent of the beat frequencies that form the core of radio signal processing and electronics.

{% include image.html url="/images/dynamic_flowers/manufacture-petal_welding.jpeg" description="Welding fixture for petals" %}

{% include image.html url="/images/dynamic_flowers/manufacture-petal_bending.jpeg" description="Fixture to bend stainless wire" %}

# Stand

I designed the stand to come apart into easy-to-transport pieces that are held together with spring pins.

{% include image.html url="/images/dynamic_flowers/stand-setup.jpeg" description="Dad and brother setting up the flower" %}

{% include image.html url="/images/dynamic_flowers/stand-tube_notching.jpeg" description="Notching tube on the mill before welding the stand" %}
