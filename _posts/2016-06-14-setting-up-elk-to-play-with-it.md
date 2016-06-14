---
layout: post
title: "Setting up ELK to play with it"
date: 2016-06-14 14:22:53
categories:
---

I was at an Austin DevOps meetup last night and there were some great
presentations on the ELK stack. I've played with it, but never fully taken the
jump. This is my how to on how to get it working quickly and injecting
twitter streams in it so you can start playing with [kibana][kibana].

Because all cool kid do this now, install docker:

```bash
$ curl -fsSL https://get.docker.com/ | sh
```

You could use another installation method, but this way is good for a throwaway
instance; or hell a not-so-throwaway instance. Next install docker compose from
github.

```bash
$ curl -L https://github.com/docker/compose/releases/download/1.7.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
```

I found a docker composed elk stack that spun up three containers exactly how I
wanted with little interaction, so now clone down the repo.

```bash
$ git clone https://github.com/deviantony/docker-elk
```

After you get everything down run docker-compose up to make sure everything is working as expected. You should
be able to go to the machine with http://HOSTNAME:5601 and see the kibana dashboard.

```bash
$ cd docker-elk
$ docker ps
$ docker-compose up -d
```

Bring everything back down now so you can continue configuring it.

```bash
$ docker-compose down
```

Pull down the twitter scrapper:

```bash
cd ..
mkdir  twitter_elk_example
cd twitter_elk_example
wget https://raw.githubusercontent.com/elastic/examples/master/ELK_twitter/twitter_logstash.conf
wget https://raw.githubusercontent.com/elastic/examples/master/ELK_twitter/twitter_template.json
wget https://raw.githubusercontent.com/elastic/examples/master/ELK_twitter/twitter_kibana.json
```

Edit the `twitter_logstash.conf` with your keys from https://apps.twitter.com/app/new.
You'll need to setup an `Access Token`.

After that create a directory in the `docker-elk/logstash` called: `template`,
copy the `twitter_template.json` in there. Then copy the `twitter_logstash.conf`
in the `logstash/config` directory. You'll need to edit it with the following:

```ini
input {
  twitter {
    consumer_key       => "<ADD YOUR SECRET HERE>"
    consumer_secret    => "<ADD YOUR SECRET HERE>"
    oauth_token        => "<ADD YOUR SECRET HERE>"
    oauth_token_secret => "<ADD YOUR SECRET HERE>"
    keywords           => [ "words", "you", "interested", "scraping"]
    full_tweet         => true
  }
}

filter { }

output {
  stdout {
    codec => dots
  }
  elasticsearch {
      hosts => "elasticsearch"
      index         => "twitter-%{+YYYY.MM.dd}"
      document_type => "tweets"
      template_name => "twitter_elk_example"
      template_overwrite => true
      template => "/etc/logstash/template/twitter_template.json"

  }
}
```

I should tell you here, that the `keywords` won't work with hashtags. It seems
that we connect to the "streaming" api, and at least with the account I set up
you can only get words not hashtags. I'm investigating this farther.

After this, run `docker compose up` again then:

```bash
$ curl http://HOSTNAME:9200/_cat/indices?v
health status index               pri rep docs.count docs.deleted store.size pri.store.size
yellow open   .kibana               1   1          8            0     65.8kb         65.8kb
yellow open   twitter-2016.06.14    5   1       2171            0     18.3mb         18.3mb
```

And see something like the above. Congrats you're getting data into ES!

Next you'll need to configure kibana  the indexes to search properly, so go to
"Settings" and put in the search pattern as `twitter-*` this should start grabbing
all of your indexes.

Have fun!

[kibana]: https://www.elastic.co/products/kibana
