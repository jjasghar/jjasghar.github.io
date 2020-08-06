---
layout: post
title: "bash Environmental Variables Override"
date: 2020-08-06 11:32:58
categories: sysadmin linux
---

If you don't know, environmental variables are used to override options for scripts,
apps, and other things you run on computers. In `bash` they normally look like,
something like this:
```bash
DEPLOY=production ./release-app.sh
```
In this contrived example, the `DEPLOY` option is set to `production` and any
options that are required in `./release-app.sh` would do make those changes.

Now, I was experimenting with some scripts lately, and discovered something
I hadn't expected. Lets walk through an example that will hopefully explain
it all.

## Example

Ok, lets say we have this following script:
```bash
FOO="in the script"
echo "This was outputted from" ${FOO} 
```
It outputs:
```bash
$ bash script.sh
This was outputted from the script
```
Perfect, now lets override this:
```bash
$ FOO="from the shell" bash script.sh
This was outputted from the script
```
What! This wasn't what expected. I wanted `This was outputted from the shell`.
Ok, lets try something different:
```bash
$ export FOO="from the shell"
$ bash script.sh
This was outputted from the script
```
Damn, this didn't work either.
Looks like we need to edit the script a bit, I removed the `FOO` line and changed:
```bash
echo "This is outputted ${FOO:-"from the script"}
```
This gives the `FOO` variable the default of `from the script` if it's not declared:
```bash
$ bash script.sh
This was outputted from from the script
$ FOO="from the shell" bash script.sh
This was outputted from from the shell
$ export FOO="from the shell"
$ bash run.sh
This was outputted from from the shell
```
OK this seems more reasonable, now I can change the variables around if I want to. 

One last experiment, and hopefully this should make it make total sense on the 
scoping of environmental varibles in `bash`.

Let's take it one step farther, lets say we have a file of `VAR`s that we source
then run the script:
```bash
FOO="from a file"
```
So we edit our script one more time:
```bash
source ./vars
echo "This is outputted ${FOO:-"from the script"}
```
What will happen now?
```bash
$ bash script.sh
This was outputted from a file
$ FOO="from the shell" bash run.sh
This was outputted from a file
```

Well, that's interesting. So even on the command line, using `source` you'll still
get the most "local" decaration of the `VAR` in `bash`.

Hopefully this show's the scope of how `bash` scopes it's variables, and if you're
like me, and like to have options to run scripts, this'll help you in the future.
