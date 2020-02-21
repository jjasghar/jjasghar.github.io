---
layout: post
title: "Setting up Jenkins on CentOS 8"
date: 2020-02-21 14:43:15
categories: centos jenkins sysadmin
---


Here are my notes on how I got Jenkins working on CentOS 8. This is mainly so I can go to
my blog instead of [here](https://linuxize.com/post/how-to-install-jenkins-on-centos-8/).

## Vanilla Jenkins

First thing first, we need some `jdk`. Time to install it.

```bash
sudo dnf install -y java-1.8.0-openjdk-devel wget epel-release
```

Next, we need to install the `jenkins` repo and get `jenkins` installed.

```bash
sudo wget -O /etc/yum.repos.d/jenkins.repo http://pkg.jenkins-ci.org/redhat-stable/jenkins.repo
sudo rpm --import https://jenkins-ci.org/redhat/jenkins-ci.org.key
sudo yum install jenkins
```

After installation, you need to start the service, and make sure your `firewalld` is properly
configured:

```bash
sudo systemctl start jenkins
sudo systemctl enable jenkins
sudo systemctl status jenkins
sudo firewall-cmd --permanent --zone=public --add-port=8080/tcp
sudo firewall-cmd --reload
```

Awesome, now we go to `http://<ip>:8080` and put in the inital `AdminPassword`.

```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

Wonderful, now lets get `nginx` infront of it so we can connect to a webserver instead of the
`java` application directly.

## nginx forwarding for Jenkins

If you google for `nginx jenkins` you come directly to the [Jenkins Wiki](https://wiki.jenkins.io/display/JENKINS/Jenkins+behind+an+NGinX+reverse+proxy). It's not wrong, it's just more informatin then we need
right now.

Lets first install `ngnix`.

```bash
sudo dnf install -y nginx
```

Next, start the service and get the `firewalld` in a good happy place:

```bash
sudo systemctl enable nginx
sudo systemctl start nginx
sudo firewall-cmd --permanent --zone=public --add-service=http
sudo firewall-cmd --permanent --zone=public --add-service=https
sudo firewall-cmd --reload
```

You should be able to hit `http://<ip>` and see the nginx website.

Next, I disabled SELinux. I converted it from enforcing to disabled and reboot my machine.

Now, let’s configure the meat of this blog post, lets get some self signed certs going:

```bash
sudo mkdir /etc/ssl/private
sudo chmod 700 /etc/ssl/private
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/nginx-selfsigned.key -out /etc/ssl/certs/nginx-selfsigned.crt # answer the prompts here :)
sudo openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```

Now you need to wire the nginx and jenkins together. If you are just going to do `https` the following
file that you can put in `/etc/nginx/conf.d/jenkins.conf`.

**NOTE:** You'll need to change `.domain.tld` to whatever _your_ domain is.

```
upstream jenkins {
  server 127.0.0.1:8080 fail_timeout=0;
}

server {
  listen 80;
  server_name jenkins.domain.tld;
  return 301 https://$host$request_uri;
}

server {
  listen 443 ssl;
  server_name jenkins.domain.tld;

  ssl_certificate_key /etc/ssl/private/nginx-selfsigned.key;
  ssl_certificate /etc/ssl/certs/nginx-selfsigned.crt;

  location / {
    proxy_set_header        Host $host:$server_port;
    proxy_set_header        X-Real-IP $remote_addr;
    proxy_set_header        X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header        X-Forwarded-Proto $scheme;
    proxy_redirect http:// https://;
    proxy_pass              http://jenkins;
    # Required for new HTTP-based CLI
    proxy_http_version 1.1;
    proxy_request_buffering off;
    proxy_buffering off; # Required for HTTP-based CLI to work over SSL
    # workaround for https://issues.jenkins-ci.org/browse/JENKINS-45651
    add_header 'X-SSH-Endpoint' 'jenkins.domain.tld:50022' always;
  }
}
```


After adding this to the file, restart `nginx` and you should find yourself
hosting your jenkins instance at `https://jenkins.domain.tld`.

```bash
sudo systemctl restart nginx
```
