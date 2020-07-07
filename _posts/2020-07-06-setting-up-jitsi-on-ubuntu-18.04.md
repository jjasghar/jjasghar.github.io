---
layout: post
title: "Setting up jitsi on Ubuntu 18.04"
date: 2020-07-06 16:13:58
categories: linux sysadmin
---


Here are my notes on how I got [jitsi][jitsi] working on Ubuntu 18.04. I've
it to work "back in the day" but I really wanted my tutorial re-written up.


## Pre-reqs

I spun up a VM with 32 Gigs of RAM and 8 vCPUs, and pointed the extrenal IP
to `chat.asgharlabs.io`.
On the machine I ran:
```bash
apt update
apt upgrade
apt install apt-transport-https
apt-add-repository universe
apt update
```

## Install jitsi meet

First thing I did was set the hostname with `hostnamectl`:
```bash
hostnamectl set-hostname chat.asgharlabs.io
```
Next I added the jitsi package repo:
```bash
curl https://download.jitsi.org/jitsi-key.gpg.key | sudo sh -c 'gpg --dearmor > /usr/share/keyrings/jitsi-keyring.gpg'
echo 'deb [signed-by=/usr/share/keyrings/jitsi-keyring.gpg] https://download.jitsi.org stable/' | sudo tee /etc/apt/sources.list.d/jitsi-stable.list > /dev/null
apt update
```
Then I configured the firewall:
```bash
ufw allow 80/tcp
ufw allow 443/tcp
ufw allow 4443/tcp
ufw allow 10000/udp
ufw allow 22/tcp
ufw enable
```
Then a sanity check:
```bash
ufw status verbose
```
Now install the debian package:
```bash
apt install jitsi-meet
```
**Note**: You want to select `Self-Signed` initially so you can do Let's Encrypt _later_.
If you have security rules on your cloud, you'll need to open up thees ports too:
```bash
22, 80, 443, 4443, 10000
```
Next attempt to run the cert creation script:
```bash
/usr/share/jitsi-meet/scripts/install-letsencrypt-cert.sh
```
If it errors you may have to install this package then run again:
```bash
apt install libc6
```
Now you should be able to go to your domain: <https://chat.asgharlabs.io> and see
a working jitsi link.

## Plugins

### Etherpad

Install [etherpad][etherpad] with the following:
```bash
curl -sL https://deb.nodesource.com/setup_10.x | sudo -E bash -
apt install -y nodejs
git clone --branch master https://github.com/ether/etherpad-lite.git
```
Next open up the port for etherpad to listen on:
```bash
ufw allow 9001/tcp
```
Be sure to add it to your cloud security group too.
Next open up the `/etc/nginx/sites-available/<hostname>.conf` file and add this under
`xmpp websockets` stanza.
```
   # Etherpad-lite
    location ^~ /etherpad/ {
        proxy_pass http://localhost:9001/;
        proxy_set_header X-Forwarded-For $remote_addr;
        proxy_buffering off;
        proxy_set_header       Host $host;
    }
```
Next open up `/etc/jitsi/meet/<hostname>-config.js` and add the following after:
```bash
    // List of undocumented settings used in jitsi-meet
      etherpad_base: 'https://<hostname>/etherpad/p/',
```
Now to test it out, you need to run the following. (only have `--root` if you are running
as root:
```
cd etherpad-lite && bin/run.sh --root
```

You should now have the "Open Shared Document" option on the meeting!

### Streaming to YouTube (I still haven't gotten this section to work)

#### jibri setup

The the streaming software is called: [jibri][jibri].

> Jibri provides services for recording or streaming a Jitsi Meet conference.

> It works by launching a Chrome instance rendered in a virtual framebuffer and capturing and encoding the output with ffmpeg. It is intended to be run on a separate machine (or a VM), with no other applications using the display or audio devices. Only one recording at a time is supported on a single jibri.

Being it assumes you have a _seperate_ machine for jibri, i spun up another machine
without an external IP to run this software. jibri was built with 16.04, so I spun one
up.

First commands I ran:
```bash
apt update
apt upgrade
apt install linux-image-extra-virtual
```

After this I did some ALSA and Loopback Device settings:
```bash
echo "snd-aloop" >> /etc/modules
reboot
modprobe snd-aloop
lsmod | grep snd_aloop # as a sanity check
```

Setting up `ffmepg` with X11 capture support
```bash
apt install ffmpeg
```

Installing Google Chrome and ChromeDriver
```bash
curl -sS -o - https://dl-ssl.google.com/linux/linux_signing_key.pub | apt-key add
echo "deb [arch=amd64] http://dl.google.com/linux/chrome/deb/ stable main" > /etc/apt/sources.list.d/google-chrome.list
apt-get -y update
apt-get -y install google-chrome-stable
```

Add chrome managed policies file and set `CommandLineFlagSecurityWarningsEnabled` to false. It will hide warnings in Chrome. You can set it like so:
```bash
mkdir -p /etc/opt/chrome/policies/managed
echo '{ "CommandLineFlagSecurityWarningsEnabled": false }' >>/etc/opt/chrome/policies/managed/managed_policies.json
```

Chromedriver is also required and can be installed like so:
```bash
CHROME_DRIVER_VERSION=`curl -sS chromedriver.storage.googleapis.com/LATEST_RELEASE`
wget -N http://chromedriver.storage.googleapis.com/$CHROME_DRIVER_VERSION/chromedriver_linux64.zip -P ~/
apt install unzip
unzip ~/chromedriver_linux64.zip -d ~/
rm ~/chromedriver_linux64.zip
sudo mv -f ~/chromedriver /usr/local/bin/chromedriver
sudo chown root:root /usr/local/bin/chromedriver
sudo chmod 0755 /usr/local/bin/chromedriver
```

And finally the misc tools and dependancies for `jibri`
```bash
apt-get install default-jre-headless ffmpeg curl alsa-utils icewm xdotool xserver-xorg-input-void xserver-xorg-video-dummy
apt autoremove
```

Now to install the `jibri`:
```bash
wget -qO - https://download.jitsi.org/jitsi-key.gpg.key | sudo apt-key add -
sh -c "echo 'deb https://download.jitsi.org stable/' > /etc/apt/sources.list.d/jitsi-stable.list"
apt-get update
apt-get install jibri
usermod -aG adm,audio,video,plugdev jibri # sanity check
```

Edit the config file:
```bash
vi /etc/jitsi/jibri/config.json
```
**Note**: on the first pass through/read I didn't change anything.

#### Prosody (on the machine that is _not_ the recorder)

Add your recorder machine to `/etc/hosts`:
```
10.240.128.6    recorder.asgharlabs.io
10.240.128.5    auth.asgharlabs.io
10.240.128.5    chat.asgharlabs.io
```

Edit the `prosody.cfg.lua` one is a component, and another virtual host:
```lua
-- internal muc component, meant to enable pools of jibri and jigasi clients
Component "auth.asgharlabs.io" "muc"
    modules_enabled = {
      "ping";
    }
    storage = "internal"
    muc_room_cache_size = 1000
```
and
```lua
VirtualHost "recorder.asgharlabs.io"
  modules_enabled = {
    "ping";
  }
  authentication = "internal_plain"
```
Next create the two accounts jibri will use:
```bash
prosodyctl register jibri auth.asgharlabs.io jibriauthpass
prosodyctl register recorder recorder.asgharlabs.io jibrirecorderpass
```

#### Jicofo setup

Edit the `/etc/jitsi/jicofo/sip-communicator.properties`:
```
org.jitsi.jicofo.BRIDGE_MUC=JvbBrewery@auth.asgharlabs.io
org.jitsi.jicofo.jibri.BREWERY=JibriBrewery@auth.asgharlabs.io
org.jitsi.jicofo.jibri.PENDING_TIMEOUT=90
```

#### Jitsi Meet

Edit the `/etc/jitsi/meet/chat.asgharlabs.io-config.js`:
```javascript
fileRecordingsEnabled: true, // If you want to enable file recording
liveStreamingEnabled: true, // If you want to enable live streaming
hiddenDomain: 'recorder.asgharlabs.io',
```


#### YouTube setup

On the [youtube][youtube] side, go to the creator studio then the "Go Live"
button on the right hand side. Create a new stream, and change it from
`Public` to Private for testing. Name the stream too. :)

Grab the "Stream Key", and add it in to the meeting. Assuming you're the mod on the
meeting.

Create a room on jitsi and try to stream!



[jitsi]: https://jitsi.github.io/handbook/
[jibri]: https://github.com/jitsi/jibri
[etherpad]: https://github.com/ether/etherpad-lite
[youtube]: https://www.youtube.com/dashboard
