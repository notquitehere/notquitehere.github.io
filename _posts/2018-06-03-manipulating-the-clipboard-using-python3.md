---
title: "Manipulating the clipboard on Mac using python3"
date: 2018-06-03
published: true
excerpt: "Accessing and manipulating the contents of the Mac clipboard using Python3."
cover:
tags: python
---

Sometimes I make a lot of effort in order to be extremely lazy. In this instance I got bored of manually generating folder names from book titles so decided to create a script to do it for me.

Initially I was copying the title, pasting it into the `input()` then copying out the result and pasting it into the new folder dialog. Even this seemed like too much effort. A short web search, led me to [Build a Shared Clipboard Utility in Python : Page 3](http://www.devx.com/opensource/Article/37233/0/page/3). The relevant bit for my purposes (since I'm only using this on my Mac) is Listing 4 which gives this snippet:

```python
import subprocess

def get_clipboard_data():
    p = subprocess.Popen(['pbpaste'], stdout=subprocess.PIPE)
    retcode = p.wait()
    data = p.stdout.read()
    return data.decode('utf-8')

def set_clipboard_data(data):
    p = subprocess.Popen(['pbcopy'], stdin=subprocess.PIPE)
    p.stdin.write(data.encode('utf-8'))
    p.stdin.close()
    retcode = p.wait()
```

I have added the `encode()` and `decode()` calls to the clipboard functions in order that they not have to be repeated across calls.

Now I have this I need to add it to my existing string manipulation function and make it run on demand (as it intercepting **all** clipboard traffic would be annoying).

```python
import regex as re

translator = str.maketrans('. ', '-_')

if input('Ready: ').lower() != "x":
    op = re.sub('[^{\\w .}]+', "", get_clipboard_data()) # remove all punctuation apart from periods
    op = re.sub('__+', "_", op).translate(translator).lower() # replace any resulting __ with _, switch . for -
    set_clipboard_data(op)
    print(op)
else:
    break
```

Now all I have to do is copy the filename, cmd+tab over to my terminal app ([iTerm2](https://www.iterm2.com/)) and press enter.

