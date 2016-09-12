---
layout: post
title: "Moving to capistrano version 3 from version 2 git submodules"
date: 2014-06-04 17:13:43 -0500
comments: true
categories: rails
---

I'm in the process of moving from [capistrano](http://capistranorb.com/) version 2
to capistrano version 3. This is a post of a nasty hiccup I had, hopefully it'll be
useful for someone else.

## git submodules

I had a requirement to get git submodules to work with one of my applications. It's seems
that v3 doesn't have the "clone" version anymore, so the `.git` directory isn't there.
It seems it uses the `git archive` and doesn't have an "out of the box" support for submodules.
See [here](http://stackoverflow.com/questions/19403138/capistrano-v3-deploy-git-repository-and-its-submodules/19573220#19573220).
I did some searching and found the above stack overflow and  [this gist](https://gist.github.com/jjasghar/d9369c60c51f79d6e7cc)
which gave me the bulk of what I needed to get done.

There were some gotchas on this so I'm going to do my best to document them here.
First off, as you can see on [line 20](https://gist.github.com/jjasghar/d9369c60c51f79d6e7cc#file-submodule_strategy-rb-L20)
there is a new variable named `repo_path`.  This is due to the need now of the
`git clone` to the file system and a place it needs to put it. So that means you need
to add the `repo/` directory in your `:deploy_to` directory. Might I suggest the something like:

```bash
ssh <deployuser>@<remotehost>://var/www/<app>/repo/
```

Then after the fact, you might want to steal this cap rake task:

```ruby
desc "Check that we can access the repo directory"
task :check_repo_permissions do
  on roles(:all) do |host|
    if test("[ -w #{fetch(:deploy_to)/repo} ]")
      info "#{fetch(:deploy_to)/repo} is writable on #{host}"
    else
      error "#{fetch(:deploy_to)/repo} is not writable on #{host}"
    end
  end
end
```

Also may I suggest you putting this at the "top" of your `deploy.rb` something like:

```ruby
before :deploy, :check_repo_permissions
```

So just in case you're `cap` task will fail out early on of you have decided to remove, or
do something with that repo directory.

Now if you go ahead and run `cap staging deploy` (you're testing in STAGING right?)
You might run into some errors. Most are pretty self explanatory, and if you take care of them
and run `cap` again, you should be good...but I digress.

After adding that repo directory, you want to add the above [gist](https://gist.github.com/jjasghar/d9369c60c51f79d6e7cc) to `lib/capistrano/submodule_strategy.rb`
and add the following two lines to your `Capfile`:

```ruby
require 'capistrano/git'
require './lib/capistrano/submodule_strategy'
```

After that, you need to add to your `config/deploy.rb`:

```ruby
set :git_strategy, SubmoduleStrategy
```

This should wire everything up now.

There is one other _optional_ file, `.capignore` that needs to be created, it is a `.gitignore` for this cap deployment.
I ended up taking the example:

```
.capignore
lib
config
.git
```

and adding it directory to the `(:deploy_to)/repo/.capignore`. I believe you can add it to your the root of your repo,
and it should get there too, but I didnt do it that way. The only place it's referanced is [here](https://gist.github.com/jjasghar/d9369c60c51f79d6e7cc#file-submodule_strategy-rb-L45) so if you see fit, you could probably just remove the referance.

You should be able to do a `cap staging deploy` now and you should notice something like:

```bash
...
DEBUG[639d0d29] Command: cd /var/www/app/repo && ( GIT_ASKPASS=/bin/echo GIT_SSH=/tmp/app/git-ssh.sh /usr/bin/env git submodule update --init )
DEBUG[639d0d29]   Submodule 'i-hate-submodules' () registered for path 'i-hate-submodules'
...
```

And you should be good to go!
