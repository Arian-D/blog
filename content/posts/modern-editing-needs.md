+++
title = "Modern editing needs"
date = 2025-09-16
tags = ["editors", "emacs", "helix", "opinion", "rant"]
draft = false
+++

Times have changed. The boomer-coded text editor definition of a grid
of bytes is just not enough. Code has structure, tooling, footguns
(especially when it's a dynamically typed language), and many other
attributes that shouldn't be edited by typing `vi code.py` into a
console and expecting things to be done in a timely manner while not
making mistakes. It simply is not possible by a bad developer like me.


## A zoomer perspective of the past {#a-zoomer-perspective-of-the-past}

When I started to _seriously_ code, I was already in college and LSPs
were a thing. While IDEs are things I'm familiar with and use, LSPs
bridge 90% of the gap. From what I've read on the interwebs, the
IDE-only era was grim, and text editors that had genuine advantages
(Emacs for extensibility and Vim for efficient editing) had their own
matrix of extensions that provided a subpar experience. There were
hacks for IDE-like features like [ghcide](https://github.com/haskell/ghcide), but they were never
sufficient, and those hacks were for [corpowagie](https://knowyourmeme.com/memes/wagie)s (who didn't write
Haskell).

Syntax highlighting was difficult. Every editor ha(d|s) their own
custom parsing engine for each language with its custom rules and
custom errors. If you like Emacs, Lisp, parsing, or C-like languages,
I'd very much implore you to read [cc-engine.el](https://cgit.git.savannah.gnu.org/cgit/emacs.git/tree/lisp/progmodes/cc-engine.el). It is a feat of
engineering spanning 4 decades with contributions from people around
the globe. It's close to 16k lines at the time I'm writing this, and
it's actively being worked on. While it's a much more impressive
engine than [Vim's hacky c.vim](https://github.com/vim/vim/blob/master/runtime/syntax/c.vim), it's not bug free.

Tooling was bad. Debugging wasn't easy. It still is not, but at least
there is a protocol for it.


## What exists now. {#what-exists-now-dot}

LSPs
: [The protocol](https://microsoft.github.io/language-server-protocol/) that provides IDE like features like
    autocompletion, documentation lookup, and jumping to
    definition. These things are powerful, and navigating/editing a
    large codebase without them is the equivalent of telling your
    company "I want to waste your money by taking longer to code while
    being in the dark as far as knowing what's wrong with the code." I'm
    very much biased and think they're necessary. They warn you of
    errors and provide non-intrusive completion. If the completion is
    intrusive, or doesn't warn you properly, you're using the wrong LSP.

DAP
: [DAP](https://microsoft.github.io/debug-adapter-protocol/) is another gift from the world's most beloved
    conglomorate. I'll be honest; I don't use it much, but the few times
    I've tried it it has solved my problem(s).

Tree sitter
: [This](https://tree-sitter.github.io/tree-sitter/) was the final nail in the coffin for
    IDEs[^fn:1]. Tree sitter is an incremental parser, and it's designed to
    be fast. Tree sitter gave a lispy like AST definition of non-lisp
    languages to the editor and made parsing a breeze. I've never
    written tree-sitter grammars, but I have messed around with
    text-objects and it feels like lisp, because it is lisp.


## What I need {#what-i-need}

I'm going to be more picky. What _I_ consider bare-minimum does not
reflect anyone else's, but to me, these are the bare minimums in 2025.


### What are the bare minimums? {#what-are-the-bare-minimums}

Tree-sitter-powered syntax highlighting out of the box
: I want
    correct syntax highlighting, not some regex hack from 2 decades
    ago. Emacs _barely_ passes this, but at least major languages have a
    `*-ts-mode`, however this eliminates [VSCode](https://github.com/microsoft/vscode/issues/50140), [Vim](https://github.com/vim/vim/issues/12508), and [Lem](https://github.com/lem-project/lem/issues/757).

LSP support out of the box
: While Neovim has an [lsp client](https://neovim.io/doc/user/lsp.html)
    builtin, I can't randomly open some `.rs` file and expect [rust
    analyzer](https://rust-analyzer.github.io/) to show me docs when I `K`; it requires a little bit of
    configuration. Many languages have official LSPs and for those that
    don't, there are well-recognized ones, and that should be picked if
    it's on the path. This eliminates Neovim, and [Kakoune](https://github.com/kakoune-lsp/kakoune-lsp).

Motion→Action modal editing out of the box
: This eliminates Emacs,
    though it is still my primary editor since I use [Meow](https://github.com/meow-edit/meow). I personally
    do not like Vim's modal editing, so Kakoune, Meow, or Helix are
    my first choices.


### Who lacks the bare minimums? {#who-lacks-the-bare-minimums}

[Helix](https://helix-editor.com/)
: Helix is my favorite text editor. Helix took its
    Motion→Action model [from Kakoune](https://docs.helix-editor.com/usage.html#selection-first-editing), and while Kakoune has a much nicer
    defaults, it lacks many features that I consider to be
    necessary. Helix is simply the fastest, least buggy, and most
    accessible editor I've ever used. DAP, LSP, and TS are all
    implemented for all mainstream and even non-mainstream languages out
    of the box.

[Flow](https://flow-control.dev/)
: This is fast, but not as stable as Helix. It does have Helix
    keybinding emulation. It does have features I need. A +1 goes to
    this editor because it's written is Zig, making it much faster to
    compile, and it's much easier to [cross-compile](https://zig.guide/build-system/cross-compilation/) (I compiled it on
    Linux and `scp` d it to my Windows machine and it just worked).

[Zed](https://zed.dev/)
: While this editor is not geared towards power users,
    auto-installs LSPs like VSCode instead of picking them up from the
    path, and is a pain to install on Windows (I cargo-built it from
    source and it took a long time + 10G of dependency junk), it works
    and satisfies my basic needs. It's not as fast as the previous 2,
    but it does have Helix keybinding emulation to a greater extent than
    Flow.


### A little past the bare minimums {#a-little-past-the-bare-minimums}

There are no perfect editors. The 3 "winner" options all lack
extensibility. Helix is configured with TOML. Flow and Zed are
configured with JSON.

My _ideal_ editor would be one that also has:

Structured editing
: Not just for lisps, but also for general
    purpose languages. What some editors do is use TS text-objects, but
    I wouldn't mind it another way. My current setup is bad; Helix has
    TS text-object [queries](https://github.com/helix-editor/helix/tree/master/runtime/queries), which [evil-textobj-tree-sitter](https://github.com/meain/evil-textobj-tree-sitter/tree/master/queries) uses, which I
    end up using from [meow-tree-sitter](https://github.com/skissue/meow-tree-sitter/tree/main/queries). I'm not a huge fan of
    extra-fancy stuff like [combobulate](https://github.com/mickeynp/combobulate), but a basic syntax-aware "delete
    inside this function" or "highlight around this argument" goes a long way.

Lisp configuration
: This is pretty much self-explanatory; Emacs with
    Elisp, Lem with CL, Helix with Steel, [Neovim with Fennel](https://github.com/Olical/nfnl), and the
    list goes on.


## The future? {#the-future}

The closest option to an "ideal" editor would still be Emacs, **if**
you're willing to install some and do some configuration. Neovim is a
close second, but a lot of the default keys would need to be
modified to mimic Helix. Neovim lacks a real GUI (I can't have mixed
pitch fonts in Neovide), and Emacs is unbearably slow (I'm on an
8c/16t 5.1Ghz Ryzen with 64G of LPDDR5 RAM, with [native comp](https://www.gnu.org/software/emacs/manual/html_node/elisp/Native-Compilation.html) enabled,
and it's still slower than Neovim on a raspberry pi).

What I _hope_ for the future may or may not match the reality, but here
it goes:

-   [The Steel PR](https://github.com/helix-editor/helix/pull/8675) is finally merged into Helix.
-   Having used Guix on a daily basis for a whlie now, I don't expect
    [Guile-Emacs](https://guile-emacs.org/) to have a decent speed, since Guile **3** is slow, Emacs is
    slow, the Emacs elisp interpreter is slow, and the [Guile elisp
    interpreter](https://www.gnu.org/software/guile/manual/html_node/Emacs-Lisp.html) is slow, but hey, the people crazy enough to work on the
    [relaunch](https://emacsconf.org/2024/talks/guile/) might figure something out.
-   Lem starts to support TS out of the box instead of following the
    custom modes like legacy Emacs, and something Meow/[Helix](https://github.com/lem-project/lem/issues/1876)-like makes
    its way to the default build.

If you ask "Why don't you build a tool instead of waiting?"  I will
respond with the fact that I have Emacs. It may be slow, buggy,
single-threaded, resource-intensitve, and old, but it's the only
option that satisfies all my requirements, so we march forward.

[^fn:1]: Not really. Intellij products, Visual Studio, Android Studio, and
    especially the vendor-locked XCode are still out there and thriving,
    but _a lot_ of language-specific IDEs just became unnecessary when tree
    sitter and language server became popular.
