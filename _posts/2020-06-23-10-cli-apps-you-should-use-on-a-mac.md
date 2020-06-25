---
layout: post
title: "10 CLI Apps you should use on a Mac"
date: 2020-06-23 14:59:38
categories: sysadmin devops development
---

Hi! So you've found my list of the 10 CLI Applications that I think every modern
developer should use on a Mac. These are the commands that I have found myself use
day in and day out, and normally don't even think about using I just use them.

I do have to say I had to capture them on a notepad over a week or two to see how
often I did, and was surprised on the consistency I did use them. OK lets go!

## iTerm

Alright, alright, I know I'm cheating here. [iTerm][iterm] isn't actually a CLI
application, but an interface _to_ your CLI applications. You may have found the
`Terminal` under `Applications > Utilities > Terimial.app` but honestly it's very
static. `iTerm2` is unbelievably powerful, and should become your main way to CLI
apps. There's tons of options, and knobs and dials to turn. I strongly suggest having
the quick tips enabled, and just use it; before you know it, it becomes your second
home.

## brew

The first real CLI app is the way to the world of CLI apps out there. [brew][brew] is
the way to install many applications in the Mac ecosystem. If you know anything
about Ubuntu or Red Hat Linux, you may know of `apt` or `dnf` package managers,
`brew` is Mac version.

In order to install `brew` you have to bring up a terminal (*cough* iTerm *cough*)
and run the following:
```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install.sh)"
```
After this you'll have the `brew` command and you can start installing applications.
One of the first apps you should install, is `wget` which grabs files from the internet
and puts them places. Run the following:
```bash
brew install wget
wget https://google.com/blank.html
open blank.html
```
You're web browser should open with the famous [blank.html][blank] from Google!

## zsh (oh-my-zsh)

The newest versions of MacOS `zsh` became the main shell. There is a wrapper called
[oh-my-zsh][ohmyzsh] that sets your command prompt to 11. There are a ton of themes,
plugins, and shortcuts to make you typing things out so much faster. Whatever language
you're planning on using, there are helpers in `oh-my-zsh`, whatever infrastructure
you're planning on using there's plugins and helpers for that too.
Installing it is pretty straight forward too:
```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```
I believe it installs `git` but default too, which you should probably have installed
also. :)

## tmux

[tmux][tmux] is something I wish I had started to use from the get-go. To quote the
main page:

> tmux is a terminal multiplexer. It lets you switch easily between several programs in one terminal, detach them (they keep running in the background) and reattach them to a different terminal.

It's a way to interface with your terminal sessions via the keyboard and split your
windows to test multiple things. Take a moment and think of this process, without
`tmux` if you had your code in one window, and a terminal in the other running
your code out putting logs. Lets say you needed to bring up another command prompt
to run something else. You'd have to either make another tab, or a new window taking
you away from the running application.
With `tmux` all you have to do is `Control-b c` to create a new terminal, or 
`Control-b "` to split under the running command. With a little bit of practice
it becomes second nature and you can work at the speed of thought. (Yes that's
a `vim` reference.)

## ag

[the_silver_searcher][ag] or `ag` is an application that most don't find till much later
in their career. It's a _super_ fast way to search through files for a string.
So take a moment and imagine this: You have a code base and you need to find
every line that has the word `foo` in it. With `ag` it is just,
```bash
cd src/ # assuming the code is in src/
ag foo
...list-o-files-here...
...list-o-files-here...
...list-o-files-here...
```
It can do so much more, but use `ag` and the next app...oh nelly, you can make
some super fast changes...

## sed

`sed` is something everyone should learn to use as soon as they start using the CLI.
`sed` is actually a programming language, and unbelievably powerful but 95% of the
times you'll use it it'll be:
```bash
sed -i '' s/foo/bar/g file.txt
```
Now, if you've come from Linux and notice the `-i ''` and wonder why it's there.
That's actually the main reason why I started writing this blog post, on a Mac, it's
actually BSD `sed` which requires the `-i ''` flag for backup. If you put `-i .bak`
for instance, you'd change `foo` to `bar` in file.txt, but you'd also have a file.txt.bak
created in original state. `-i ''` tells `sed` to skip the creation of the backup
Now lets take `ag` and play with some `sed`.
Take our fake `src` directory...
```bash
for i in $(ag -l foo src/)
do
  sed -i '' s/foo/bar/g $i
done
```
So to walk through this example, we use `ag` to search all our `src` directory outputting
the files that have `foo` in them. Then we push that through `sed` and changes
every instance of `foo` to `bar`.
And that's just a tip of an iceberg of other programmatic things you can do plugging
these two apps together.

## awk

`awk` is another programming language, but basic usage it can be glued together
with both `ag` and `sed` to make your life much much better. You can pull data
out of files or `csv`s very easily with `awk`. Take the example here:
```bash
$  ~ cat file.csv
1,dog,brown,large
2,cat,white,small
3,fish,orange,tiny
$  ~ awk -F ',' '{print $2}' file.csv
dog
cat
fish
```
If I had this `file.csv` and I just wanted the second column, using `awk` with the
`-F ',' '{print $2}'` where it splits on the `,` and then takes the second
column. There are tons of options, but now that you have that string/list, you
could push it in with an `for` loop and then push it through `sed` changing 
what you need. The options are truly limitless, and again with just a little
practice you can start thinking of really neat ways to programmatically doing things.

## alias

`alias` is something that you'll find yourself using over and over. It aliases commands
to simple short things to you don't have to type out massive things/remember. There are
a ton of aliases you can use, and here are some ones I use on a daily basis.
```bash
alias v=vim
alias k8s='ibmcloud ks cluster config --cluster MY_K8S_CLUSTER'
alias k=kubectl
alias workshop='ibmcloud login --apikey o4h6IcZIM_A_FAKE_API_KEYK9HC1cWmDskAxbYz9HUH3c'
```
So let me walk through these 4.
1. `v` aliases to `vim`. Now if i need to edit a file, I can just do `v /etc/hosts` or
the like and get the file up. It might seem ridiculous at first, but bringing your 
index finger down and then tab completing a file over and over is much much faster
then index finger down, middle finger up, index finger down. Take a moment and see
the difference.
2. `k8s` is my way to make sure my `KUBECONFIG` is pointing at the correct Kubernetes
cluster I use. This is just an example of a pretty regular "login" command for a 
specific server, and you can envision what you could grow off it.
3. `k` for `kubectl`, I use `kubectl` all the time. Just like `v` for `vim`, not
having to tab complete `kubectl` allows me to think of what I'm doing not _how_
to do it.
4. Finally, `workshop`, which is another login command. As you can see you
can string larger commands together and even use crazy API keys and things
like that in aliases.
Hopefully this has inspired you to look at your `history` and see those commands
you type over and over, and ideally come up with some `aliases` for yourself.

## jq / yq

The final command, *cough* commands *cough* is [jq][jq] and [yq][yq]. I'm bundling
them together because they are two different interfaces to for two different
types of files, but in essence does the same thing. (I believe `jq` inspired `yq`
creation) `jq` is for parsing `json` documents/files, while `yq` is for `yaml`
files. As you start doing more and more web development you will start needing
to find information in both `yaml` and `json` docs' and `jq` and `yq` are there
to make them more human readable (colored and whatnot) and even parse specific
sections to manipulate or report. If you haven't needed them yet, don't worry,
you will, and it'll become something like `ag` that you use without thinking.


## Bonus: Brewfile

So if you've made it this far, either I've hit a cord with this post, or
maybe you're just curious what an old Sys Admin like myself uses. If you took
my advice hopefully you've installed `brew` by now. If you went along and
`brew` installed each good for you, but I wanted to get something repeatable,
so I've created a _simple_ `Brewfile` for you.
Go ahead, open your favorite text editor, make the file called `Brewfile`
copy the following into it, and run:
```bash
brew bundle
```
You'll get all the CLI apps I talked about above, and now you can grow
your `Brewfile` to what you find in the future.

```
tap "bazelbuild/tap"
tap "homebrew/bundle"
tap "homebrew/cask"
tap "homebrew/cask-versions"
tap "homebrew/core"
tap "homebrew/services"
tap "nektos/tap"
brew "jq"
brew "kind"
brew "the_silver_searcher"
brew "tmux"
brew "yq"
brew "zsh"
```

[iterm]: https://www.iterm2.com/
[brew]: https://brew.sh/
[blank]: https://google.com/blank.html
[ohmyzsh]: https://ohmyz.sh/
[tmux]: https://github.com/tmux/tmux/wiki
[ag]: https://github.com/ggreer/the_silver_searcher
[jq]: https://stedolan.github.io/jq/
[yq]: https://mikefarah.gitbook.io/yq/
