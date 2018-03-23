+++
title = "My Elvish Configuration With Commentary"
author = ["Zamboni Diego"]
date = 2017-11-16T20:21:00+01:00
tags = ["config", "howto", "literateprogramming", "elvish"]
draft = false
creator = "Emacs 25.3.2 (Org mode 9.1.8 + ox-hugo)"
toc = true
featured_image = "/images/elvish-logo.svg"
summary = "In this blog post I will walk you through my current Elvish configuration file, with running commentary about the different sections."
+++

Last update: **March 23, 2018**

In this blog post I will walk you through my current [Elvish](http://elvish.io) configuration file, with running commentary about the different sections.

This is also my first blog post written using [org-mode](http://orgmode.org/), which I have started using for writing and documenting my code, using [literate programming](http://www.howardism.org/Technical/Emacs/literate-programming-tutorial.html). The content below is included unmodified from my [rc.org file](https://github.com/zzamboni/dot-elvish/blob/master/rc.org) (as of the date shown above), from which the [rc.elv](https://github.com/zzamboni/dot-elvish/blob/master/rc.elv) file is directly generated.

Without further ado...


## Paths {#paths}

First we set up the executable paths. We set the `GOPATH` environment variable while we are at it, since we need to use it as part of the path.

```elvish
E:GOPATH = ~/Personal/devel/go/
paths = [
  ~/bin
  $E:GOPATH/bin
  ~/Dropbox/Personal/devel/hammerspoon/spoon/bin
  ~/.gem/ruby/2.4.0/bin
  /opt/X11/bin
  /Library/TeX/texbin
  /usr/local/bin
  /usr/local/sbin
  /usr/sbin
  /sbin
  /usr/bin
  /bin
]
```


## Package installation {#package-installation}

The bundled [epm](https://elvish.io/ref/epm.html) module allows us to install and manage Elvish packages.

```elvish
use epm
```

For now I use these packages:

-   [github.com/zzamboni/elvish-modules](https://github.com/zzamboni/elvish-modules) contains all my modules except completions and themes. Maybe these should be separated eventually, but for now this works fine.
-   [github.com/zzamboni/elvish-themes](https://github.com/zzamboni/elvish-themes) contains my prompt themes (only [chain](https://github.com/zzamboni/elvish-themes/blob/master/chain.org) for now).
-   [github.com/zzamboni/elvish-completions](https://github.com/zzamboni/elvish-completions) contains my completer definitions.
-   [github.com/xiaq/edit.elv](https://github.com/xiaq/edit.elv), which includes the `smart-matcher` module used below.
-   [github.com/muesli/elvish-libs](https://github.com/muesli/elvish-libs) for the git utilities module.

```elvish
epm:install &silent-if-installed=$true   \
  github.com/zzamboni/elvish-modules     \
  github.com/zzamboni/elvish-completions \
  github.com/zzamboni/elvish-themes      \
  github.com/xiaq/edit.elv               \
  github.com/muesli/elvish-libs
```

The modules within each package get loaded individually below.


## Base modules {#base-modules}

Load the bundled [re](https://elvish.io/ref/re.html) module to have access to regular expression functions.

```elvish
use re
```

The bundled [readline-binding](https://elvish.io/ref/bundled.html) module associates some Emacs-like keybindings for manipulation of the command line.

```elvish
use readline-binding
```

I add a couple of keybindings which are missing from the default `readline-binding` module:

-   `Alt-backspace` to delete word

    ```elvish
    edit:insert:binding[Alt-Backspace] = $edit:kill-small-word-left~
    ```

-   `Alt-d` to delete the word under the cursor

    ```elvish
    edit:insert:binding[Alt-d] = { edit:move-dot-right-word; edit:kill-word-left }
    ```


## Git utilities {#git-utilities}

I use muesli's git utilities module.

```elvish
use github.com/muesli/elvish-libs/git
```


## Aliases {#aliases}

Elvish does not have built-in alias functionality, but this is implemented easily using the [alias](https://github.com/zzamboni/modules.elv/blob/master/alias.org) module, which stores the alias definitions as functions under [~/.elvish/aliases/](https://github.com/zzamboni/dot-elvish/tree/master/aliases) and loads them automatically.

```elvish
use github.com/zzamboni/elvish-modules/alias
```

For reference, I define here a few of my commonly-used aliases:

```elvish
alias:new dfc e:dfc -W -l -p -/dev/disk1s4,devfs
alias:new ls e:ls --color=auto
alias:new more less
alias:new v vagrant
```


## Nix package manager {#nix-package-manager}

I use the [Nix](https://nixos.org/nix/) package manager on macOS, the [nix](https://github.com/zzamboni/modules.elv/blob/master/nix.org) module sets up the necessary environment variables.

```elvish
use github.com/zzamboni/elvish-modules/nix
nix:multi-user-setup
```


## Completions {#completions}

From the  [elvish-completions](https://github.com/zzamboni/elvish-completions) package:

For `git`:

```elvish
use github.com/zzamboni/elvish-completions:git
```

For `vcsh` (uses the git completer):

```elvish
use github.com/zzamboni/elvish-completions:vcsh
```

For `cd`:

```elvish
use github.com/zzamboni/elvish-completions:cd
```


## Prompt theme {#prompt-theme}

I use the [chain](https://github.com/zzamboni/theme.elv/blob/master/chain.org) prompt theme, ported from the fish theme at <https://github.com/oh-my-fish/theme-chain>.

```elvish
use github.com/zzamboni/elvish-themes/chain
chain:bold-prompt = $true
```

Elvish has a [comprehensive mechanism](https://elvish.io/ref/edit.html#prompts) for displaying prompts with useful information while avoiding getting blocked by prompt functions which take too long to finish. For the most part the defaults work well. One change I like to make is to change the [stale prompt transformer](https://elvish.io/ref/edit.html#stale-prompt) function to make the prompt dim when stale:

```elvish
edit:prompt-stale-transform = { each [x]{ edit:styled $x[text] "gray" } }
```

Another possibility is to make the prompt stay the same when stale - useful to avoid distractions (disabled for now):

```elvish
edit:prompt-stale-transform = $all~
```

I also like the continuous update of the prompt as I type (by default it only updates on Enter and on `$pwd` changes, but I like also git status changes to be updated automatically), so I increase its eagerness.

```elvish
edit:-prompt-eagerness = 10
```


## Automatic proxy settings {#automatic-proxy-settings}

When I am in the office, I need to use a proxy to access the Internet. For macOS applications, the proxy is set automatically using a company-provided PAC file. For the environment variables `http_proxy` and `https_proxy`, commonly used by command-line programs, the [proxy](https://github.com/zzamboni/modules.elv/blob/master/proxy.org) module allows me to define a test which determines when the proxy should be used, so that the change is done automatically.

First, we load the module and set the proxy host.

```elvish
use github.com/zzamboni/elvish-modules/proxy
proxy:host = "http://proxy.corproot.net:8079"
```

Next, we set the test function to enable proxy auto-setting. In my case, the `/etc/resolv.conf` file contains the `corproot.net` domain (set through DHCP) when I'm in the corporate network, so I can check for that.

```elvish
proxy:test = {
  and ?(test -f /etc/resolv.conf) \
  ?(egrep -q '^(search|domain).*corproot.net' /etc/resolv.conf)
}
```


## Long-running-command notifications {#long-running-command-notifications}

The [long-running-notifications](https://github.com/zzamboni/modules.elv/blob/master/long-running-notifications.org) module allows for producing a notification when a command takes longer than a certain time to finish (by default the period is 10 seconds). The module automatically detects when [terminal-notifier](https://github.com/julienXX/terminal-notifier) is available on macOS and uses it to produce Mac-style notifications, otherwise it prints a notification on the terminal.

```elvish
use github.com/zzamboni/elvish-modules/long-running-notifications
```


## Directory and command navigation and history {#directory-and-command-navigation-and-history}

Elvish comes with built-in location and command history modes. I use the `narrow` module, which allow for more customization, including pre- and after- hooks for each of the modes. I use the default binding for history mode (<kbd>C-r</kbd>), but change the location mode binding to <kbd>Alt-l</kbd> to avoid conflicting with the binding of <kbd>C-l</kbd> to "clear screen" in `readline-binding`. I disable the lastcmd binding because it is taken over by the `bang-bang` module below.

```elvish
use narrow
narrow:bind-trigger-keys &location=Alt-l &lastcmd=""
```

I have decades of muscle memory using <kbd>!!</kbd> and <kbd>!$</kbd> to insert the last command and its last argument, respectively. The [bang-bang](https://github.com/zzamboni/elvish-modules/blob/master/bang-bang.org) module allows me to keep using them.

```elvish
use github.com/zzamboni/elvish-modules/bang-bang
```

The [dir](https://github.com/zzamboni/modules.elv/blob/master/dir.org) module implements a directory history and some related functions. I alias the `cd` command to `dir:cd` so that any directory changes are kept in the history. I also alias `cdb` to `dir:cdb` function, which allows changing to the base directory of the argument.

```elvish
use github.com/zzamboni/elvish-modules/dir
alias:new cd "use github.com/zzamboni/elvish-modules/dir; dir:cd"
alias:new cdb "use github.com/zzamboni/elvish-modules/dir; dir:cdb"
```

`dir` also implements a narrow-based directory history chooser, which I bind to <kbd>Alt-i</kbd> (I have found I don't use this as much as I thought I would - the built-in location mode works nicely).

```elvish
edit:insert:binding[Alt-i] = $dir:history-chooser~
```

I bind `Alt-b/f` to `dir:left-word-or-prev-dir` and `dir:right-word-or-next-dir` respectively, which "do the right thing" depending on the current content of the command prompt: if it's empty, they move back/forward in the directory history, otherwise they move through the words of the current command. In my Terminal.app setup, `Alt-left/right` also produce `Alt-b/f`, so these bindings work for those keys as well.

```elvish
edit:insert:binding[Alt-b] = $dir:left-word-or-prev-dir~
edit:insert:binding[Alt-f] = $dir:right-word-or-next-dir~
```


## Dynamic terminal title {#dynamic-terminal-title}

The [terminal-title](https://github.com/zzamboni/elvish-modules/blob/master/terminal-title.org) module handles setting the terminal title dynamically according to the current directory or the current command being executed.

```elvish
use github.com/zzamboni/elvish-modules/terminal-title
```


## Loading private settings {#loading-private-settings}

The `private` module sets up some private settings such as authentication tokens. This is not on github :) The `$private-loaded` variable gets set to `$ok` if the module was loaded correctly.

```elvish
private-loaded = ?(use private)
```


## O'Reilly Atlas {#o-reilly-atlas}

I sometimes use the [O'Reilly Atlas](https://atlas.oreilly.com/) publishing platform. The [atlas](https://github.com/zzamboni/modules.elv/blob/master/atlas.org) module contains some useful functions for triggering and accessing document builds.

```elvish
use github.com/zzamboni/elvish-modules/atlas
```


## Smart matching for completion {#smart-matching-for-completion}

The [smart-matcher](https://github.com/xiaq/edit.elv/blob/master/smart-matcher.elv) module tries prefix match, smart-case prefix match, substring match, smart-case substring match, subsequence match and smart-case subsequence match automatically.

```elvish
use github.com/xiaq/edit.elv/smart-matcher
edit:completion:matcher[''] = $smart-matcher:match~
```

Other possible values for `edit:completion:matcher` are `[p]{ edit:match-prefix &smart-case $p }` for smart-case completion (if your pattern is entirely lower case it ignores case, otherwise it's case sensitive).  `&smart-case` can be replaced with `&ignore-case` to make it always case-insensitive.


## Environment variables {#environment-variables}

Default options to `less`.

```elvish
E:LESS = "-i -R"
```

Use vim as the editor from the command line (although I am an [Emacs](https://github.com/zzamboni/dot-emacs/blob/master/init.org) fan, I still sometimes use vim for quick editing).

```elvish
E:EDITOR = "vim"
```

Locale setting.

```elvish
E:LC_ALL = "en_US.UTF-8"
```


## Utility functions {#utility-functions}

The [util](https://github.com/zzamboni/elvish-modules/blob/master/util.org) module includes various utility functions.

```elvish
use github.com/zzamboni/elvish-modules/util
```


## Exporting aliases {#exporting-aliases}

We populate `$-exports-` with the alias definitions so that they become available in the interactive namespace.

```elvish
-exports- = (alias:export)
```
