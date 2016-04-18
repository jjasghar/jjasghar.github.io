---
layout: post
title: "SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B"
date: 2015-07-20 16:18:16 -0500
comments: true
categories: sysadmin chef linux
---

## tl;dr:

You're getting self signed cert errors using Berkshelf or `knife.rb`, add this to your `knife.rb` and run this command:

```ruby
ssl_verify_mode :verify_none
```

```bash
echo '{"ssl":{"verify": false }}' > ~/.berkshelf/config.json
```

## Explanation

If you have a self signed cert on your chef-server, there's a change you've seen this before using berkshelf:

```
E, [2015-07-20T16:15:33.369649 #34774] ERROR -- : Ridley::Errors::ClientError: SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed
E, [2015-07-20T16:15:33.369737 #34774] ERROR -- : /opt/chefdk/embedded/lib/ruby/gems/2.1.0/gems/celluloid-0.16.0/lib/celluloid/responses.rb:29:in `value'
```

This is berkshelf telling you you're not running a signed cert, and it bombs out.

Looking at [this](https://github.com/berkshelf/berkshelf/issues/1266) they give the answer, which is:

```bash
~% chef exec berks upload --no-ssl-verify
```

Now it's possible it's `knife` sending back something like that error. Checkout [Joshua Timberman](https://twitter.com/jtimberman)'s post [here](http://jtimberman.housepub.org/blog/2014/12/11/chef-12-fix-untrusted-self-sign-certs/)
to help out with specific `knife` issues.


Update:

Thanks to [Ryan Cragun](https://twitter.com/ryancragun) my co-worker and general badass, pointing out a way to get around this too.

```bash
~$ echo '{"ssl":{"verify": false }}' > ~/.berkshelf/config.json
```

Now you can drop that `--no-ssl-verify`.

Another Update:

So it seems you might see this with `knife cookbook upload` or any `knife` command for that matter:

```bash
ubuntu@aoeu:~/chef-repo$ knife status
ERROR: SSL Validation failure connecting to host: 172.16.20.62 - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed
ERROR: Could not establish a secure connection to the server.
Use `knife ssl check` to troubleshoot your SSL configuration.
If your Chef Server uses a self-signed certificate, you can use
`knife ssl fetch` to make knife trust the server's certificates.
Original Exception: OpenSSL::SSL::SSLError: SSL_connect r
```

And if you run `knife ssl check` you'd see:

```bash
ubuntu@aoeu:~/chef-repo$ knife ssl check
Connecting to host 172.16.20.62:443
ERROR: The SSL certificate of 172.16.20.62 could not be verified
Certificate issuer data: /C=US/ST=WA/L=Seattle/O=YouCorp/OU=Operations/CN=chefie.novalocal/emailAddress=you@example.com
Configuration Info:
OpenSSL Configuration:
* Version: OpenSSL 1.0.1m 19 Mar 2015
* Certificate file: /opt/chef/embedded/ssl/cert.pem
* Certificate directory: /opt/chef/embedded/ssl/certs
Chef SSL Configuration:
* ssl_ca_path: nil
* ssl_ca_file: nil
* trusted_certs_dir: "/home/ubuntu/chef-repo/.chef/trusted_certs"
TO FIX THIS ERROR:
If the server you are connecting to uses a self-signed certificate, you must
configure chef to trust that server's certificate.

By default, the certificate is stored in the following location on the host
where your chef-server runs:

  /var/opt/opscode/nginx/ca/SERVER_HOSTNAME.crt

Copy that file to your trusted_certs_dir (currently: /home/ubuntu/chef-repo/.chef/trusted_certs)
using SSH/SCP or some other secure method, then re-run this command to confirm
that the server's certificate is now trusted.
```

So you do what the command says:

```bash
ubuntu@aoeu:~/chef-repo$ knife ssl fetch
WARNING: Certificates from 172.16.20.62 will be fetched and placed in your trusted_cert
directory (/home/ubuntu/chef-repo/.chef/trusted_certs).
Knife has no means to verify these are the correct certificates. You should
verify the authenticity of these certificates after downloading.
Adding certificate for chefie.novalocal in /home/ubuntu/chef-repo/.chef/trusted_certs/chefie_novalocal.crt
ubuntu@aoeu:~/chef-repo$ knife status
ERROR: SSL Validation failure connecting to host: 172.16.20.62 - hostname "172.16.20.62" does not match the server certificate
ERROR: Could not establish a secure connection to the server.
Use `knife ssl check` to troubleshoot your SSL configuration.
If your Chef Server uses a self-signed certificate, you can use
`knife ssl fetch` to make knife trust the server's certificates.

Original Exception: OpenSSL::SSL::SSLError: hostname "172.16.20.62" does not match the server certificate
```

That look fimilar, and it feels like you're in a loop....

Turns out you can bypass this check completley by adding this to your `knife.rb`.

```ruby
ssl_verify_mode :verify_none
```

Then you'll be right as rain.
