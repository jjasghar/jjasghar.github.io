---
layout: post
title: "vault with a consul back end"
date: 2016-11-14 21:24:41
categories: vault sysadmin consul linux
---

# vault with a consul back end

Here are my notes on how I got [vault][vault] working with a [consul][consul] backend,
building with Habitat. I should mention I'm trying to get these plans into core,
the PRs are [285][285] and [286][286].

## Useful links

Lets start with some useful links first:

This link is the offical blog post about using Chef and Vault in tandem:
<https://www.hashicorp.com/blog/using-hashicorp-vault-with-chef.html>

This link is from Sean Walberg leveraging the same thing and his notes.
<https://ertw.com/blog/2016/11/08/managing-secrets-in-chef-with-hashicorp-vault/>

## consul

### Setting up a consul cluster

First thing you need to do: spin up three consul nodes.

Convert the `default.toml` in `consul` from `127.0.0.1`, to `0.0.0.0`, and
`mode = false` with `website = true`

Build the package:

```shell
$ hab studio enter
[1][default:/src:0]# build
[2][default:/src:0]# hab pkg export docker core/consul
```

Run the package(s):

```shell
$ docker run -p 8501:8500 -it core/consul:latest
$ docker run -p 8502:8500 -it core/consul:latest
$ docker run -p 8503:8500 -it core/consul:latest
```

Spin up another container to do some administrative commands:

```shell
$ docker run --privileged -i -t ubuntu /bin/bash
root@402825380ff6#:/# apt-get update && apt-get install curl -y && curl -s -L http://bit.ly/consul-bootstrap | bash
```

Now have the consul cluster be created:

```shell
root@402825380ff6#:/# cd ~/consul
root@402825380ff6#:~/consul# ./consul join -rpc-addr=172.17.0.2:8400 172.17.0.2 172.17.0.3 172.17.0.4
```

Go to <http://localhost:8501/ui/#/dc1/services/consul> to verify everything is there

### Setting up some health checks

Stay on your administrative container:

```shell
root@402825380ff6#:/# apt-get install -y inetutils-ping nginx
root@402825380ff6#:/# mkdir /etc/consul.d
root@402825380ff6#:/# echo '{"check": {"name": "ping",
  "script": "ping -c1 google.com >/dev/null", "interval": "30s"}}'   >/etc/consul.d/ping.json
root@402825380ff6#:/#  echo '{"service": {"name": "web", "tags": ["nginx"], "port": 80,
  "check": {"script": "curl localhost >/dev/null 2>&1", "interval": "10s"}}}'   >/etc/consul.d/web.json
root@402825380ff6#:/# cd ~/consul
root@402825380ff6#:~/consul# ./consul agent -data-dir=/tmp/consul -node=playground -bind=0.0.0.0 -config-dir=/etc/consul.d -join=172.17.0.2
```

Go to <http://localhost:8501/ui/#/dc1/services/consul> to verify your new health check(s).

## vault

### Creating the vault container

First thing you need to do is edit the `default.toml`.
Convert the `default.toml` from `127.0.0.1`, to `0.0.0.0`, and `mode = false`
and location to `172.17.0.2`.

Enter the hab studio and build the `.hart`.

```shell
$ hab studio enter
[1][default:/src:0]# build
[2][default:/src:0]# hab pkg export docker core/vault
```

Start a vault instance pointing to the consul cluster.

```shell
$ docker run -p 8200:8200 -it core/vault:latest
```

Spin up another container if you want to bootstrap vault connections, or use
the above administrative container.

```shell
$ docker run --privileged -i -t ubuntu /bin/bash
root@402825380ff6:/# apt-get update && apt-get install curl -y && curl -s -L <http://bit.ly/2eUDh3H> | bash
root@402825380ff6:/# export VAULT_ADDR='http://172.17.0.7:8200'
root@402825380ff6:/# cd ~/vault
root@402825380ff6:~/vault# ./vault init
```

You should seem something like the following:

```
Unseal Key 1: Nz3hSxDa2Sw5x/kHyIFKn2p+BrfQePri+B3/TJ+g9OYB
Unseal Key 2: 5QXGmgqfGnRboh+xBXDJXf3dgAb7xgTjAiczZk9+YzUC
Unseal Key 3: PfIlI9NYVRF1fGubS2rHHYE6q5AZtZnTCavXLgVsR5ID
Unseal Key 4: Saf1Jude3cLPBEED5wc3yJdjrS/jJrLvuwSKJcK8bC0E
Unseal Key 5: kVAWnz6Zkqfh2jUpqR05iOuEhrkBVS/fsIhubYiuSIoF
Initial Root Token: 6d725951-b18d-caeb-5c51-4ad45eb54af0
```

Next unseal the vault:

```shell
root@402825380ff6:~/vault# ./vault unseal
```

### vault gem

Here are some useful notes on using the vault gem.

```shell
root@402825380ff6:~/vault# curl -L https://chef.io/chef/install.sh | bash -s -- -P chefdk
root@402825380ff6:~/vault# eval "$(chef shell-init bash)"
root@402825380ff6:~/vault# gem install vault
```

When you get the gem installed, open up `pry`

```shell
root@402825380ff6:~/vault# pry
```

The following will connect to your vault instance and create `secret/bacon`.

```ruby
require 'vault'
Vault.address = 'http://172.17.0.7:8200'
# test a failing token first
Vault.token   = "6d725951-b18d-caeb-5c51-bad"
# now put in the correct token.
Vault.token   = "6d725951-b18d-caeb-5c51-4ad45eb54af0"
Vault.logical.write("secret/bacon", delicious: true, cooktime: "11")
Vault.logical.read("secret/bacon")
```

The following will use the `vault` client to list and read `secret/bacon`.

```shell
root@402825380ff6:~/vault# ./vault status
root@402825380ff6:~/vault# ./vault auth
root@402825380ff6:~/vault# ./vault list secret/
root@402825380ff6:~/vault# ./vault read secret/bacon
```

### curl

The following is the way to use the vault API via curl to get things from
vault.

```shell
root@402825380ff6:~/vault# export VAULT_TOKEN>=6d725951-b18d-caeb-5c51-4ad45eb54af0
```

The following will overwrite `secret/bacon` with with `"bar":"baz"`, then query
against it and pipe it into `jq`. I should mention that `jq` isn't required here.

```shell
root@402825380ff6:~/vault# apt-get install -y curl jq
root@402825380ff6:~/vault# curl -X POST -H "X-Vault-Token:$VAULT_TOKEN" -d '{"bar":"baz"}' <http://172.17.0.7:8200/v1/secret/bacon>
root@402825380ff6:~/vault# curl -X GET -H "X-Vault-Token:$VAULT_TOKEN" http://172.17.0.7:8200/v1/secret/bacon | jq .
root@402825380ff6:~/vault# curl -X POST -H "X-Vault-Token:$VAULT_TOKEN" -d '{"tasty":"very", "cooktime":"11"}' http://172.17.0.7:8200/v1/secret/bacon
root@402825380ff6:~/vault# curl -X GET -H "X-Vault-Token:$VAULT_TOKEN" http://172.17.0.7:8200/v1/secret/bacon | jq .
```

[vault]: https://vaultproject.io
[consul]: https://www.consul.io
[285]: https://github.com/habitat-sh/core-plans/pull/285
[286]:https://github.com/habitat-sh/core-plans/pull/286
