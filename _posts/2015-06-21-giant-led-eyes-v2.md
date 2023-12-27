---
title:      Giant LED eyes v2
tags:       3d-printing, bugzilla
category:   3d-printing
published:  true
---

Having proved I could print a shell 5 times the size of a standard LED I thought I'd repeat the process on [MadLab's](http://madlab.org.uk) newly acquired resin printer (it's a [Form 1+](http://formlabs.com/products/form-1-plus/)) to see what the difference in quality was.

![Comparison of FDM and SLA printing techniques on the same part](/assets/2015-06-21-fdm-sla.jpg){:.img-center}

As you can see it's a much cleaner, more solid looking print albeit a touch wonky. It also looks a whole lot more like an LED than my first print.

![Close up of the SLA print](/assets/2015-06-21-sla-closeup.jpg){:.img-center}

OK, it's a lot wonky:

![The other side of the print showing how wonky it had gone](/assets/2015-06-21-sla-wonky.jpg){:.img-center}

This really surprised me, and my initial explanation was that the wobble must have been caused by the printer being on a flimsy table while lots of people trooped past to get lunch. Closer inspection however revealed that the wobble was all on the "bottom" of the print. This suggested that it hadn't had enough support to keep itself the right shape during the printing process. Again, I was surprised as, having not used this printer before, I was quite careful to [RTFM and follow the instructions](http://formlabs.com/support/guide/). With slight trepidation therefore I ignored the auto-orientation button, rotated my print to the angle I thought I would print best at and then auto generated supports. Having done that, I spent a few minutes removing the majority of the internal ones and replacing them with the minimum number that the software thought it could handle (I wasn't quite brave enough to just leave them out entirely). The result is pictured below

![SLA print with the internal structure I created](/assets/2015-06-21-internal-structure.jpg){:.img-center}

As you can see it's a lot cleaner print, I didn't move the printer to do it, I just used the settings my instincts and experience of 3D printing told me to use rather than the auto-generated ones (kinda makes me think of [this t-shirt](http://teespring.com/autorouter)). Once I'd got a clean print, I discovered that the tolerance on the Form 1+ is a lot tighter than that on the [Mendel 90](http://reprap.org/wiki/Mendel90) we have in [HacMan](http://hacman.org.uk). So where the [Adafruit NeoPixel Jewels](http://shop.pimoroni.com/products/adafruit-neopixel-jewel-7-x-ws2812-5050-rgb-led-with-integrated-drivers) I am using to make these monsters glow was a tight fit on the original it now falls out and had to be hot glued in place (I'd have used something more elegant but we seem to be suffering from a bit of an adhesive shortage).

![Hot glue holding the NeoPixel ring in place](/assets/2015-06-21-neopixel-hot-glue.jpg){:.img-center}

So now it was time to connect it to an [Arduino](http://www.arduino.cc/) and make it do funky things:  The code it's running here is a version of the `theaterChase` code included in the [Adafruit NeoPixel Library](https://github.com/adafruit/Adafruit_NeoPixel), I had to tweak it a little I didn't want it to include the central NeoPixel (`0`) when cycling and I wanted it to only light two pixels, one on each side of the Jewel in order to create a circling effect.

```c
void rotate_single_colour(uint32_t c, uint8_t wait) {

    /* iteration starts at 1 to make it ignore the central pixel of the jewel. */

    for (int i=1; i &lt; pixels.numPixels(); i++) {
        // wipe all the pixels, so we're starting from blank each time.
        // this is simpler than attempting to selectively turn off pixels

        for (int d = 0; d &lt; pixels.numPixels(); d++ ) {
            pixels.setPixelColor(d, 0);
            }
        // set the first pixel to the chosen colour
        pixels.setPixelColor(i, c);

        // work out which pixel is directly opposite the lit one
        int z = i + 3;

        // check that the iterated pixel no. is actually within the jewel
        if ( z &lt; pixels.numPixels() ) {
            pixels.setPixelColor(z, c);
        } else {
            /* if the pixel isn't within the jewel take away the no of pixels then add 1 to compensate for the fact that 0 isn't being used. */

            z = z - pixels.numPixels() + 1;
            pixels.setPixelColor(z, c);
        }

        pixels.show();
        delay(wait);
    }
}
```

Did you really come to see where we'd got to with putting the giant BUG! together? Here you go:

![BUGZILLA!](/assets/2015-06-21-bugzilla-assembly.jpg){:.img-center}

That can is full btw.

## Reminder:

If you want to see the BUG! in all it's hugemongous glory and make yourself a normal sized one we will be at [Liverpool MakeFest](https://lpoolmakefest.wordpress.com/) next Saturday 27th June 2015 and [Manchester MakeFest](http://www.mosi.org.uk/whats-on/makefest.aspx) on 8th/9th August 2015.
