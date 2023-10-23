---
layout: post
title: "Shared vCard for free-ish"
date: 2023-10-23 08:25:15
categories: devrel
---

I was at a conference and ran out of business cards. For a Developer Advocate
like me in certain countries this is a big problem. I normally had only a
handful of minutes to impress and convince a developer that I know what
I’m talking about and am a worthy person to follow up with in the future.

I noticed in this country people do the NFC tap or QR code exchange of
contacts when business cards weren’t available. It got me curious on
what options were out there, and I discovered I could easily build
a free-ish version quite easily.

I did mine with an iPhone, but assuming Android can export the `vcf`
file this should work great.

First off, you need to find you contact in the Contacts App. You should
find a “Share Contact” button at the bottom, select the specific contact
info you want to share (you can even do Note too!), and “Save it files.”
This will put it in you iCloud file share, and move over to your laptop.

Grab the file from your iCloud share (`your name.vcf`) and copy it to
some place on your laptop. I suggest changing the file name to something
without a `" "` in it, like `jj_asghar.vcf`.  Now you need a place to put
this file "permanently" online. I have a domain attached to a GitHub
repository <https://jjasghar.me> that I host a simple HTML "short cut"
website, which was a logical place to post this `.vcf` file.

After getting the file added to my repository I now have a "permanent"
link to <https://jjasghar.me/jj_vcard.vcf>.

And now all I had to do was create a QRcode to it:

```bash
qrencode https://jjasghar.me/jj_vcard.vcf -o ~/Downloads/jjasghar_vcard_github.png
```

And if I wanted a "tap" share contact, I put that same link into an NFC tag or programmable
card and :boom: done! (this is where the "ish" comes into play you need to purchase
these)

FYI: People think you need to program the vCard into the QRcode or NFC, but it’s not,
it’s actually a web link to a hosted place. Leveraging something like GitHub for this
now you don’t need to pay the monthly or subscription cost for the exact same sharing tech.
