---
title:          "Update: tracking my reading in Obsidian"
date:           2025-08-24
published:      true
# cover:
tags:           obsidian
excerpt:        "Tracking my reading in Obsidian has been far more successful than my previous efforts would suggest was possible. However, the initial setup was complicated and views were reliant on external files. The recent introduction of the Bases core plugin however has made the setup for these collections much simpler."
discussion_id:  2025-08-24-tracking-reading-with-obsidian-bases
---

Having finally managed to find a method of tracking my reading which I am actually willing to maintain I decided to mess with the system by attempting to port it wholesale over to [Obsidian Bases](https://help.obsidian.md/bases). Bases is a new core plugin which provides database style functionality within Obsidian. It is simpler to use than dataview[^dv1] and much, much faster to render. It also works **really** well on mobile which is one of the main ways I interact with my notes.

In the process of converting my files I found that, as Bases is view based rather than being code embedded in a markdown file, I only needed/wanted to create two bases files (tracking and shelves) rather than the many dataview files I had originally.

[^dv1]: I'm talking here about the original dataview rather than dataview core. The latter is meant to be a lot faster but also requires you to create React components which isn't really something I'm that keen on doing in my note taking app.

## Pre-requisites

- [Obsidian](https://obsidian.md/), with the bases core plugin enabled
- [Book Search Plugin](https://github.com/anpigon/obsidian-book-search-plugin)

## Adding books

In my setup books notes are created in a library folder using the book search plugin. I automatically download the cover thumbnails but they couldn't be used in my javascript (and I never cared enough to figure out why). Bases will use these downloaded files though so I have updated my custom book note template to always put that link into the "cover" section of the front matter.

## Tracking everything

I have 3 types of tracking file:

- Currently reading
- To read
- Read in <year>

Because all of my book notes are in a single folder I started by creating a global filter for the base limiting the indexing to that folder: `Filter > All views > All the following are true > where folder is library`.

### To read / currently reading view

The to read and currently reading views are very similar. They each

- have a filter which limits them to the relevant shelf:
    - to-read: `Filters > This view > All the following are true > shelf contains to-read`
    - currently reading: `Filters > This view > All the following are true > shelf contains reading`
- are sorted by `file.ctime`
- display:
    - cover-image
    - title as link
    - author
    - series

Because of these similarities once I'd created one of them I could duplicate it and just change the filter.

### Read by year

I currently have read lists for both 2024 and 2025, these are identical apart from the filter so again I created one and duplicated it. These views have:

- a custom filter to allow partial string matching:

    `Filter > This view > All the following are true > Advanced filter > where readDates.toString().contains("2025")`
- a filtered read-dates list, which is used for sorting the books. Without filtering bases sorts by the first date in the list which messes up the order for any books that have been read multiple times
- display:
    - cover-image
    - title as link
    - author
    - series
    - filtered read dates

### Adding properties

Properties can be added to a Base using [functions](https://help.obsidian.md/bases/functions). These are added in the body of a formula to do things like display images and filter lists. I added the following functions to re-create my dataview layouts:

1. Cover image: `image(cover)`, to display it at a useful size I then increased the row height to "Tall".
2. Filename as link: `link(file.name, title)`
3. Filtered read dates: `readDates.filter(value.toString().contains("2025"))`

## Shelves

The other thing I'm doing with my Obsidian library is using it to catalogue the books I have and what format(s) they're in. To do this I've been adding shelves such as "kindle", "mobi", "pdf", "mp3", "audible" and "physical". I have a bunch of files that group these into shelves such as "audiobook", "ebook" and "dead-tree". These I have also recreated as a single Base file.

The initial setup for this file was very similar to the tracking file. I limited the view globally to my library folder and for each view used the "contains any of" filter to grab only the entries I wanted. Even the columns are very similar to those I used in the tracking base, and the only reason these aren't in the same file is to avoid the visual overload of having so many views in one place.

Columns:

- Cover
- Author
- Title (as link)
- Series
- Type

This last one is view specific as I want to display the format in the folder, it uses the same filtering model as the filtered read dates did but instead of looking for a partial string it looks for one of a set of partial strings. This one is for audiobooks:

```
shelf.filter(value.toString().containsAny("audible", "mp3"))
```

That's it. Using Bases, the whole setup is done in seconds, now all I need to do is read some more books so I can make it to my target for this year.