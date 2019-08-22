---
layout: post
title: "bats-core and Learning to Test"
date: 2019-06-26 15:41:17
categories: sysadmin bash
---

As a traditional System Administrator you write a lot of `bash`. That's just a
fact of life. Hopefully you've taken a look at a previous blog article I've
written about [shellcheck][shellcheck] to help standardize on your formatting
of your `bash` scripts.

Now that you're comfortable running a linter, the next step is to verify your
changes. There's a handful of options of this, not limited to [InSpec][inspec]
or [ServerSpec][serverspec], but I'm going to focus on a natural progression
testing framework called Bash Automated Testing System or [bats-core][batscore].
There's some history about this project, but I'm going to skip that for now, and
focus on some practical examples of why spending some time using this framework
can make your life easier in the long run.

We are going to take the concept of Test Driven Development here, if you have
developer friends, it's a well accepted way to write code now. There's no reason
why we can't adopt this for our System Administration of our machines. The idea
is pretty simple, you write your test first, then you write your code. If you've
heard of the "Red Green Interate" process it's the same thing. First you write a test that
fails then you write your code to make it pass, then you can interate on it to
make it easier or cleaner if desired. You are probably wondering why you would
bother with this, simply said in "6 months" you can't say with full confidence
that you'll remember exactly what your script does. (I used 6 months as a generic
time, but the easiest thing to say is that it's your "future self.")

The process may seem odd, or even doubling up work, but it pays off in dividends.
To have a security blanket of knowing what you expect to happen will happen without
you debugging is freeing. Imagine the ability to run just one command and have a
sanity check, that's what `bats-core` can bring you; so lets figure out how to
make it happen!

## Writing Your first test

Lets start off with something simple. Ideally you've installed `bats-core` if not
take a look [here][install]. It's pretty straight forward, though I strongly suggest
getting the newest release possible, I've discovered that the distribution packages
are pretty out of date.

Our first test is going to be something that is insanely redundant, but it shows off
the power of bats from the get go. Create a working directory and create a file with
something like the name of `test.bats`.

Put the following into the file, and I'll walk you through what we've done.
```bash
@test "verify bash version" {
  run bash --help
  [ "$status" -eq 0 ]
  [ "${lines[0]}" = "GNU bash, version 4.4.20(1)-release-(x86_64-pc-linux-gnu)" ]
}
```

First off, the `lines` command may be different for you, but you should get the point.
This test will now run the `bash --help` command check that it exits `0` and that the
first line has that specific version of `bash`.

You can run it with a simple:
```bash
$ bats test.bats
 ✓ verify bash version

1 test, 0 failures
```

Success! Congratulations you've written your first test and now know how get `bats-core`
to read output. Now lets move to a more _real_ example.

## Red-Green Interate

In the real world, you'll want to write out your tests first to fail, like discussed above.
Lets write out a test that fails first. I want to make sure that I have a handful of tools
I use on every machine available for me. Lets say, `vim`, `inspec`, `shellcheck`,
`gnome-terminal`, and `zoom`.

Just like above, I write some tests to "verify" that the each file is there. Something
like this:
```bash
@test "verify vim version" {
  run vim --version
  [ "$status" -eq 0 ]
  [ "${lines}" = "VIM - Vi IMproved 8.0 (2016 Sep 12, compiled Jun 06 2019 17:31:41)" ]
}

@test "verify inspec version" {
  run inspec --version
  [ "$status" -eq 0 ]
  [ "${lines[0]}" = "3.5.0" ]
}

@test "verify shellcheck version" {
  run shellcheck --version
  [ "$status" -eq 0 ]
  [ "${lines[0]}" = "ShellCheck - shell script analysis tool" ]
}

@test "verify gnome-terminal version" {
  run gnome-terminal --version
  [ "$status" -eq 0 ]
  [ "${lines[0]}" = "# GNOME Terminal 3.28.2 using VTE 0.52.2 +GNUTLS -PCRE2" ]
}

@test "verify zoom version" {
  run zoom --version
  [ "$status" -eq 0 ]
  [ "${lines[0]}" = "ZoomLauncher started." ]
}
```

Now I ran this on my laptop I'm writing out this blog post. This is what I got:
```bash
~ > bats test.bats
 ✓ verify vim version
 ✓ verify inspec version
 ✗ verify shellcheck version
   (in test file test.bats, line 15)
     `[ "$status" -eq 0 ]' failed with status 127
 ✓ verify gnome-terminal version
 ✓ verify zoom version

5 tests, 1 failure

~ >
```

So it seems I already have the tools I expected apart from `shellcheck`, I go ahead
and run the command `shellcheck --version` and low and behold I don't have it installed.

I install in, and run my test again...and get...
```bash
~ > bats test.bats
 ✓ verify vim version
 ✓ verify inspec version
 ✓ verify shellcheck version
 ✓ verify gnome-terminal version
 ✓ verify zoom version

5 tests, 0 failures

~ >
```

Now this seem obvious after reading it, but if you start adding this to your thought process
you'll be able to write `.bats` files for different applications and situations. You won't
have to worry that when you install that application, was the file in the correct place,
or if you have a in house built application, to run a `verify.bats` against a server to
make sure it's what you expect to be there.

Take a moment and think about setting up a machine for your Development team. They
may want a specific version of `Java`, and `Tomcat`, or `ruby` with `passenger` on
the machine. Yes, you should be using configuration management to control these options,
but if you had a `bats` test to quickly verify state, it can save time and effort.

This is extremely powerful, and spending sometime to learn it will only benefit you.


[batscore]: https://github.com/bats-core/bats-core#installation
[inspec]: https://inspec.io
[install]: https://github.com/bats-core/bats-core#installation
[serverspec]: https://serverspec.org/
[shellcheck]: https://jjasghar.github.io/blog/2019/06/15/shellcheck-and-you-should-too/
