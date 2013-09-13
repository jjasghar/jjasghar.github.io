---
layout: post
title: "Learning sinatra and where I'm going from here"
date: 2013-09-13 15:29
comments: true
categories: ruby sinatra
---

I'd like to start off by thanking Darren Jones, for writing the book [Jump Start Sinatra](http://sitepoint.com/jumpstart-sinatra).  This book is amazing, it's extremely well written, straight forward and hands on. Darren was able to break down and explain how sinatra works, runs and what you can do with it in a rapid fire primer.


Darren has you create a simple "sinatra song web site" with a database backed, css, javascript (via coffeescript), and even gets you to post to heroku.  The book is only about 166 pages long, and fast, but gives you the primer to understand what you need to get done to make a functional app with little trouble and overhead. With in the first couple chapters Darren spurred me to wonder how a json object with curl POST was able to be put the data into a db, and with the this tutorial Darren has opened the avenue for me to be able to figure it out.


One issue that I had with the book, it doesn't you how to write tests against the app you create.  Due to new programming practices, this seems like a nasty gap.  I've decided before I figure out how to get my json input output working, I'm going to figure out how to write tests against the app I created.


If you are curious on the app that is built from this book, I have posted it [here](https://github.com/jjasghar/sinatra_song_app), but as time progresses I'll probably add things to it. Fair warning there.
