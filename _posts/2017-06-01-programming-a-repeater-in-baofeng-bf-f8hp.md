---
layout: post
title: "Programming a repeater in Baofeng BF-F8HP"
date: 2017-06-01 21:58:19
categories: ham
---

I recently got licenced as a ham radio operator. After doing some research,
the beginner radio I landed on was the Beofeng BF-F8HP. It seems that the instructions
are for the previous generation and there were...complications to getting the thing to
tune into a local repeater. Here are some notes that I've tested a few times that
hopefully will help someone else down the line.

First thing:

ERASE EVERYTHING. Especially if you just opened it. The settings are crazy and confusing,
and it's best to start with a blank slate.

How to reset it:

```
[Menu]-[4 VOX]-[0 SQL]
[Menu]
[Menu] # now everything is erased
[Menu]-[1 STEP]-[4 VOX] # now everything is in Chinese
[Menu]-[Down arrow] # to English
[Menu]
```

This is when you realize you need a cable and a programming application, I'll report back on this
another time.

Lets program our first repeater, this is for my local one which is closest to my house.

```
[VFO/MR] # Puts it in Frequency Mode
[1 STEP]-[4 VOX]-[6 ABR]-[6 ABR]-[4 VOX]-[O SQL]-[Menu] # to tune into 146.60
[Menu]-[2 TXP]-[6 ABR]
[Menu]-[0 SQL]-[0 SQL]-[0 SQL]-[6 ABR]-[0 SQL]-[0 SQL]-[Menu] # to set offset freq 000.600
[Menu]-[2 TXP]-[5 WN]
[Menu]-[Down Arrow]-[Menu] # to set the frequency direction of -
[Menu]-[1 STEP]-[3 SAVE]
[Menu]-[1 STEP]-[6 ABR]-[2 TXP]-[2 TXP]-[Menu] # to set off the TX-CTCS to 162.6Hz
[Menu]-[2 TXP]-[7 TDR]
[Menu]-[WHATEVER CHANNEL YOU WANT]-[Menu] # Save the channel so you don't have to do this again
[Menu]
```

You should be good now, 73!
