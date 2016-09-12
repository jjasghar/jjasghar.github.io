---
layout: post
title: "ERROR: Excon::Errors::CertificateError: SSL_connect and knife"
date: 2016-04-16 19:49:10 -0500
comments: true
categories: chef linux
---

I was playing around with [knife-rackspace][knife-rackspace] and when I
attempted to run a simple `knife rackspace server list` I got this error:

{% highlight shell %}
19:44:56 ~ > knife rackspace server list
ERROR: Excon::Errors::CertificateError: SSL_connect returned=1 errno=0
state=SSLv3 read server certificate B: certificate verify failed
(OpenSSL::SSL::SSLError)Unable to verify certificate. This may be an issue
with the remote host or with Excon.Excon has certificates bundled, but these
can be customized.`Excon.defaults[:ssl_ca_path] = path_to_certs`,
`ENV['SSL_CERT_DIR'] = path_to_certs`, `Excon.defaults[:ssl_ca_file] =
path_to_file`, `ENV['SSL_CERT_FILE'] = path_to_file`,
`Excon.defaults[:ssl_verify_callback] = callback` (see OpenSSL::SSL::
SSLContext#verify_callback), or `Excon.defaults[:ssl_verify_peer] = false`
(less secure).
{% endhighlight %}

Needless to say this was frustrating. I spent a while attempting to research
why this was happening, and eventually I figured out how to at least do a work
around. I do need to disclaim this, this is just a workaround, this isn't a fix.

I read the error, and noticed: `Excon.defaults[:ssl_verify_peer] = false`. I tried
every why possible to put this on the command line, but eventually I put it in my
`knife.rb`. As I said this isn't a _fix_ per se, but a work around so [Excon][excon]
stops complaining about the SSL certificate.


[knife-rackspace]: https://github.com/chef/knife-rackspace
[excon]: https://github.com/excon/excon
