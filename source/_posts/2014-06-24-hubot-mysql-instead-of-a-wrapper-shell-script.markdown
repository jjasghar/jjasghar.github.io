---
layout: post
title: "hubot mysql instead of a wrapper shell script"
date: 2014-06-24 10:40:32 -0500
comments: true
categories: hubot sysadmin mysql
---

So you've gotten hubot to do some cool things. You've even got it to shell out to run some
useful commands and scripts. (An example would be [hubot-chef](http://github.com/hubot-scripts/hubot-chef/blob/master/src/chef.coffee)
shelling out and running knife.) So you've started playing with pulling data from
mysql or the like, and you've come up with something like the following.

```sql
mysql -u$user -p$pwd -h $host -N -e "select name, id from mydatabase"
```

Pretty straight forward eh? You run it on your cli and yeah you get the data you expect. You attempt to run it with hubot maybe something
like this?

```coffee
 robot.respond /database name and id$/i, (msg) ->
   msg.send "determining the name and ids"
   exec "bash /home/hubot/bash_scripts/name_id_database.sh", (err, stdout, stderr) =>
     msg.send stdout
```

It might come out all gross, yeah that's a different post about [cli-table](https://github.com/LearnBoost/cli-table) which I hope i'll be
doing after this post.  Anyway i digress; that's great, so you get the data hubot gives you the date, but man you have your password in your
shell script, you have to shell out to do it, and honestly, it seems a tad bit hacky right?  Luckly, [Matt Bridges](https://twitter.com/mattdbridges) showed
me how to leverage coffeescript and the [mysql npm](https://www.npmjs.org/package/mysql) package to do just that.

First off, you'll need to add to your `package.json` something like:

```json
 "dependencies": {
          ....
          "mysql": ">= 2.0.1",
          ....
```

At the top of your coffee script add something like this, it will open up the ability to start calling the commands.

```coffee
  mysql = require 'mysql'
```

Awesome, now lets convert that top sql to coffee, first thing you need to do is create the connection:

```coffee
  connection = mysql.createConnection
    host: 'mydbhostname'
    user: 'myuser'
    password: process.env.DB_PASSWORD
```

As you can see it's pretty self explanatory. You create the object called `connection` then give it some variables. The password is interesting here
it's now is a environment variable that you can just add to your hubot (however you choose, like in heroku: `heroku config:set DB_PASSWORD=a_really_strong_pa$$word`) so
you don't have to have it checked into you scm/code.

After this, now you'll want to do something like:

```coffee
    sql = "select name, id from mydatabase"
    sql = mysql.format(sql)
    connection.query sql, (err, results) ->
      throw err if err
      for row in results
        msg.send row
```

This also is pretty self explanatory, you put your sql statement in as a variable, format it with mysql, open the connection via the query command, then
if it errors, throws the error, otherwise outputs each row and a seporate `msg.send`.

Yes, yes, this is dirty, gross and if you have anything more than 3-4 users you're gonna get annoyed really fast. Again, this is just an example, the cli-table tutorial
will make this more enjoyable.
