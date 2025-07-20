---
title:      "Fractals in Rust"
date:       2025-07-19
published:  true
cover:      /assets/2025-07-19-julia.png
tags:       rust fractals
excerpt:    Unravelling the math behind Julia sets and colour gradients through the medium of rust.
---

Rust and Python have a common problem: they both make it really easy to do very complex things in a single step. As a
result, this video about [rendering the Julia set](https://youtu.be/g4vN2Z0JuZI?si=ZdzPqqF5_vnsnJjH) sent me down a
fractal hole.

Fractals aren't something I've played with much before, so first I needed to go look at some math. Quadratic Julia sets,
it turns out, are fairly simple. They're generated using the mapping $f(z) = z^2 + c$ for a fixed $c$ and almost all
values of $c$ will generate something. Notable exceptions to this are -2 and 0[^1].

[^1]: <https://mathworld.wolfram.com/JuliaSet.html>

To create a Julia set fractal $z$ is initialised using the coordinates for each pixel within the image space and then
repeatedly updated, calculating its magnitude on each iteration. The result of this calculation is used to identify
whether or not to colour the pixel.

So the key steps are:

- [Complex number multiplication](#complex-number-multiplication)
- [Magnitude calculation](#magnitude)

## Complex number multiplication

Alright, it's been a while lets look at multiplying complex numbers... like binomials complex numbers are multiplied using the FOIL method (Firsts, Outers, Inners, Lasts). Which gives me

$$
(a+bi)(c+di) = ac + adi + bci + bdi^2
$$

However, because $i^2$ is always -1 this can be simplified to[^2]

$$
(a+bi)(c+di) = (ac-bd)+(adi+bci)
$$

[^2]: For an actual explanation see [Maths is Fun: Complex Number Multiplication](https://www.mathsisfun.com/algebra/complex-number-multiply.html).

That's not super complicated to to replicate, especially if tuples are used to represent the numbers:

```rust
fn complex_multiplication(n1: (f64, f64), n2: (f64, f64)) -> (f64, f64) {
    let re = (n1.0 * n2.0) - (n1.1 * n2.1);
    let im = (n1.0 * n2.1) + (n1.1 * n2.0);
    (re, im)
}
```

And can be extended to return $z^2 + c$:

```rust
fn step(z: (f64, f64), c: (f64, f64)) -> (f64, f64) {
    let re = (z.0 * z.0) - (z.1 * z.1) + c.0;
    let im =  (z.0 * z.1) + (z.1 * z.0) + c.1;
    (re, im)
}
```

## Magnitude

Having updated the value of $z$ you need to check whether its magnitude has gone above 2 (thus taking it outside the
edge of the set), the greater the number of iterations required to do this the brighter the pixel will be. As a complex
number can, for these purposes, be considered coordinates; the magnitude is equivalent to the hypotenuse of the right
angle triangle formed by the x-axis and an imaginary perpendicular line intersecting the point itself.

```rust
let hypotenuse = (re * re + im * im).sqrt();
```

## Is it in the Julia set?

Sticking with greyscale as used in Tim's video that gives:

```rust
fn step(z: (f64, f64), c: (f64, f64)) -> (f64, f64) {
    let re = (z.0 * z.0) - (z.1 * z.1) + c.0;
    let im =  (z.0 * z.1) + (z.1 * z.0) + c.1;
    (re, im)
}

fn julia(c: (f64, f64), x: f64, y: f64) -> u8 {
    let mut re = x;
    let mut im = y;

    for i in 0..255 {
        let hypotenuse = (re * re + im * im).sqrt();
        if hypotenuse > 2.0 {
            return i as u8;
        }

        (re, im) = step((re, im), c)
    }
    255
}
```

Which is almost twice as long as the original but doesn't require any imports.

### Generating the image

This bit of code is essentially the same as it was in Tim's video with a couple of minor changes:

1. I increased the image size
1. I switched from using 3.0 as the scaling factor to using $Pi$ which should improve the centering - though plausibly
   not noticeably. Regardless, I have no desire to create a Mail Sorting Engine so we'll stick with observed constants
   rather than coerced ones[^3].

[^3]: Pratchett, Terry. Going Postal. 2005. Corgi. (Support your local independent booksellers.)

```rust
fn main() {
    let width = 1920;
    let height = 1080;

    let scale_x = PI / width as f64;
    let scale_y = PI / height as f64;

    let mut img = ImageBuffer::new(width, height);

    for (x, y, pixel) in img.enumerate_pixels_mut() {
        let cx = x as f64 * scale_x - PI/2.0;
        let cy = y as f64 * scale_y - PI/2.0;

        let c = (-0.8, 0.156);

        let value = julia(c, cx, cy);
        *pixel = Rgb([value, value, value])
    }

    let _ = img.save("julia1.png");
}
```

<div class="gallery two-by-one">
    <figure>
        <img alt="Julia set generated from 0.28+0.008i" src="/assets/2025-07-19-julia1.png">
        <figcaption>Julia set generated from 0.28+0.008i</figcaption>
    </figure>
    <figure>
        <img alt="Julia set generated from -0.79+0.15i" src="/assets/2025-07-19-julia2.png">
        <figcaption>Julia set generated from -0.79+0.15i</figcaption>
    </figure>
</div>

## Adding colour

Assigning the same number to each colour channel gives a greyscale image. To get a coloured image we need to mutate the
distance value to make the values slightly different in each channel. After reading through KrazyDad's extremely
comprehensive tutorial[^4] I settled on this range of colours:

[^4]: [Making annoying rainbows in javascript](https://krazydad.com/tutorials/makecolors.php)

<p>
<script type="text/javascript">
    // Thanks are owed to KrazyDad (krazydad.com) for this javascript snippet
    var frequency = .3;
    for (var i = 0; i < 20; ++i) {
        red = Math.sin(frequency*i + 3) * 140 + 50;
        green = Math.sin(frequency*i - 1) * 140 + 50;
        blue = Math.sin(frequency*i + 1) * 140 + 50;
        // Note that &#9608; is a unicode character that makes a solid block
        document.write( '<font style="color: rgb(' + red + ',' + green + ',' + blue + ')">&#9608;</font>'); }
</script>
</p>

To build RGB values from the single return value we need to convert each value using the following attributes:

- frequency: how close to make the sine wave oscillations (period)
- width: the maximum value we can get out (amplitude) - this wants to be approximately half our total range so, as the
  maximum valid number is 255 this should be 255/2 or ~128
- center: the center position of the wave (where the zero point is)
- phase: how far to offset the wave from 0

Rust doesn't have Python style list comprehensions natively (they can be added with `cute`) but it does provide an
[enumerate function](https://rust-guide.com/en/documentation/iterators/enumerate) so this rather repetetive assignment
can be done in a DRY manner.

```rust
    const COLOUR_FREQUENCY: f32 = 0.3;
    const COLOUR_WIDTH: f32 = 140.0;
    const COLOUR_CENTER: f32 = 50.0;

    fn main() {
        // snip
        for (x, y, pixel) in img.enumerate_pixels_mut() {
            // snip
            let mut rgb = [0; 3];
            for (idx, phase) in [3.0, -1.0, 1.0].iter().enumerate() {
                rgb[idx] = (f32::sin(COLOUR_FREQUENCY * value + phase) * COLOUR_WIDTH + COLOUR_CENTER) as u8;
            }

            *pixel = Rgb(rgb);
        }
    }
```

On its own this works but it's a touch stark as each value is discrete. Increasing the iterations and subtracting a
fractional amount ($\log_2(\log_2(|z|))$) from the return value gives smoother contours. Be warned though, if you end
up with an [amorphous blob](#amorphous-blob) this takes AGES to generate due to the sheer number of pixels inside the
shape.

```rust
for i in 0..5000 {
    let hypotenuse = (re * re + im * im).sqrt();
    if hypotenuse > 2.0 {
        return i as f32 - (hypotenuse.log2()).log2() as f32;
    }

    (re, im) = step((re, im), c)
}
```

<div class="gallery two-by-two">
    <figure>
        <img alt="Julia set generated from -1.476+0.0i" src="/assets/2025-07-19-julia3.png">
        <figcaption>Julia set generated from -1.476+0.0i</figcaption>
    </figure>
    <figure>
        <img alt="Julia set generated from -0.162+1.04i" src="/assets/2025-07-19-julia4.png">
        <figcaption>Julia set generated from -0.162+1.04i</figcaption>
    </figure>
    <figure>
        <img alt="Julia set generated from 0.25+0.25i" src="/assets/2025-07-19-julia5.png" id="amorphous-blob">
        <figcaption>Amorphous blob Julia set generated from 0.25+0.25i</figcaption>
    </figure>
</div>

## Comparison

To see how far we've come from the original output of the code, here it is side by side with the same fractal
($c = -0.8+0.156i$) generated using the updated code.

<div class="gallery two-by-one">
    <figure>
        <img alt="Original Julia set rendering in greyscale" src="/assets/2025-07-19-julia.png">
        <figcaption>Original Julia set rendering in greyscale</figcaption>
    </figure>
    <figure>
        <img alt="Colour Julia set using the updated code" src="/assets/2025-07-19-julia6.png">
        <figcaption>Coulour Julia set using the updated code</figcaption>
    </figure>
</div>

## Resources

- [Understanding Julia and Mandelbrot Sets](https://www.karlsims.com/julia.html)
- [Fractals continuous colouring](https://www.paridebroggi.com/blogpost/2015/05/06/fractal-continuous-coloring/)
- [Optimising a Julia Fractal Animation](https://rahulpai.co.uk/optimising-a-julia-fractal-animation.html)