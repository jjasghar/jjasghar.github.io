---
layout: post
title: "Skipping phases in habitat"
date: 2016-07-14 12:19:31
categories:
---

This is how to skip a build phase in a `plan.sh` in [habitat][habitat].

```bash
do_install() {
    # I have no need for to install this app ;)
    return 0
}
```

Note the `return 0`, not and not do `exit 0`. If you do the exit, you'll end the
plan completely.

[habitat]: https://www.habitat.sh/
