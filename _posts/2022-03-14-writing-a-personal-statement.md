---
title: Writing a personal statement
date: 2022-03-14
tags: python
published: true
---

Writing a personal statement for an application can be tricky, especially if all you have initially is some take on "because it's awesome!". To get past that first hurdle it can be tempting to just write that for the required length so, lets do it:

```python
def write_personal_statement(file_name, statement, max_length):
    with open(file_name, 'a') as f:
        f.write((statement * (max_length//len(statement)+1))[:max_length])

if __name__ == '__main__':
    write_personal_statement('ps.md', "Because it's awesome! ", 5000)
```

Your output will look something like this:

<pre style="background-color: #15171a; color: #e5eff5; white-space: pre-wrap">
Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Because it's awesome! Becaus
</pre>

Just don't upload it by mistake.

<iframe width="740" height="416" src="https://www.youtube.com/embed/dkF7Bj5cxJU?si=ESt1Uj8yBckRNish" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>