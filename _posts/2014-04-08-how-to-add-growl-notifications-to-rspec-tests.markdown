---
layout: post
title: "How to add growl notifications to rspec tests"
date: 2014-04-08 10:45:59 -0500
comments: true
categories: ruby rails
---

With some pushing from my work I've decided to start learning rails, with rails comes TDD.
I'm reading some books and watching some screencasts, and rspec testing came up as the default testing way.  I'm a
huge [test-kitchen](http://kitchen.ci), or [bats](https://github.com/sstephenson/bats) or the like tester with chef
but never got a way to automate the testing in the background. Turns out using rspec+guard+growl allows for you to
be focused on your present screen and have the tests being run with a pop up.

I use vim as my development platform. This is why I needed a way to run rspec in the background, hopping out of the vim buffer breaks my focus. So I used guard-rspec, it was perfect for what I wanted to do, problem was it was in another tab or terminal again breaking my focus. I figured I have growl installed, so this is how I set up this perfect rspec+guard+growl setup.

First add these two gems to your Gemfile:
```ruby
group :development, :test do
    gem ...
    gem 'growl'
    gem 'growl_notify'
end
```

NOTE: I suggest adding this to the development/test group, you won't want this in your production deploy.

Go ahead and run `bundle`.

After that, you need to install something called [GrowlNotify 2.1](http://growl.cachefly.net/GrowlNotify-2.1.zip), I'd double check your growl version, it only works with 2.1.x.

After installing spin up `guard`, and you should see a nice notification. :)
