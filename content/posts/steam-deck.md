+++
title = "Steam Deck: The Linux User's Dream"
date = 2022-11-23
tags = ["gaming", "steamdeck", "linux", "arch"]
draft = false
+++

When I received my [Steam Deck](https://www.steamdeck.com/) about -checks Steam account- 3 months
ago, I was already aware of what it was [capable of](https://www.steamdeck.com/en/tech), considering that I
had been browsing [r/SteamDeck](https://www.reddit.com/r/steamdeck) while waiting for it. What I did <span class="underline">not</span>
know was its capabilities as a Linux desktop or even server.

You may ask "what is so special about another Switch clone?" if you
don't know what it is, and if you do you may ask "what is so special
about a handheld pc?"

The truth is, [Valve](https://www.valvesoftware.com/en) got the hardware **and** the [software](https://en.wikipedia.org/wiki/SteamOS) aspects right;
the latter being an [Arch](https://archlinux.org/)-based OS with _full_ root access (I'm talking
`#` shells and being able to `sudo rm -rf /` 😈), which is the main reason I
was able to do so many things with it.


## The hardware {#the-hardware}

If you have been out of the loop, it's a normal Arch Linux machine,
and by normal I mean the cpu is `x86_64` instead of `Aarch64` (ARM). While
this might not lead to a great battery life like the Nintendo
Switch[^fn:1], it does make it possible to run PC games, but
more importantly you can even run things like [Kubernetes](https://www.reddit.com/r/homelab/comments/yg0alv/of_course_the_steam_deck_can_run_kubernetes/), old PC games,
and pretty much most software that was made in the past few decades
for Linux and Windows.


## The environment {#the-environment}

Just like any typical Arch install, SteamOS comes with an OpenSSH
server:

```shell
sudo systemctl statrt sshd
```

That's it! You now have remote access. You can do whatever you would
do with a typical linux laptop. It compiles code faster than my
T580 thinkpad, and it's half the price. 🙂.


## The OS {#the-os}

If you haven't been keeping up with the new wave of GNU/Linux OSs
coming out, you might have missed a new theme:
immutability[^fn:2]. Essentially, the idea is that if you can't change it,
you can't break it. Some recent ones that I have used are:

[Fedora CoreOS](https://getfedora.org/coreos/)
: Immutable server OS

[Fedora Silverblue](https://silverblue.fedoraproject.org/)
: similar to CoreOS, but with a Gnome desktop

[OpenSUSE MicroOS](https://microos.opensuse.org/)
: [SUSE](https://www.suse.com/)'s take on the same idea

The big question you might ask is "if every package is managed _for_ me,
how do I install things, or even modify things?" The answer is simple:
containers. For desktop apps you would use [Flatpak](https://www.flatpak.org/)s, and for services
you can use docker or [podman](https://podman.io/) (rootless and daemonless docker).

You _can_ install stuff like

```shell
sudo pacman -S <some tool>
```

but the problem is that the package would be wiped after a system
update. I will discuss ideas on how to get around this [below](#distrobox).


## Experiments {#experiments}

Now that you have a general idea of what type of system we're dealing
with, let me tell you about the things I've tried on it


### Virtualization {#virtualization}

The answer is a simple yes. This is not an m1 mac attempting to
emulate another architecture; with a simple [QEMU](https://wiki.archlinux.org/title/QEMU) install, I was able
to virtualize Terry Davis' [TempleOS](https://templeos.org/) just for fun, but you can
virtualize Windows, or any other OS if you have the image for it. The
cherry on top is that they can be added as a game to steam like [any
other game](https://steamcommunity.com/sharedfiles/filedetails/?id=156644206).


### Linux Desktop software {#linux-desktop-software}

Again, it's a normal machine. I installed [Doom Emacs](https://github.com/doomemacs/doomemacs), [LibreWolf](https://librewolf.net/), and
Discord with no issues. The only thing to keep in mind is that if you
would like to set keybindings and use it in game-mode, you'd need to
add it as a non-steam game as mentioned [above](#virtualization) or using [BoilR](https://github.com/PhilipK/BoilR).


### Ansible {#ansible}

Just like with _almost_ any other device that comes with SSH and Python,
the steam deck works with ansible. The most useful module is the
[flatpak module](https://docs.ansible.com/ansible/latest/collections/community/general/flatpak_module.html) for installing desktop apps. Check out my [playbooks](https://github.com/Arian-D/steam-deck-setup) to
get an idea on how to declaratively-ish set up your Deck.


### Distrobox {#distrobox}

Coming back to the point from the talk about [the OS](#the-os), you _technically_
cannot install software globally, since the updates wipe your
changes. An interesting way of getting around this is [Distrobox](https://distrobox.privatedns.org/), which
I discovered from the community, and have since been using it on my
other devices as well.

Distrobox lets you run docker images of most Linux distributions, but
it also applies changes that make it look like a real environment,
such as mounting your home directory, getting GUI apps to work, and
etc[^fn:3].


### Remote directory mounting {#remote-directory-mounting}

Emulation on the Deck is great[^fn:4], but newer consoles tend to have games
that go beyond the usual 20-30G. Even on the model that I own with the
512G SSD, it is not enough for storing tens of AAA games[^fn:5]. My desktop
is always running, and it has more than enough storage, but how do we
get the files across if they're stored remotely?[^fn:6]

The answer comes back to my all time favorite protocol: SSH. You can
do more than use a remote shell and transfer files; [SSHFS](https://www.redhat.com/sysadmin/sshfs) lets you
mount directories over the network[^fn:7].

Last piece of the puzzle is to get it to mount whenever there's a
connection. You guess it! SystemD![^fn:8]

SystemD lets you write and manage services as a [normal user](https://wiki.archlinux.org/title/Systemd/User), and it's
great because logging and network dependency can be taken care of,
without any hassle. A simple service that I wrote to mount my ps3
games was this, which I put at `.config/systemd/user/ps3.service`[^fn:9]:

```cfg
[Unit]
Description=Mount remote SSH directories
After=network.target

[Service]
ExecStart=sshfs -f roms-server:roms/ps3 /home/deck/Emulation/roms/ps3
ExecStop=fusermount -u /home/deck/Emulation/roms/ps3
Restart=on-failure

[Install]
WantedBy=network.target
```

Now, enable the service with

```shell
systemctl --user enable ps3
```

And voila! Now all of the remote games become visible when you turn on
the Deck, without taking up any of the local storage.


## Final thoughts {#final-thoughts}

There is so much more to be done like [changing your boot video](https://steamdeckrepo.com/), [set up
3ds with another display](https://www.reddit.com/r/SteamDeck/comments/xiw6hc/dual_screen_camera_monitor_setup_w_minor_update/), or just [use it as a gaming PC](https://www.youtube.com/watch?v=ue0A2Sxr5Fc), but this post
wasn't meant to be a guide; these were simply things I did and learned. For
an extensive guide hit up [Steam Deck Guide](https://github.com/mikeroyal/Steam-Deck-Guide).

[^fn:1]: <https://screenrant.com/steam-deck-battery-life-nintendo-switch/>
[^fn:2]: If you have done any pure [functional programming](https://en.wikipedia.org/wiki/Functional_programming) (like
    [Haskell](https://www.haskell.org/)), this should seem familiar. _If_ this interests you, check out [NixOS](https://nixos.org/).
[^fn:3]: Distrobox only needs docker or podman, and since you cannot
    install either permanently, there is a script on their repo to take
    care of that. Check [this](https://github.com/89luca89/distrobox/blob/main/docs/posts/install_rootless.md) out for more information.
[^fn:4]: You can manually install emulators, use the [RetroDECK](http://retrodeck.net/)
    flatpak, or use [EmuDeck](https://www.emudeck.com/), which is what I did.
[^fn:5]: The reason games -at least in my experience- run so
    smoothly is because of the fact that the games are loaded into memory,
    rather than actively interacting with the storage.
[^fn:6]: Just to clarify, we are not streaming the game, but rather
    the game roms. This would put the load on the Deck's APU, but it also
    runs much better than cloud gaming services, if your internet is slow.
[^fn:7]: This is very similar to [NFS](https://en.wikipedia.org/wiki/Network_File_System), but there is no need to open
    an extra port or install a new program, and obviously, SSH traffic is
    thoroughly encrypted.
[^fn:8]: Another detail I'm omitting is the "What if I'm not
    home?" scenario. I personally have a [WireGuard](https://docs.linuxserver.io/images/docker-wireguard) vpn running with an
    open port on my router, but if you do <span class="underline">not</span> want to open a port, there's
    always Tailscale. I highly suggest [this](https://tailscale.com/blog/steam-deck/) article by Xe, which goes
    significantly more in depth than other sources.
[^fn:9]: `roms-server` in this instance is a remote host that hosts my
    roms which I aliased in `.config/ssh`. You could also put
    `user@yourhost` if you don't want to create a `config` file for SSH.