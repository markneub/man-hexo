---
title: LED Music Visualizer
date: 2012-01-14 21:31:28
tags:
 - project
 - electronics
 - audio
 - LEDs
---

<div class="aspect-ratio four-three"><iframe src="//www.youtube.com/embed/___XwMbhV4k" frameborder="0" allowfullscreen="" style="margin-bottom: 20px;"></iframe></div>

Audio is passed in via a standard audio cable through the back of the unit. Inside, a spectrum analyzer chip breaks down the sound into seven frequency bands, passing the results to an Arduino microcontroller. The Arduino takes this information and directs four LED drivers to display the audio spectrum by powering a matrix of 49 LEDs in time with the music.

<!-- more -->

## Parts List ##

- 1x Arduino Duemilanove
- 1x Bliptronics Spectrum Analyzer shield
- 1x 9 VDC 1A regulated switching power adapter
- 1x 5 VDC 1A regulated switching power adapter
- 4x Capacitor Ceramic 0.1 µF
- 1x DC Barrel Power Jack/Connector
- 28x green LEDs
- 14x orange LEDs
- 7x red LEDs
- 49x T1 3/4 LED mounting hardware
- 4x 1.5 kΩ resistor
- 4x TI TLC5940 LED driver
- 2x Breadboard
- Wire, wood, screws

## Schematics & Source

- {% asset_link visualizer_bb_lg.png Breadboard %}
- {% asset_link visualizer_schem_lg.png Schematic %}
- {% asset_link visualizer.ino.txt "Arduino source code" %}

## Construction Video

<div class="aspect-ratio sixteen-nine"><iframe src="//www.youtube.com/embed/D_83ZUk8p-U" frameborder="0" allowfullscreen=""></iframe></div>