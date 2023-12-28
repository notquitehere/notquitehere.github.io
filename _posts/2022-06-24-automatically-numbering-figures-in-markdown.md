---
title: Automatically numbering Figures in Markdown
date: 2022-06-25
published: true
tags: markdown
category:
---

<style>
    /* initialise the counter */
    body { counter-reset: figureCounter; }
    /* increment the counter for every instance of a figure even if it doesn't have a caption */
    figure { counter-increment: figureCounter; }
    /* prepend the counter to the figcaption content */
    figure figcaption::before {
        content: "Fig " counter(figureCounter) ": "
    }
</style>

Markdown is really powerful, both for it's simplicity and for the fact that it can be extended by using HTML/CSS.

This allows you to do things that Markdown can't natively such as adding a caption to a figure:

```html
<figure id="my-image">
    <img src="my_image.jpg" alt="don't forget to put some alt text here">
    <figcaption>This will be rendered below the image</figcaption>
</figure>
```

And even automatically numbering them by setting up a css `counter` in an internal `<style>` block. Fortunately for what we're about to do, `<style>`  blocks **do not** have to be in the document `<head>` to work.

```css
<style>
    /* initialise the counter */
    body { counter-reset: figureCounter; }
    /* increment the counter for every instance of a figure even if it doesn't have a caption */
    figure { counter-increment: figureCounter; }
    /* prepend the counter to the figcaption content */
    figure figcaption:before {
        content: "Fig " counter(figureCounter) ": "
    }
</style>
```

Now every `<figcaption>` will be prepended with a fig number that is associated with it's location, even if you don't caption one of them.

<div style="display: flex; column-gap: 0.5em;">
<figure>
    <img src="https://placekitten.com/600/500" alt="placeholder kitten from placekitten.com">
    <figcaption>First fig</figcaption>
</figure>
<figure>
    <img src="https://placekitten.com/600/500" alt="placeholder kitten from placekitten.com">
</figure>
<figure>
    <img src="https://placekitten.com/600/500" alt="placeholder kitten from placekitten.com">
    <figcaption>Last fig</figcaption>
</figure>
</div>

## Special Cases

### Obsidian

Although this will work for **most** markdown editors, it does not work in Obsidian. However, should you want to recreate the behaviour there you can do so by creating an appropriately named css file in `<vault>/.Obsidian/snippets` and then activating the snippet via Settings > Appearance > CSS Snippets.

### Ghost

If you use <a href="https://ghost.org/">Ghost</a> you will find that this method as written targets the Feature Image as well as any images you have defined as `<figure>` yourself. To get around that you can target `.gh-content`:

```css
.gh-content { counter-reset: figureCounter; }
.gh-content figure { counter-increment: figureCounter; }
.gh-content figure figcaption::before {
    content: "Fig " counter(figureCounter) ": "
}
```

## References

- [How to Automatically Number Elements with CSS Counters](https://www.freecodecamp.org/news/numbering-with-css-counters/)
- [Numbered Headings in Markdown via CSS](https://gist.github.com/patik/89ee6092c72a9e39950445c01598517a)
- [Obsidian CSS Snippets](https://blog.dev0.sh/2022/03/14/obsidian-css.html)
