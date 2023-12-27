---
title:      3D Printing with Ninjaflex on a Mendel90
date:       2016-08-18 16:46:01
category:   3d-printing
tags:       3d-printing, Ninjaflex
published:  true
---

Today, for the first time I tried printing with [Ninjaflex](https://ninjatek.com/products/filaments/ninjaflex/). Having not printed with it before, I was a little concerned that it might prove a touch difficult. It really didn't, but I did learn that most people set their printers to run a lot hotter and faster than I do.

- **Printer**: [Mendel90](http://reprap.org/wiki/Mendel90)
- **Filament**: [3mm Ninjaflex in blue](http://shop.3dfilaprint.com/sapphire-blue-3mm-5-metre-wrap-463-p.asp)

Before I started I did a bit of reading around the subject:

* <http://forums.reprap.org/read.php?292,325378>
* <http://forums.reprap.org/read.php?1,269018>

Based on the advice in those articles (and a test print at 195&deg;C which failed to extrude properly) I tried the following settings:

* Print speed: 30mm/s
* Printing temperature: 210&deg;C
* Bed temperature: 50&deg;C
* Retraction:
    * Minimum travel: 1mm
    * Z hop when extracting: 0.1mm

If you would like the full profile I used it's here: <https://raw.githubusercontent.com/HACManchester/cura-settings/master/ninjaflex.ini> I coated the bed in dilute PVA just as I would if I were using PLA in the printer although it appears you can also get good results using blue printers tape or possibly even nothing on the bed.

![The base of the octopus as it's being printed](/assets/2016-08-18-octopus-printing.jpg)

The final print is a bit bobbly in places but has generally come out pretty well:

{:.gallery.two-by-one}
![Closeup of the finished octopus](/assets/2016-08-18-finished-octopus.jpg)
![Closeup of the finished octopus with its head being squished.](/assets/2016-08-18-finished-octopus-sqashed-head.jpg)
