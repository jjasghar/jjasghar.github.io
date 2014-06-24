---
layout: post
title: "hubot mysql cli-table tutorial"
date: 2014-06-24 13:09:23 -0500
comments: true
categories: hubot sysadmin mysql
---

So you get some output in a fun like the previous post with mysql.  If you are just looking I'm taking the example like:

```js
    sql = "select name, id from mydatabase"
    sql = mysql.format(sql)
    connection.query sql, (err, results) ->
      throw err if err
      for row in results
        msg.send row
```

So you've probably noticed that it comes out HORRIBLY. This tutorial will leverage the [cli-table npm](https://www.npmjs.org/package/cli-table) so it _should_ look
something like:

```
//╔══════╤═════╤══════╗
//║ foo  │ bar │ baz  ║
//╟──────┼─────┼──────╢
//║ frob │ bar │ quuz ║
//╚══════╧═════╧══════╝
```

So lets make this happen.  First off just like the mysql plugin you'll need to add something like this to your `package.json`.

```json
"dependencies": {
    ...
    "cli-table": "latest",
    ...
    }
```

After this you need to create an object like the mysql plugin...

```js
Table = require "cli-table"
```

Now you have `Table` available to you so you can leverage it to make the tables.  First thing you need to do is create the actual table which is this line:

```js
    table = new Table({head: ['id', 'name'], style: {head:[], border:[], 'padding-left':1, 'padding-right': 1 }})
```

Then at the `connection.query` line you'll want to push the rows into the table loop through them then in _one_ message push it out, which is what the next snippet 
does.

```js
    connection.query sql, (err, results) ->
      throw err if err
      for row in results
        table.push [row.id, row.name]
      msg.send table.toString()
```

So the full thing would be something like the following:

```js
  robot.respond /database user id$/i, (msg) ->
    table = new Table({head: ['id', 'name'], style: {head:[], border:[], 'padding-left':1, 'padding-right': 1 }})
    msg.send "Looking on your names and ids"

    connection = mysql.createConnection
      host: 'mydbhostname'
      user: 'mydbuseraccount'
      password: process.env.DB_PASSWORD

    sql = "SELECT id, name, FROM mydatabase"

    connection.query sql, (err, results) ->
      throw err if err
      for row in results
        table.push [row.id, row.name]
      msg.send table.toString()
```
And boom, you now have a nice readable outputted chart that hubot can answer for you when you need. :)
