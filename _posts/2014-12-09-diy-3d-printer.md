---
layout: post
title: DIY 3D Printer
date: 2014-12-09 09:29
headerImage: false
tag:
- projects
projects: true
hidden: true
blog: false
star: false
category: project
author: anthonymonori
description: One of my latest weekend projects was the DIY 3D printer that I have put together with the help of Arduino Verkstadt, in Malmö, Sweden.
---

One of my latest weekend projects was the DIY 3D printer that I have put together with the help of Arduino Verkstadt, in Malmö, Sweden. The printer is a modified version of the Mendel Prusa i3 3D printer, called Steelwonder, and it can be found on [Github](https://github.com/anthonymonori/Steelwonder) or on Arduino Verkstad's [website](http://verkstad.cc/3d_Printer/).

To quote the description from the Github repository of the printer:
> _The frame P3Steel is a remix of Twelvepro's redesign of Josef Prusa's i3. [...] the printed parts have been borrowed from the box design from the current Mendel Prusa i3 box design. [...] The plastic parts have mostly been sourced from the box design of the main Prusa i3 branch. [...] The shroud was sourced from this made by stratop80, also available on tinkercad. Although it was revised and only used for inspiration. [...] The source for the Y-endstop holder for the moving y-carriage can be found here here is made by Antonio Navarro._


**So to start with, I am going to list all the mechanical parts of the printer:**

* 3mm steel frame
* Smooth and threded rods
* Electronics holder
* 3-angle screen holder
* Hot-end cooling mechanism (ventilator)
* 40mm fan shroud
* Shroud for encoder
* Y-belt holder (back, front, support)
* Shroud for nozzle head
* GT2 6mm belt
* GT2 pulleys
* Wheel and other parts for the extruder
* 5x NEMA 17 stepper motors
* PCB hotbed
* Glass plate/mirror for heated bed
* Lots of tools, bearings, nuts, washers and screws


**And now the electronics:**

* 110/220 V power supply
* Arduino Mega 2560
* RAMPS 1.4 board
* Stepper motor drivers
* 20x4 LCD display with backlight
* HRS 163 reverse type normal size SD Card reader
* EC11E15244B2 alps encoder
* 4050 level shifter
* 100K ohm NTC thermistor

As the first step in the process of building the printer, the steel frame had to be assembled. The following YouTube video demonstrates the steps:

<iframe width="560" height="315" src="//www.youtube.com/embed/In_Q6NkX3fs" frameborder="0" allowfullscreen></iframe>

**Here's a picture of it, as well:**

![The steel frame of the 3D printer](/assets/images/projects/diy-3d-printer/diy-frame.jpg)

After that, the rods and the Y-axis extruder support shrouds were installed. Then later on, the hot bed and the power supply as well.

![The steel frame from a different angle](/assets/images/projects/diy-3d-printer/diy-frame-2.jpg)

Finally, the Arduino board, the RAMPS board, the thermistor, the noozlehead and the stepper motors were mounted and connected using the following diagram:

![Diagram of the wiring](/assets/images/projects/diy-3d-printer/diagram.svg)

**Here's the final outcome:**

![The 3D printer](/assets/images/projects/diy-3d-printer/diy-outcome.jpg)

The printer is running on the latest version of Marlin open source software for 3D printers, found [here](https://github.com/ErikZalm/Marlin). For modelling, I use AutoCAD 2014 and with the help of [Slic3r](https://github.com/alexrj/Slic3r) I convert them into G-code files, a file format recognized by RepRap printers.

Currently I have a 1.75 mm noozlehead installed on my extruder and I'm using PLA plastic for printing. The printer still requires some tweaking and calibration in the future.

_Special thanks to Daniel and Tere from Arduino Verkstadt for helping us out during the weekend!_

_To see my other projects, please visit the [Projects](/projects) page of my website. Many thanks!_
