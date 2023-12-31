---
title: rsync everything
date: 2022-06-07
published: true
excerpt: "A quick post with the basics of rsync for those (like me) who forget the syntax on a regular basis."
---

There are lots of great tutorials about rsync. This is not intended to be one of them. Instead its a glorified cheat sheet, a reminder of the syntax and switches when I realise I have once again forgotten them.

## Basics

```bash
rsync -azP source/ destination
```

Fun with flags

- `-a`: archive, this combines recursive syncing ( `-r` ) with the preservation of symbolic links, special and device files, modification times, groups, owners, and permissions.
- `-z`: adds compression which reduces network transfer.
- `-P`: this is another combination flag, this time covering `--progress` and `--partial` giving you both a progress bar and the ability to resume an interrupted transfer.


Trailing slashes:

Add a trailing slash if you want to sync the contents of your source directory to your destination one. Without it the folder itself will be sunk resulting in a `destination/source/source-contents` folder structure.

## Testing

Not sure you've got your command right? Lets do a dry run to see

```bash
rsync -anv source/ destination
```

Fun with flags:

- `-n`: this is the bit that tells rsync to do a dry run
- `-v`: verbose, useful if you want to actually know the outcome


## Remote sync

One of the main strengths of rsync is the ability to transfer files between local and remote systems

```bash
rsync -azP source username@location:destination
```

If you have `~/.ssh/config` set up you can also do

```bash
rsync -azP hostname:source destination
```

## Excluding things

Exclude a specific file

```bash
rsync -azP --exclude 'file.txt' source destination
```

or directory

```bash
rsync -azP --exclude 'dir' source destination
```

or just the contents of the directory

```bash
rsync -azP --exclude 'dir/*' source destination
```

multiple files/directories

```bash
rsync -azP --exclude={'file.txt', 'dir', 'file1.md', 'dir2'} source destination
```

This last one is getting a bit complicated, so if you're doing this you might whatn to use an exclude file instead:

```bash
rsync -azP --exclude-from='exclude-file.txt' source destination
```

Then in your exclude file:

```
file1.txt
file2.txt
dir
dir/*
```

## Resources

- [How To Use Rsync to Sync Local and Remote Directories \| DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-use-rsync-to-sync-local-and-remote-directories)
- <https://linuxize.com/post/how-to-exclude-files-and-directories-with-rsync/>

