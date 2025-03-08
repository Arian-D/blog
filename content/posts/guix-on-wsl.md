+++
title = "Guix System on WSL"
date = 2025-03-07
tags = ["guix", "lisp", "wsl", "windows", "nix"]
draft = false
+++

If you know what both [WSL](https://learn.microsoft.com/en-us/windows/wsl/about) and [Guix System](https://guix.gnu.org/#get-guix-system), your first reaction might
be a bit of a surprise. WSL is Microsoft's Linux subsystem for
Windows, a proprietary operating system that embeds [candy crush ads](https://answers.microsoft.com/en-us/microsoftedge/forum/all/how-can-i-get-rid-of-my-login-screen-idiotically/0e7801ef-1b32-4f07-884e-ff9384c67d0b) in
the start bar that is less stable than its alternatives, and
everything it stands for screams corporate, while GNU Guix System is a
[free](https://www.gnu.org/philosophy/free-sw.en.html) OS trying to be as pure as possible, both in terms of being
immutable and also avoiding non-free software (not just proprietary,
but also non-free source-available software). Guix is the antithesis
of Windows.

While the fans of the two generally might not overlap, I use both, and
I do like a lot of features [Guile](https://www.gnu.org/software/guile/) and Guix provide like the repl
interaction through Emacs (with [Geiser](https://www.gnu.org/software/guile/manual/html_node/Using-Guile-in-Emacs.html)), [Guix shells](https://guix.gnu.org/manual/en/html_node/Invoking-guix-shell.html), [Guix containers](https://guix.gnu.org/manual/en/html_node/Invoking-guix-shell.html#index-container),
[Guix pack](https://guix.gnu.org/manual/en/html_node/Invoking-guix-pack.html) ing docker containers or rpm or deb files, and many more
things. Windows is convenient and I use it for Work. While the bad rep
Windows gets is fully justified, it has improved drastically since
Microsoft's early days (when Linux [was not their favorite](https://www.theregister.com/2001/06/02/ballmer_linux_is_a_cancer/)), and now it
has a [nice terminal](https://github.com/microsoft/terminal), a [decent package manager](https://github.com/microsoft/winget-cli), and [some sort of
tiling](https://www.microsoft.com/en-us/windows/learning-center/organize-screen-with-snap-layouts).


## Guix on WSL {#guix-on-wsl}

If your attention span is as bad as mine and you need a quick TL;DR
for building the image on a machien with guix, here's the command:

```bash
guix system image -t wsl2 -e '(@ (gnu system images wsl2) wsl-os)'
```

Initially, when I wanted Guix on WSL, I thought it would be a huge
pain. I was already using using the [NixOS on WSL](https://github.com/nix-community/NixOS-WSL), and I had ~~read~~ skimmed
through some articles explaining WSL image format, and my guess was
I'd have to do lots of busy work to get a usable tarball[^fn:1]

I looked it up to see if anyone had done this before, and I came
across two articles:

1.  [This](https://wiki.systemcrafters.net/guix/wsl/) System Crafters article
2.  [This](https://gist.github.com/giuliano108/49ec5bd0a9339db98535bc793ceb5ab4) GitHub gist

While the GH gist was a bit older (at least it's calling it Guix
System not GuixSD[^fn:2]), it still seemed like it was too much work.

Out of curiosity, I went ahead and searched in the Guix Info manual in
Emacs (`C-s wsl C-s`) and learned that some fine folks have already done
the work. If you don't have Emacs, you can just search in the [online
manual](https://guix.gnu.org/manual/en/guix.html). In fact, if you do `guix system image --list-image-types`, you'll
see WSL2 as one of the image types.

I did a bit of digging, and found [wsl2.scm](https://git.savannah.gnu.org/cgit/guix.git/commit/gnu/system/images/wsl2.scm?id=7eddfea4a01e95a58a2c18f71bc4a345a1ad378f). Not only does Guix provide
a wsl2 image builder, there's also this sample `wsl-os` system object
that you can use.

Now, the command above should make sense, and you should be able to
build your own image.


## Importing {#importing}

Now, one way to import the image you just built is to somehow send it
to your Windows machine and import it like this in powershell:

```powershell
wsl --import guix $HOME\Guix .\guix-wsl.tar.gz
```

But if you have SSH access, it's much simpler. Remember that Guix
(unlike Nix that leaves residue `./result` symlinks in the working
directory) returns the path of the built file as the output. Using
this simple fact, you can have a one-liner to build, send, and import
the image:

```shell
cat $(guix system image --expression='(@ (gnu system images wsl2) wsl-os)' --image-type=wsl2) | ssh windows-machine wsl --import guix C:\Guix -
```

-   To explain this, the `$(...)` portion builds and outputs the path of
    the built tarball
-   `cat` prints out the content of the tarball to the standard output.
-   `|ssh windows-machine ...` is the fun part where the remote Windows
    machine _reads_ it into powershell's standard input.
-   `wsl --import guix C:\Guix -` imports and creates the WSL distro under
    `C:\Guix`, calls it `guix`, and reads from standard input because of `-`
    (which I discovered thanks to `wsl --help`)


## Pre-built images {#pre-built-images}

I'd highly doubt there would ever be official pre-built images for
public consumption; it just doesn't make sense considering GNU's
goals. Because of that, I set up [this repo](https://github.com/Arian-D/guix-wsl) and GitHub workflow thanks
to a [GitHub action I found for guix](https://github.com/PromyLOPh/guix-install-action). It builds and adds the stock wsl
guix image to the release that you can easily import.

It's very barebones with no instructions, but it's a good starting
point so that folks who don't have a Linux machine or don't want to go
through the trouble of setting up Guix could easily download the
release and use it on Windows.


## Resources for the curious {#resources-for-the-curious}

Microsoft's guide on how to build WSL distros
: <https://learn.microsoft.com/en-us/windows/wsl/build-custom-distro>

Building a WSL tarball from a Docker image
: <https://learn.microsoft.com/en-us/windows/wsl/use-custom-distro>

How NixOS builds its WSL tarball
: <https://github.com/nix-community/NixOS-WSL/tree/main/modules>

Suse's wiki page on WSL
: <https://en.opensuse.org/openSUSE:WSL>

[^fn:1]: WSL images are simply [tarballs](https://en.wikipedia.org/wiki/Tar_(computing)) like docker images, despite
    sometimes having `.wsl` extensions (just like `.ova` files).
[^fn:2]: Guix System [used to be called GuixSD](https://lists.gnu.org/archive/html/help-guix/2021-01/msg00151.html).
