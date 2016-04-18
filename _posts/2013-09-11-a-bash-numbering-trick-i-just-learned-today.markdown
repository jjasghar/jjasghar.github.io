---
layout: post
title: "a bash numbering trick I just learned today"
date: 2013-09-11 13:05
comments: true
categories: bash linux
---

I've been using bash for...a really really long time.  I've always done `seq 1 10` in my for loops to get `1 2 3 4 5 6 7 8 9 10`.  Today I learned these three ways do the exact same thing. (Note: bash v4 above) I'm writing this out, mainly because I want to remind myself of this again one day.

```bash
for ((i=1; i<=10; i++)); do echo $i; done

for i in `seq 1 10` ; do echo $i; done

for i in {1..10} ; do echo $i; done
```

Dang you really do learn something new every day.

Note: A good friend of mine [awaxa](https://github.com/awaxa/) pointed out that {1..10} is the fastest in this case because it's a built into bash.  Thanks awaxa!
