---
layout: post
title:  "Musical Heart"
date:   2014-02-14 21:58:29 -0800
---
<div class="aspect-ratio sixteen-nine"><iframe width="640" height="390" src="//www.youtube.com/embed/MlsUbHCxCiI" frameborder="0" allowfullscreen="" style="margin-bottom: 20px;"></iframe></div>

## Description ##

A week before Valentine's Day, I had an idea for a fun interactive project. I rush ordered parts and managed to get it done just in time. The device uses a stethoscope and a microphone to capture a user's heartbeat, then visualizes it with music box tones and a flashing electroluminescent panel. Below are some process pictures and notes of what I made.

## Build Process ##

{% asset_img process1.jpeg "Prepping heart shapes" %}

I prepped some scrap wood and laser cut it into heart shapes. This wood is Brazilian mahogany I got from my Dad that was originally leftover ends from a rebuilt front porch. These two pieces will be combined to form the lid of the box. The bottom half of the top (shown here to the left of the top half) has three holes laser etched into it to allow space for small neodymium magnets, which will match magnets in the body of the box to keep the enclosure shut, but allow access when necessary.

{% asset_img process2.jpeg "Making the comp-heart-ment" %}

The glued up top with hidden magnets is shown in the middle. Next, I cut the bottom of the box (at top) from the same mahogany. The main body of the compartment (bottom) is made from poplar. It's almost completely black at this point due to charring from the laser cutter. At a thickness of 3/4", it was barely thin enough to laser cut.

{% asset_img process3.jpeg "Glued up comp-heart-ment" %}

Here's the glued up compartment, with the magnets nicely hidden away from view. There are three magnets to complement those in the lid, placed just beneath the surfaces of the three pillars in the body. They were drilled in from the opposite direction with a drill press.

{% asset_img process4.jpeg "Stethoscope fitting" %}

The hole in the bottom fits a stethoscope, kindly donated to the project by a doctor friend. You can see by the darker color that I've finished the mahogany at this point. I chose to use Danish oil, which lends a nice color and also helps to protect the finish.

{% asset_img process5.jpeg "Adding stethoscope tubing" %}

I had to chisel out the inside of the compartment a little bit to make room for the vinyl tubing from the stethscope, but it ultimately fit in very nicely. I used a quick set epoxy to secure the stethoscope inside the housing. At the top of the enclosure, you can also see the vintage power switch I got from my grandfather.

{% asset_img process6.jpeg "Widening tubing" %}

The next step was to cut down the vinyl tubing and widen the hole at the end using a cordless electric drill. It was a little difficult to keep the tube still, but I got the result I wanted after a few tries. You can also see a better view of the power switch here.

{% asset_img process7.jpeg "Mounting microphone" %}

After preparing the end of the tube, I mounted a small electret microphone in the end of the tubing. It kept wanting to slide out, so I pressed it in place with some super glue and ultimately had a very nice fit.

{% asset_img process8.jpeg "Wiring up the Arduino" %}

I chose to use the Arduino Pro Mini 5V for this project. Here I have a 9V battery (off screen) connected to an LM7805 voltage regulator, the power switch, and the Arduino.

{% asset_img process9.jpeg "Assembling components" %}

Here's a slightly better view of the components. At this point I've also added some circuitry to connect the microphone to an LM386 amplifier and then to an analog input on the Arduino. The amplifier circuit is attached to a small piece of protoboard, cut out on the bandsaw. This was my first time working with protoboard and I attached my components upside down (it's a lot easier if you place the parts on the side opposite the one with the metal contacts, and solder on the contact side!).

{% asset_img process10.jpeg "Final circuitry" %}

The final circuitry, with the Arduino programming wires yet to be packed in. I've added two more subsystems: a SOMO-14D audio-sound module wired to a speaker, and a 3V EL inverter switched by a PN2222 transistor. The inverter is connected to a red electroluminescent panel that's hot glued to the inside of the lid of the enclosure, upside down in the upper left part of the picture. One nice property of electroluminescent panels is that they can be cut to shape and still function.

{% asset_img process11.jpeg "Final result" %}

The final result, with everything wired up and the lid snapped in place.

{% asset_img stand.jpg "Final result with 3D printed stand" %}

Bonus picture: 3D printed stand that looks a bit like a mustache.

## Parts List ##

- 1x Arduino Pro Mini 5V
- 1x 9V Li-ion battery
- 1x LM7805 5V regulator
- 1x SPDT switch
- 1x stethoscope
- 1x small electret microphone
- 1x LM386 amplifier
- 1x SOMO-14D audio-sound module
- 1x small speaker
- 1x PN2222 bipolar junction transistor
- 1x 3V to 110VAC EL inverter
- 1x 10cm x 10cm red EL panel
- 6x small neodymium magnets
- Various capacitors and resistors
- Wire, wood, glue, epoxy