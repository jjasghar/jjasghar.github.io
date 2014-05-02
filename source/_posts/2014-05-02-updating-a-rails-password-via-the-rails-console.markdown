---
layout: post
title: "Updating a rails password via the rails console"
date: 2014-05-02 11:28:40 -0500
comments: true
categories: rails ruby sysadmin
---

I'm in the process of learning [ruby on rails](http://www.railstutorial.org/) and it's going amazingly, plus [Michael Hartl](http://michaelhartl.com/) as a great teacher, I strongly suggest
going through his screencasts.

As I was watching his screencasts, I forgot one of my test passwords and realized I didn't have a quick way to redo the password. There is a simple handful of commands fix this:

```ruby
irb(main):001:0> u = User.find_by_id(1)
  User Load (3.5ms)  SELECT "users".* FROM "users" WHERE "users"."id" = 1 LIMIT 1
=> #<User id: 1, name: "JJ Asghar", email: "jjasghar@gmail.com", created_at: "2014-04-18 17:46:48", updated_at: "2014-04-25 23:21:10", password_digest: "$2a$10$or/wJw1I8Wpf0lXNbtawveoQXETGJbUkv/VwXQXzn92r...", remember_token: "uUJTExCyt4mpAOpQWd4PMA">
irb(main):002:0> u
=> #<User id: 1, name: "JJ Asghar", email: "jjasghar@gmail.com", created_at: "2014-04-18 17:46:48", updated_at: "2014-04-25 23:21:10", password_digest: "$2a$10$or/wJw1I8Wpf0lXNbtawveoQXETGJbUkv/VwXQXzn92r...", remember_token: "uUJTExCyt4mpAOpQWd4PMA">
irb(main):003:0> u.password = "a_stupid_pa$$word"
=> "_stupid_pa$$word"
irb(main):004:0> u.password_confirmation = "a_stupid_pa$$word"
=> "_stupid_pa$$word"
irb(main):005:0> u.save
   (2.0ms)  BEGIN
  User Exists (12.2ms)  SELECT 1 AS one FROM "users" WHERE (LOWER("users"."email") = LOWER('jjasghar@gmail.com') AND "users"."id" != 1) LIMIT 1
   (15.8ms)  UPDATE "users" SET "password_digest" = '$2a$10$Za7prnrSthhE90AvB15BNOMl8sf3oL7onBpZ45nH/9skwHfn1EFA.', "remember_token" = 'zOZYjEVovO143qrX1yhibw', "updated_at" = '2014-04-25 23:24:39.329502' WHERE "users"."id" = 1
   (4.6ms)  COMMIT
=> true
irb(main):006:0>
```

As you can see, you need to create an object via a `find_by_<something>`, in my case `id` and write it to `u`. After that I change the password key to `a_stupid_pa$$word` along with the password_confirmation. I then write it to the
database via `u.save`.

You can also do this for anything that the object has, you just have to make sure that you follow the constraints you put in place. If you see an issue with the save, if it comes back as `false`, put a `!` at the end and it should bubble up the
exception telling you why it won't work, example:

```ruby
[1] pry(main)> u = User.find_by_id(1)
  User Load (0.1ms)  SELECT "users".* FROM "users" WHERE "users"."id" = 1 LIMIT 1
=> #<User id: 1, name: "JJ Asghar", email: "jjasghar@gmail.com", created_at: "2014-04-18 17:12:49", updated_at: "2014-04-24 20:17:57", password_digest: "$2a$10$GeLuTvSL4giPYJWIozdj6e1UxrxM0NauI9ilXoB9pZDs...", remember_token: "ZiAb_plRQdp9WaKjbZ4CAA">
[2] pry(main)> u.password = "1234"
=> "1234"
[3] pry(main)> u.save
   (0.1ms)  begin transaction
  User Exists (0.1ms)  SELECT 1 AS one FROM "users" WHERE (LOWER("users"."email") = LOWER('jjasghar@gmail.com') AND "users"."id" != 1) LIMIT 1
   (0.1ms)  rollback transaction
=> false
[4] pry(main)> u.save!
   (0.1ms)  begin transaction
  User Exists (0.1ms)  SELECT 1 AS one FROM "users" WHERE (LOWER("users"."email") = LOWER('jjasghar@gmail.com') AND "users"."id" != 1) LIMIT 1
   (0.0ms)  rollback transaction
ActiveRecord::RecordInvalid: Validation failed: Password is too short (minimum is 6 characters), Password confirmation can't be blank
from /Users/jasghar/.rvm/gems/ruby-2.0.0-p195/gems/activerecord-3.2.16/lib/active_record/validations.rb:56:in `save!'
[5] pry(main)>
```
