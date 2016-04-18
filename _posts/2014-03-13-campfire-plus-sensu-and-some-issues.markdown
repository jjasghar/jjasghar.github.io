---
layout: post
title: "Campfire + Sensu and some issues"
date: 2014-03-13 14:45:02 -0500
comments: true
categories: sysadmin campfire sensu
---

I've got sensu enabled now, finally taken nagios out to pasture and shot it. You may now clap.

Thanks! Ok, anyway, when you get your new toy you want to start leveraging some
of the cool things it can do.  My company uses campfire as our main daily communication,
and luckily there's a [campfire handler](https://github.com/sensu-plugins/sensu-plugins-campfire)
for sensu! Naturally we should leverage it right? Well, it seems with 0.12 sensu,
it's not exactly a cup o' tea.

Go ahead and add you campfire handler and you'll notice quickly:

```ruby
/opt/sensu/embedded/lib/ruby/2.0.0/net/http.rb:917:in `connect': SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (Faraday::Error::ConnectionFailed)
        from /opt/sensu/embedded/lib/ruby/2.0.0/net/http.rb:917:in `block in connect'
        from /opt/sensu/embedded/lib/ruby/2.0.0/timeout.rb:51:in `timeout'
        from /opt/sensu/embedded/lib/ruby/2.0.0/net/http.rb:917:in `connect'
        from /opt/sensu/embedded/lib/ruby/2.0.0/net/http.rb:861:in `do_start'
        from /opt/sensu/embedded/lib/ruby/2.0.0/net/http.rb:850:in `start'
        from /opt/sensu/embedded/lib/ruby/2.0.0/net/http.rb:1366:in `request'
        from /opt/sensu/embedded/lib/ruby/2.0.0/net/http.rb:1125:in `get'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday-0.8.7/lib/faraday/adapter/net_http.rb:73:in `perform_request'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday-0.8.7/lib/faraday/adapter/net_http.rb:38:in `call'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday-0.8.7/lib/faraday/response.rb:8:in `call'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday-0.8.7/lib/faraday/response.rb:8:in `call'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday_middleware-0.9.0/lib/faraday_middleware/response_middleware.rb:30:in `call'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday-0.8.7/lib/faraday/response.rb:8:in `call'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday_middleware-0.9.0/lib/faraday_middleware/request/encode_json.rb:23:in `call'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday-0.8.7/lib/faraday/connection.rb:247:in `run_request'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/faraday-0.8.7/lib/faraday/connection.rb:100:in `get'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/tinder-1.9.1/lib/tinder/connection.rb:76:in `get'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/tinder-1.9.1/lib/tinder/campfire.rb:34:in `rooms'
        from /opt/sensu/embedded/lib/ruby/gems/2.0.0/gems/tinder-1.9.1/lib/tinder/campfire.rb:48:in `find_room_by_name'
        from campfire_test.rb:7:in `<main>'
```

Note: I'd like to thank [Bethany Erskine](https://github.com/skymob) for all the advice and the [gist](https://gist.github.com/skymob/6161155) I just stole to explain this.

Ok, honestly it'll be locked up in the `sensu-client.log` but if you break up the json it'll look like the above. Anyway....

So as you can see it fails with the `SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (Faraday::Error::ConnectionFailed)` trying to verify the certs,
I spent some time trying to updated the pkg version of the ssl certs, reading more than I ever wanted to know about SSL, finally I pinged Bethany asked her after finding that gist.

She responded back with something that just seemed so obious, if you look at the [handler](https://github.com/sensu/sensu-community-plugins/blob/master/handlers/notification/campfire.rb#L30) it defaults to true.

Change that to it to something like:
```ruby
Tinder::Campfire.new('mydomain', :ssl_verify => false, :token => '7505e9c7ed5c30a77dTHIS_IS_FAKE93029a494eb7c3d20')
```

And...all the sudden you've got messages in your campfire channel.

Yes, yes, you should verify ssl, but honestly, for what you get out of sensu for this, it's gravey for now. Plus I think after I ping [Sean Porter](https://github.com/portertech/) about this issue in a passive/aggressive manner
he'll probably get it fixed. Love ya Sean :)
