---
layout: post
title: "Playing Artemis on macOS"
date: 2016-11-30 17:44:17
categories: gaming macOS
---

I recently discovered Artemis. It's amazing, but the problem is that it only runs
on Windows, iOS, and Android. These are my notes on how I got it working on one of
my macOS machines.

1. Install [PlayOnMac](https://www.playonmac.com/en/).
1. Followed the onscreen instructions, including installing XQuartz
1. Download the [ArtemisDemo](http://artemis.eochu.com/ArtemisDEMOInstall.exe), you'll need it for the inital install.
1. Click on "Install a program"
1. Click on "Install a non-listed program"
1. Read the popups for the installation
1. Create a new "virtual drive"
1. "Select another file" as the installation program, select the `ArtemisDEMOInstall.exe`
1. Select `Artemis` as the installation shortcut
1. Complete installation
1. Select `Artemis` in the "PlayOnMac" screen
1. Click "Run"
1. You should see the game (Demo).

If you have a copy of the game already for instance on something like Steam,
and would like to copy it in, you can take the folder from something like:
`C:\Program Files(x86)\Steam\steamapps\common\Artemis` and copy it into something like:
`/Users/YOURUSER/Library/PlayOnMac/wineprefix/DRIVENAME/drive_c/Program Files`

After this, you can open up the folder, and double click `Artemis.exe`, and you should
see the application full start.

![One of the greatest games](http://artemis.eochu.com/wp-content/uploads/2016/02/big-front.png)
