---
layout: post
title: "shellcheck and You Should Too"
date: 2019-06-15 13:45:09
categories: sysadmin bash
---

I'll start off by saying it's been a while since I've been a traditional sysadmin.
I've transitioned over the last few years from "Production Engineer" to "DevOps
Engineer" then to a talking head AKA Developer Advocate. (That's supposed to be
tongue and cheek, but keeping technical as a Developer Advocate is HARD.) I do still
write some code and scripts, but all in all I don't have a production environment
to care and feed for. I'll say this, I miss it, and maybe one day I'll go back to
help run a team, but right now I'm not in the right career place.

But I digress; this post is about [shellcheck][shellcheck_site] and the power
that can come of it. As a Developer Advocate, I write a lot of code that glues
things together, being I spend my time on Linux and Mac, `bash` is the obvious
choice. I started talking to people at different conferences I go to, and the idea
of [linting code][linting] is still considered a Developer practice instead of
an Operator practice. I'm here to try to set the record straight.

## Why?

First thing first, *why*. Standardizing on coding practices, and yes you now code,
will save you from insane overhead and tech debt in the future. When your teams'
`bash` scripts start to look the same,
it's easier to read, and with that, people start paying attention to what's happening
instead of where the `do` or the `;` is; `shellcheck` enforces this. I should say,
you can over ride these in `shellcheck` but that's a team decision and out of scope
of this post. (Read up [here](https://github.com/koalaman/shellcheck/wiki/Ignore) if you
need to go down this path.)

## How?

Now we come to *how*. There's a [handful of ways to get it](https://github.com/koalaman/shellcheck#installing), `apt`, `yum`, `homebrew`
and even a `docker` container can be ran to run this application. It's really easy
to add to your CI/CD pipeline too, pulling out every `.sh` file, and running
`shellcheck` against it.

Adding it to your `Makefile` is simple:

```make
check-scripts:
    # Fail if any of these files have warnings
    shellcheck myscripts/*.sh
```

And even Travis CI, has it built in with just adding this to your `.travis.yml`:

```yml
script:
  # Fail if any of these files have warnings
  - shellcheck myscripts/*.sh
```

Personally I use the `docker` container on my local laptop. I wrote a simple script that
I named `shellcheck.sh` and any time I save a script `.sh` I run `shellcheck.sh script.sh`
against it. As you can see, it's _very_ straight forward.

```bash
#!/bin/bash

if [ $# -eq 0 ]
  then
    echo "You need to add an script to this check, shellcheck.sh script.sh"
    exit 1
else
  if [[ "$(ping www.google.com -c 1 | grep '100% packet loss' )" != "" ]]
  then
    echo "You can't seem to connect to the internet, you should proobly fix that."
    exit 1
  fi
  docker pull koalaman/shellcheck:latest
  docker run -v "$PWD:/mnt" koalaman/shellcheck "$1"
fi
```

It gives me a piece of mind that my code all is standardized and just "works" as expected.
The output of `shellcheck` also tells you exactly why your offending code is, with a dedicated
wiki page [here][shellcheck_wiki] with some very explicit why you shouldn't do it this way.
I applaud the main developer [Vidar Holen][vidar] and team to help make use better
developers.

## Wrap Up

Honestly, I'm just here to convince you, dear reader, to give this linter a shot. The overhead is
minimal, and the benefits are immense. This is one of those things that you shouldn't waste
time on debating, it will just make your life, and your team's life easier. You can get it
into CI, run locally and make sure everyone writes their scripts with a unified pattern with
only a tad bit of work.


**NOTE**: Yes, I ran the above script _through_ `shellcheck` too.

[linting]: https://en.wikipedia.org/wiki/Lint_%28software%29
[shellcheck_site]: https://www.shellcheck.net/
[shellcheck_wiki]: https://github.com/koalaman/shellcheck/wiki/Checks
[vidar]: https://github.com/koalaman
