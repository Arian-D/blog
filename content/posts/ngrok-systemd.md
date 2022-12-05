+++
title = "Ngrok SystemD Service"
date = 2022-12-04
tags = ["linux", "systemd"]
draft = false
+++

This is going to be a quick post, but it's still something that I
like. I love systemd, and try to (over)use it every time I can.

This is one of those use-cases where I can use [ngrok](https://ngrok.com/) to freely demo my
app without having to set up a cloud instance or open up a port in my
router. The simple usage of running ngrok is fine for simple cases
like hosting a minecraft server for an hour, but if you need to run it
in the background you'd need a service. This is all done without the
use of `sudo` or becoming root; you just need to set up ngrok and have a
machine with systemd.

Over at `.config/systemd/user/ngrok.service` you can make a simple
service

```cfg
[Unit]
Description=Ngrok

[Service]
ExecStart=/tmp/ngrok http --log=stdout 8080

[Install]
WantedBy=multi-user.target
```

Your `/tmp/ngrok` could be (and should be) placed elsewhere for a permanent
location. Then,

```shell
systemctl --user daemon-reload
systemctl --user start ngrok
```

and that's it! You got yourself a <span class="underline">user-level</span> service, i.e. you can do
this on any Linux machine you can ssh to _without_ the need for any sort
of privilege.

You can use this as a backdoor while red-teaming[^fn:1]. You can
use it for a local minecraft server. You can use it for simple
web-apps. Good luck 🙂.

[^fn:1]: This provides some security, since ngrok supports _any_
    tcp-based protocol like ssh, and it provides https out of the box. It
    also is a generic domain, whereas having your static internet facing
    C2 server make a reverse ssh-tunnel would be much more
    revealing. My knowledge of backdoor-ing is limited so I'd point you to
    [this](https://blog.rootshell.be/2020/12/10/sans-isc-python-backdoor-talking-to-a-c2-through-ngrok/).