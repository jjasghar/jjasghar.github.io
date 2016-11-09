---
layout: post
title: "hashicorp vault errors"
date: 2016-11-08 16:55:57
categories: linux hashicorp security
---

### Authentication

I started to play with [vault][vault] for my job. Here are some errors I had
trying to learn using it.

```shell
16:51:59 JJs-MacBook-Pro ~ > vault write secret/hello vault=world
Error writing data to secret/hello: Error making API request.

URL: PUT http://127.0.0.1:8200/v1/secret/hello
Code: 400. Errors:

* missing client token
```
As you can see I get a `missing client token` error. This is due to me
not authenticating with the vault, this was resolved via:

```shell
16:53:19 JJs-MacBook-Pro ~ > vault auth
Token (will be hidden):
Successfully authenticated! You are now logged in.
token: da435292-0882-df8d-633f-79a50945e1bc
token_duration: 0
token_policies: [root]
16:55:12 JJs-MacBook-Pro ~ > vault write secret/hello vault=world
Success! Data written to: secret/hello
```

### Notes

It seems if you attempt to write to a key, for instance: `secret/hello`
you write over it.

```shell
16:55:12 JJs-MacBook-Pro ~ > vault write secret/hello vault=world
Success! Data written to: secret/hello
16:55:16 JJs-MacBook-Pro ~ > vault read secret/hello
Key             	Value
---             	-----
refresh_interval	768h0m0s
vault           	world
17:00:50 JJs-MacBook-Pro ~ > vault write secret/hello this=something
Success! Data written to: secret/hello
17:00:54 JJs-MacBook-Pro ~ > vault read secret/hello
Key             	Value
---             	-----
refresh_interval	768h0m0s
this            	something
```
