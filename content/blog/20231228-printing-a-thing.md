+++
title = "3D Printing Stupid Shit: Pt. 1"
description = "I turn my laziness into productiveness by 3d printing something I could have gone to the store for and saved 99.8% in costs."
date = 2021-05-01T09:19:42+00:00
updated = 2021-05-01T09:19:42+00:00
draft = false
template = "blog/page.html"
slug = "printing stupid shit pt 1"

[extra]
lead = ""
math = true
toc = true
images = ["/images/20231228-cad.png"]
+++

I often need something mundane and either creative or vigorous to do as an outlet; drawing, chopping wood, doing yardwork, hitting a punching bag, etc. Lately this has been 3d printing objects that serve some purpose around the house. My _pièce de résistance_ is a part that fixes a drawer in the kitchen due to a bent rail.

I was doing some cleaning in my office and decided it would be nice to have some hooks for these hand grippers I have on my desk.

I could have gone to the hardware store, but I felt like being lazy, so I did what any lazy person would do....I modeled some hooks in CAD software and 3d printed some instead. It may sound crazy but the math kinda maths.

<video src="https://cdn.fs10xer.dev/gripper_holder.mp4" controls poster="/images/20231228-finished_hooks.jpg" width="100%"></video>

<p style="font-size: 12px;">This sick retro 8-bit anime themed song came on the rotation during the making of this video and I decided it's the theme song for this vid.</p>

[audio source](https://www.newgrounds.com/audio/listen/952216)



## The Math

* The closest hardware store is 2 miles away.
* The average price of wall hooks I see on their store is $3 (for a pack of 6).
* My truck gets about ~15 miles to the gallon.
* Gas is ~$5 where I live.
* 1 spool of filament is 1kg and cost me $30.
* 1 gram is therefore about $0.03.
* 1 hook will print about 6 grams of filament.

$$ costdriving = {((miles/milespergallon) * gasprice) + hookcost} $$

vs

$$ costprinting = {(spoolcost/spoolamount) * gramsused * hooksprinted} $$

#### If I were to drive ~4 miles to purchase hooks it would cost me ~$4.33.

#### If I were to 3d print hooks it would cost me ~$0.72, _plus_ I don't have to go anywhere or get dressed.

The choice is clear.

## Modeling

I need basically a simple j-hook and plan to use a drywall screw to fix it to the wall. I need 4 of them.

I use [FreeCAD](https://www.freecad.org/) (FOSS) for modeling. I keep it pretty simple and get most of this info from youtube. A few straight lines with dimensions, padding, some circles with pockets, and that's good for now. Export this as an STL and load it into the slicer.

<img src="/images/20231228-cad.png" width="100%" />

## Slicing

I use [PrusaSlicer](https://www.prusa3d.com/page/prusaslicer_424/) to import the CAD file and prep it for printing. I already have this tuned to my printer's Z-offset, some custom macros, and a brim. I export the g-code and prepare the printer.

<img src="/images/20231228-slicer.png" width="100%" />

## Printing

I have an Ender 5 Pro. It's a good machine for tinkering. It takes a bit to get it dialed in but you can customize it a lot. Once it's dialed you get great prints.

<img src="/images/20231228-printer.gif" width="40%" />

The printer has been flashed with [Klipper](https://www.klipper3d.org/) to give me some lower level capabilities and higher performance. I'm also using [OctoPrint](https://octoprint.org/) running on a Raspberry Pi which controls the printer via USB serial and has a nice web interface for managing things remotely. The printer is 5 feet away from my desk but it is nice to check on prints when doing other stuff. I mainly use this for the timelapse feature which is a lot of fun.

<img src="/images/20231228-octoprint.png" width="100%">

## Results

It took me about 4 hours to print all the hooks. This is the end result. It's not my Mona Lisa but I think it's pretty sick.

<img src="/images/20231228-finished_hooks.jpg" width="100%"> 