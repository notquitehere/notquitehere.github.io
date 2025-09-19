---
title: "Tracking my reading in Obsidian"
date: 2024-06-30
published: true
tags: obsidian
updated: 2025-08-24
# excerpt: ""
# cover:
---

At some point last year I moved from doing a half arsed job of tracking my reading on GoodReads to doing a slightly less
half arsed job of tracking it on Oku. My intention in doing so was to facilitate embedding a roughly up-to-date list of
things I was reading onto a "Now" page here. I never got to the latter and as I recently discovered that Oku is no
longer available on the Apple App Store in the UK I realised that I might as well move this whole functionality into my
PKM.

## Pre-requisites

- [Obsidian](https://obsidian.md/)
- [Book Search Plugin](https://github.com/anpigon/obsidian-book-search-plugin)
- [Dataview](https://blacksmithgu.github.io/obsidian-dataview/)

## Adding Books

Although a lot of people prefer to keep everything in the base folder of their PKM I find that for distinct collections
like this one it's better to keep it all together in a single folder. So I started by creating a library folder and
setting up my Book Search Plugin to always add new books to it.

I also created a `book-note` template to be used when I invoke "Create a New Book Note".

## Shelves

### To read

This is for me one of the most important shelves - here is where I dump book recommendations and things I come across
that I'd like to read so that I don't have them scattered all over the place. None the less the dataview query is very
simple.

```js
TABLE WITHOUT ID
	"![|60](" + cover + ")" as Cover,
	link(file.link, title) as Title,
	author as Author,
	join(list(publisher, publish)) as Publisher
FROM "library"
WHERE shelf="to-read"
SORT file.ctime ASC
```

### Reading

The currently reading query is almost identical to the to-read one: the only difference is the shelf being referenced:

```js
TABLE WITHOUT ID
	"![|60](" + cover + ")" as Cover,
	link(file.link, title) as Title,
	author as Author,
	join(list(publisher, publish)) as Publisher
FROM "library"
WHERE shelf="reading"
SORT file.ctime ASC
```

### Read this year

**Make sure you have dataviewjs enabled for your vault.**

This one is a little bit more complex than the other shelves, because I opted for multiple read dates I need a function
to check whether any of them are in the year I'm interested in and then regurgitate the WHOLE book so that it can appear
in the list more than once.

To achieve this I'm going to write a function which iterates over the read dates array and returns the book if its
been read in the relevant year.

```js
function booksReadThisYear(allBooks) {
    let bookList = [];

    allBooks.map(b => {
        b.readDates.map(d => {
            // If the year in the date matches the year that the file was created in include the book.
            // The year could be hardcoded, however, using the file creation year makes it simple to use this script in
            // a template file and always have it be correct (as long as the file was created in the right year).
            if (new Date(d).getFullYear() === dv.current().file.cday.year) {
                let bk = Object.assign({}, b);
                bk.readDates = d;
                bookList.push(bk)
            }
        })
    })

    // sort ascending - switch a and b in the return statement for descending
    bookList.sort((a,b) => {
        return new Date(a.readDates) - new Date(b.readDates)
    });

    return bookList
}
```

In order to avoid having this throw errors I need to weed out any books that haven't got any read dates. There are two
options for doing this:

- use an if statement to check `readDates` is not empty before attempting to iterate over them
- use a `where()` pipe to remove them first.

I chose to do the former BEFORE passing the remaining part of the list to the function.

```js
dv.pages('"library"').where(b => b.readDates)
```

Once that's done I use the returned array to construct a dataview table:

```js
dv.table(
    ["cover", "title", "author", "series", "read on"],
    booksRead.map(b => [
        // render the cover image
        "![" + b.cover + "|80](" + b.cover + ")",
        // render a link to the book file with the title as the display name
        dv.fileLink(b.file.path, false, b.title),
        b.author,
        b.series,
        b.readDates,
    ])
);
```

Here's the full query in the right order:

```js
function booksReadThisYear(allBooks) {
    let bookList = [];

    allBooks.map(b => {
        b.readDates.map(d => {
            if (new Date(d).getFullYear() === dv.current().file.cday.year) {
                let bk = Object.assign({}, b);
                bk.readDates = d;
                bookList.push(bk)
            }
        })
    })

    bookList.sort((a,b) => {
        return new Date(a.readDates) - new Date(b.readDates)
    });

    return bookList
}

const booksRead = booksReadThisYear(dv.pages('"library"').where(b => b.readDates));

dv.table(
    ["cover", "title", "author", "series", "read on"],
    booksRead.map(b => [
        "![" + b.cover + "|80](" + b.cover + ")",
        dv.fileLink(b.file.path, false, b.title),
        b.author,
        b.series,
        b.readDates,
    ])
);
```

## Bibliography

Things I read to help me make this:

- [How I track books and reading with Obsidian](https://ajy.co/how-i-track-books-and-reading-with-obsidian/)
- [Obsidian Book Search Plugin](https://github.com/anpigon/obsidian-book-search-plugin)