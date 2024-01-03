---
title:      "Automatic tag pages for Jekyll on Github Pages"
date:       2024-01-03
published:  true
cover:
tags:       jekyll python git
---

The extremely useful [jekyll-tagging](https://github.com/pattex/jekyll-tagging) plugin doesn't work on github pages, but it is still possible to automatically generate tag clouds and tag pages, just slightly more difficult.

This heavily leverages Jekylls `site.tags` variable which handily collects all of your tags together with their associated posts and returns them as a map.

## Display tags on a post

Each post has access to its own tags via `page.tags` so you can display your tags by iterating over that array somewhere within `_layouts/post.html`

```liquid
{% raw %}
<ul>
    {%- for tag in page.tags -%}
        <li><a href="/tag/{{ tag }}">{{ tag }}</a></li>
    {%- endfor -%}
</ul>
{% endraw %}
```

## Individual tag pages

### Create a layout

In order to generate individual tag pages you will need a layout for them in `_layouts`

```liquid
{% raw %}
---
layout: default
---
<h2>{{ page.tag }}</h2>
<ul>
{% for post in site.tags[page.tag] %}
  <li><a href="{{ post.url }}">{{ post.title }}</a> ({{ post.date | date_to_string }})
  {%- if site.show_excerpts -%}
    {{ post.excerpt }}
  {%- endif -%}
  </li>
{% endfor %}
</ul>
{% endraw %}
```

### Add a collection

There are a few ways to do this bit however I've gone with creating a collection. This allows me to have the tags appear at `/tags/<tag>` rather than `/tag/<tag>`.

```yml
collections:
    tag:
        permalink: /tags/:title
        output: true

defaults:
  -
    scope:
      path: ""
      type: "tag"
    values:
      layout: tag_page
```

### Generate the pages

In order to generate a page for each tag you need to create a `_tag` folder and a `<tag>.md` file for each tag you've used. That can be done manually. This is all you need:

```
---
tag: python
---
```

That's pretty simple though so it's easy enough to generate a script thats a bit more sophisticated to generate the files for us[^tag-script-link]. To do this effectively we need to:

- [generate a list of the files we already have (existing)](#existing)
- [generate a list of the tags that are currently in use (valid)](#valid)
- [create any files that are valid but not existing](#create-files)
- [remove any files that are existing but not valid](#remove-files)

And finally, because remembering to do things is hard, we should make a git hook that runs the script when we commit.

[^tag-script-link]: The script I use is committed to github along with this blog and can be found here: <https://github.com/notquitehere/notquitehere.github.io/blob/main/_tag_gen.py>

#### Generate a list of existing files {#existing}

This just requires us to list the stem[^stem] of all the files in the `_tag` folder

```python
from pathlib import Path

tags_path = Path("_tag")

existing_tag_files = [f.stem for f in tags_path.glob("*.md")]
```

[^stem]: in Pathlib the file name (`f.name`): `example.txt` can be split into `f.stem`: `example` and `f.suffix`: txt

#### Generate a list of the tags that are currently in use {#valid}

This one is a little more complex as we need to iterate over all the files in the `_posts` folder to get the tags. Files in other folders aren't processed by default so we're going to ignore them for the moment.

As the tags will always be in the frontmatter we can stop processing the file when we find the end of the block. This isn't going to make a lot of difference for a small blog but for a larger one it will save a fair amount of time.

Once we have the list we can use `set()` to ensure there is only one of each tag present.

```python
used_tags = []

for post in Path("_posts").glob('*.md'):
    with open(post) as f:
        end = False
        for line in f:
            if line.startswith("---"):
                if end:
                    # got to the end of the front matter so stop now
                    break
                end = True
            elif line.startswith("tag"):
                used_tags.extend(line.split(":")[1].split())

used_tags = set(used_tags)
```

#### Create any files that are valid but don't exist {#create-files}

Because `used_tags` is a `set` we can use [`difference`](https://docs.python.org/3.12/library/stdtypes.html#frozenset.difference) to identify which tag files need to be generated.

We can also leverage [`write_text`](https://docs.python.org/3.12/library/pathlib.html#pathlib.Path.write_text) from Pathlib to create the file in a single line.

```python
for tag in used_tags.difference(existing_tag_files):
    tag_file_content = f"---\ntag: {tag}\n---"
    tags_path.joinpath(f"{tag}.md").write_text(tag_file_content)
```

#### Remove any files that exist but aren't valid {#remove-files}

Again we can use `difference` to identify any files which need to be deleted however this time we need to turn `existing_file_tags` into a set as only the non-operator can be any iterable.

Pathlib's `unlink` method can be used to remove a file or symbolic link.

```python
for tag in set(existing_tag_files).difference(used_tags):
    tags_path.joinpath(f"{tag}.md").unlink()
```

#### Create a githook[^jekyll-hook]

Local hooks live in `.git/hooks/` which is automatically generated when you initialise a new repository. In order to turn our script into a git hook we need to add an exception (to stop the commit when the tags files change) and a shebang to ensure the correct version of python is being used.

If you're not using a virtual environment (and you don't really need to with this script as it's not relying on anything that isn't supported by stock python) you can link to the python version found by running `which python3` on the command line.

We also need to add a custom exception to the script to make it clear what the issue is when it fails.

```python
#!/usr/local/bin/python3

class CommitError(BaseException):
    """Raised when tags have changed to prompt the user to restart the process."""
```

At the end of the script we can use [symmetric_difference](https://docs.python.org/3.12/library/stdtypes.html#frozenset.symmetric_difference) to check if there are any differences between the two lists and stop the commit if there are.

```python
if used_tags.symmetric_difference(existing_tag_files):
    raise CommitError("Tags have changed. Add tag file changes to commit.")
```

Copy, move or symlink the script file into `.git/hooks/` as `pre_commit` (no extension) and make the file executable

```shell
cp _tag_gen.py .git/hooks/pre-commit
chmod +x .git/hooks/pre-commit
```

Now, whenever you commit your script will check for tag changes, generate/remove the tag files and then prompt you to add the changes to the commit.

[^jekyll-hook]: Jekyll has a built in hooks system which you could also use see for example: [Managing tags in Jekyll blog easily](https://blog.lunarlogic.com/2019/managing-tags-in-jekyll-blog-easily/)

## Tag cloud

If you want a tag cloud with css classes that match those in jekyll-tagging here's a way to do it. Liquid isn't elegant but this works and is relatively fast. At time of writing it takes my blog around 2s to regenerate on save.

Note that the `group` variable is generated by dividing `max_posts` by 5.0 in order to get even distribution even over relatively small groups of tags. This isn't particularly robust and will fail to assign appropriately on groups of tags where there aren't any assigned to multiple posts.


```liquid
{% raw %}
<div id="tag-cloud">
    {% assign max_posts = 0 %}

    {% for tag in site.tags %}
        {% if tag[1].size > max_posts %}
            {% assign max_posts = tag[1].size %}
        {% endif %}
    {% endfor %}

    {% assign group = max_posts | divided_by: 5.0 %}

    {% for tag in site.tags %}
        {% assign class = 5 %}
            {% for i in (1..4) %}
                {% assign boundry = group | times: i %}
                {% if tag[1].size < boundry %}
                    {% assign class = i %}
                    {% break %}
                {% endif %}
            {% endfor %}
        <a href="/tag/{{ tag[0] }}" class="set-{{ class }}">{{ tag[0] }}</a>
    {% endfor %}
</div>
{% endraw %}
```

## Resources:

- [How to implement Jekyll tags on github pages](https://tainenko.github.io/jekyll-tag-on-github-pages/)
- <https://www.fabriziomusacchio.com/blog/2021-08-12-Liquid_Cheat_Sheet/#site-variables>
- [Liquid docs](https://shopify.github.io/liquid/)