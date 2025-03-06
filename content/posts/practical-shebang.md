+++
title = "Practical shebangs"
date = 2025-03-03
tags = ["linux", "unix", "posix", "shell", "env"]
draft = false
+++

Usually, one of the first things a Linux/Unix user learns is to create
a shell script. The process generally goes like this:

1.  Write a `script.sh` file.
2.  Fill it with
    ```bash
    #!/bin/bash
    echo "Hello world"
    ```
3.  Make it executable with `chmod +x script.sh`
4.  And finally, run it with `./script.sh`

Nice and simple, and most \*nix users are familiar with this. For an
average user, the journey ends right then and there, but if you're
interested, you realize that there's always ways to use and abuse the
tooling.

In this post, I want to show some common (and uncommon) things I do
with shebangs. This might not be new or interesting to the sysadmin
wizards with decades of experience, but for a dumb zoomer, such as
yours truly, this is all fascinating and useful.


## Shebangs aren't just for shell scripts {#shebangs-aren-t-just-for-shell-scripts}

You probably have already seen python scripts with `#!/usr/bin/python`
at the top. As you can see, _any_ program could be passed at the top;
all that happens is that the path of the file is passed as the first
argument of the shebang.

You could even have a text file that's printed whenever you execute it

```nil
#!/bin/cat
Hello world!
```

or have a file destroy itself when you execute it

```nil
#!/bin/rm
This file will self-destruct when you execute it.
```

The pattern should be clear: the interpreter/executor goes in the
shebang, and the input goes in lines after. This is a powerful idea.


## The limit {#the-limit}

Well, there is a limit. It's the number of arguments. You can do
things like `#!/usr/bin/ls -l`, but not something like `#!/usr/bin/ls -l
-a`. Why? Because the two flags are clumped together as `"-l -a"`.
Solution? `env`.

~~Victims~~ users of NixOS or Guix System can probably see where this is
going. On those immutable systems, your `/bin` generally only contains
`sh`, and `/usr/bin/` only contains `env`. Because of that, you'll see a lot
of scripts that do things like `#!/usr/bin/env bash`, instead of
hard-referencing the executables.

What we'd want to focus on is the power of `env`, mainly the `-S`[^fn:1] flag
which allows us to include multiple arguments. Another limitation is
in-lining the file name, instead of placing it at the end of the
line. `sh`'s `"$0"` lets us do that. For instance, you can do

```nil
#!/usr/bin/env -S sh -c 'scp "$0" server:/tmp'

This file will be uploaded to the server's /tmp directory upon execution.
```


## Some examples {#some-examples}

Now that the gibberish (hopefully) makes more sense, let's look at
what we can do with it.

**Dockerfiles**
: I have a lot of `something.Dockerfile` files that are
    on my machine. Most of the time, I just wanna build and run them
    with one command. To accomplish that, I add this line to all of my Dockerfiles
    ```nil
    #!/usr/bin/env -S sh -c 'docker build -t $(basename "$0" .Dockerfile) -f "$0" . && docker run --rm -it $(basename "$0" .Dockerfile) bash'
    ```
    This assumes your file's extension is `.Dockerfile` and you want to
    launch bash when you enter the container, but remember that you can
    always change it based on your usage.

**VPN**
: I have a wireguard server at home, and whenever I needed to
    use my configuration I'd have to do `wg setconf ...`. Then, I realized
    I can sneak in sudo in the shebang. Now, the first line of my
    wireguard config reads `#!/usr/bin/env -S sudo wg setconf wg0` and I
    just do `./wg0.conf` whenever I need to connect. I don't use OpenVPN,
    but you can probably do something similar.

**Nix and Guix shells**
: I can show examples of these, but instead
    I'll refer you to the [Nix wiki](https://nixos.wiki/wiki/Nix-shell_shebang) and the [Guix manual](https://guix.gnu.org/manual/devel/en/html_node/Invoking-guix-shell.html#index-shebang_002c-for-guix-shell) for more
    comprehensive examples and description.

**uv**
: You can use uv in a similar fashion as Nix, and pass
    dependencies in a multi-line shebang. Check out this [post](https://treyhunner.com/2024/12/lazy-self-installing-python-scripts-with-uv/) for more
    info.

**Kubernetes**
: `#!/usr/bin/env kubectl apply -f`

**Ansible**
: `#!/usr/bin/env ansible-playbook`.

**Emacs Org-mode**
: Watch [this](https://emacsconf.org/2021/talks/exec/).

[^fn:1]: Unfortunately, the `-S` flag is not part of the [POSIX standard](https://pubs.opengroup.org/onlinepubs/9799919799/utilities/env.html), but
    you can still use this on Linux and FreeBSD.
