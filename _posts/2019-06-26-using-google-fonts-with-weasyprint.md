---
title: "Using Google Fonts With WeasyPrint - Two Ways"
date: 2019-06-26
modified: 2019-06-26
tags: python weasyprint pdf-generation
published: true
---

I needed to generate a pdf and I was looking for a way to do so that didn't involve me writing raw [Latex](https://www.latex-project.org/) (I use [Pandoc](https://pandoc.org/) for a reason k). So today, for the first time I tried using [WeasyPrint](https://weasyprint.org/) in order to programatically generate pdf's from [Jinja2](http://jinja.pocoo.org/docs/2.10/) templates.

Creating the template was pretty easy especially once I got my head around the fact that I could use mm instead of any of the other options, one thing that did trip me up however was the css. I'm so used to using bootstrap and google fonts that it took me a little while to work out why my css wasn't rendering properly. FWIW using Bootstrap to generate pdf's is a terrible idea, it's not intended for printing and WeasyPrint ignores media queries so even if you do load it it still looks wrong. Just craft your own css.

Something that I really did want to fix however was the font. There are two ways you can fix this in WeasyPrint:

- the 'normal' css way: by installing a local copy and using `@font-face`
- by adding the remote stylesheet to your stylesheet list when calling `write_pdf()`

## Using local versions

CSS:

```css
@font-face {
  font-family: 'Raleway';
  src: local('resources/raleway/Raleway-Light.ttf') format('truetype');
  font-weight: normal;
}

@font-face {
  font-family: 'Raleway';
  src: local('resources/raleway/Raleway-SemiBold.ttf') format('truetype');
  font-weight: bold;
}
```

In order for this to work you need to create a `FontConfiguration` object.

```python
from weasyprint.fonts import FontConfiguration

font_config = FontConfiguration()

HTML(string=out, base_url=__file__).write_pdf(
    "invoice.pdf",
    stylesheets=["style.css"],
    font_config=font_config
)
```

More detail: [WeasyPrint tutorial](https://weasyprint.readthedocs.io/en/stable/tutorial.html)

## Adding the remote stylesheet

The above works perfectly well, **however**, if you're using a font which is available through google fonts you can just add the stylesheet directly to the stylesheets list and avoid all the messing around with `@font-face` entirely:

```python
HTML(string=out, base_url=__file__).write_pdf("invoice.pdf",
                                              stylesheets=["style.css",
                                              "https://fonts.googleapis.com/css?family=Raleway:400,600&display=swap"])
```

That's all there is to it.
