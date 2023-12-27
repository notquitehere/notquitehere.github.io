---
title:      "Silly Strings: Passwords in Python"
Date:       2015-11-15
updated:    2023-12-27
category:   cryptography
tags:       Python, Cryptography, Flask, WordPress
published:  true
---

I've been playing with [Flask](http://flask.pocoo.org/) a lot in the last few days. If you're going to do anything with sessions in Flask you first need to generate a secret key which you use to cryptographically sign the session. I got a tad bored of mashing the keyboard to generate them so I wrote a short script to do it for me:

```python
import string
import random
import re

chars = string.letters + string.digits + string.punctuation
chars = re.sub('[1lI0Oo-]', '', chars)
length = 0

while length == 0:
    how_long = raw_input("how many characters? ")

    try:
        length = int(how_long)
    except ValueError:
        print "Please try again, whole numbers only."

    print ''.join(random.choice(chars) for x in range(length))
```

This produces a string of arbitrary length that excludes easily confused characters. Of course if you want to produce a string for cryptographically signing something this is a better way of doing it:

```python
import os
print os.urandom(n).encode('base-64')
```

This leverages the power of your operating system to generate a random string of an arbitrary length (n) which should be random enough for most cryptographic applications. You can't use it to generate useful passwords, but if you're mainly interested in generating secure keys this is the method you want.

## If you have to do something three times automate it:

This quick (python 2) script does the above but with a whole lot less typing:

```python
import os

n = 0

while n == 0:
    how_long = raw_input('How long? ')

    try:
        n = int(how_long)
    except:
        print "Whole numbers only."

print os.urandom(n).encode('base 64').
```

It's not the most verbose script in the whole world 'cause it's really only intended to stop me having to type `os.urandom(n).encode('base 64')` repeatedly.

Slightly improved script (ported to Python 3) which allows you to output more than one string at a time (useful for setting up WordPress).

```python
from base64 import b64encode
from os import urandom

n = 0
t = 0

while n == 0 or t == 0:
    how_long = input('How long? ')
    how_many = input('How many times? ')

    try:
        n = int(how_long)
        t = int(how_many)
    except:
        print ("Whole numbers only.")

for _ in range(1,t+1):
    print(f"\n{b64encode(urandom(n)).decode('utf-8').rstrip("=")}\n")
```