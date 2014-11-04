---
layout: post
title: Migrating emails
summary: ... or how to use your own stuff
categories: imap mail email migration gmail offlineimap linux
tags: imap mail email migration gmail offlineimap linux
---

A few days back I decided to change my mail provider from [Gmail](https://mail.google.com) to a local (German) one. Naturally I wanted to take my old mail with me.

There are a few online migration tools like [ShuttleCloud](http://shuttlecloud.com/solutions/#email-migration) or [Audriga](https://www.migrate-mail.com) but in order to copy the emails from one mailbox to another they all need the necessary passwords. So that's out of the question!

One of the many command line tools to migrate emails is [OfflineImap](http://offlineimap.org). Normally it is used to copy mail from a remote IMAP server into a local mailbox but it can also copy those mails into a remote one.

[OfflineImap](http://offlineimap.org) should be in the package repository of your linux distribution so the installation shouldn't be a problem. Otherwise the [documentation](http://docs.offlineimap.org/en/latest/INSTALL.html) is pretty good. After that you can `cp` an example config file from `/usr/share/offlineimap/offlineimap.conf` to `~/.offlineimaprc` and edit it based on the extensive comments. When you are done just run `offlineimap` and wait ... if you have many mails, wait for a long time.

The config file I used to migrate my mails from Gmail to the new provider is shown below.
There are a few specialities:

*   No passwords: One has to provide them at start.
*   Server fingerprints are stored in the config file: When you run `offlineimap` it will tell you that it doesn't know the server and show you its fingerprint. Verify and add it to the config file.
*   Don't touch Gmail: `readonly=True`
*   Blinkenlights UI: Nice ASCII art

{% highlight ini %}
[general]
metadata = ~/.offlineimap
accounts = main
ui = Blinkenlights

[Account main]
localrepository = <new-provider>
remoterepository = Gmail
status_backend = sqlite

[Repository <new-provider]
type = IMAP
remotehost = <new-provider-host>
remoteport = 993
remoteuser = <new-provider-user>
ssl = yes
cert_fingerprint = c1d95bad26e8473892c4f75aba3f9f226205479b

[Repository Gmail]
type = Gmail
remoteuser = <gmail-user>
readonly = True
cert_fingerprint = 89091347184d41768bfc0da9fad94bfe882dd358
{% endhighlight %}
