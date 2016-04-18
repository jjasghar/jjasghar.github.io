---
layout: post
title: "Because googling failed me for yes or no bash scripting"
date: 2014-01-18 14:01:51 -0600
comments: true
categories: bash sysadmin linux
---

So I was lazy, like all good sysadmins...I wanted to put in an option to say "yes or no" in a bash script.

I started with a simple `if..then..elif..else` statement, then I started googling around and found multiple ways to do it.

I ran the script a couple times, it worked like a charm, but without thinking about it I put `y` instead of the suggested `yes` and it `exit 1`ed me.  Do'h!

I continued googling around and then remembered the `case` statement.

I created this:

```bash
#!/bin/bash

read -p "yes or no: " RESPONSE

case "$RESPONSE" in
  yes|y|Yes|Y)
    echo "blah is yes"
    ;;
  no|n|No|N)
    echo "blah is no"
    ;;
  *)
    echo "You need to say yes or no, start over!"
    exit 1
    ;;
esac
```

Now you are probably wondering why I even bothered posting this. Honestly, I spent way too much time on this and I figured I'll find myself looking for this again 6+ months down the line.

Hopefully this post will save you some time in the future.
