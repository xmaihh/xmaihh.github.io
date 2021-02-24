---
title: '[ArchLinux]pacman -S sudo不成功'
date: 2018-07-10 11:18:38
categories: Linux
tags: [Archlinux]
toc: false
description: Whatever happens tomorrow,we've had today. 
---
I want to install sudo. So I type in pacman -S sudo. But then I get the following errors:
```
~$: pacman -S sudo
warning: database file for 'extra' does not exist
warning: database file for 'community' does not exist
error: failed to prepare transaction (could not find database)
```


Firstly, try running pacman -Syy, then try to install sudo again.

Check that the repositories are uncommented in /etc/pacman.conf.

Or your mirrorlist might be outdated: Generate a current list of mirrors and copy it to /etc/pacman.d/mirrorlist

Quoting from this relevant forum thread:

You can:

pick another mirror
try using an http mirror, not an ftp one (pick http mirror from the mirrorlist).
Alternatively you can manually download the databases with:
```
wget ftp://mirror.csclub.uwaterloo.ca/archlinux/community/os/x86_64/community.db
wget ftp://mirror.csclub.uwaterloo.ca/archlinux/extra/os/x86_64/extra.db
```
move them to  /var/lib/pacman/sync/  and run 'pacman -Syu' again. If you find any *.part files in ```/var/lib/pacman/sync/``` e.g. ```/var/lib/pacman/sync/core.db.part``` - remove them.

To prevent having problems like these it is critical to understand pacman. To learn more about using pacman, see the ArchWiki pacman article, and consult man pacman.
