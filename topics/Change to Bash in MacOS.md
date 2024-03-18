Change to Bash in MacOS
===

Mar 18, 2024

_P.S. Its so dumb how there is so many Mac colleagues asking me (a primarily Windows user), how to fix their Mac problems._
_And similarly, why their terminal looks so scarce compared to mine because they can't be bothered to learn how to rice ZSH._
_Like my fellow sibling in Christ, you are using a Mac!_

I prefer to use Bash. Period.

Having the same terminal emulator across Windows, MacOS, and Linux really helps to stifle any weird
surprises and behavior, and allows me to quickly work without remembering what different keyboard
shortcuts there are for other terminal emulators.

### Install Bash

```console
brew install bash
```

Obviously you need to have [Homebrew](https://brew.sh/) setup, and almost 90% of
any developer setup requires Homebrew at any point. Hence...

This should get you Bash version 5 (or above), which is the latest one. MacOS is
not upgrading the built in Bash version for god knows why.

This will install the _neue_ Bash in `/opt/homebrew/bin/bash`.

### Add the new Bash as an available shell option

```sh
# edit the shell options
sudo vim /etc/shells

# then put this line below at the top
# /opt/homebrew/bin/bash

# save and close the file

# now change the current user's shell to bash
chsh -s /opt/homebrew/bin/bash
```

Close _all_ terminals, and open a new one, to double check if Bash is in effect
by `echo $SHELL`.

### Funky shell environment?

This was not a thing for me with my old Intel Macbook, but after this far in, Bash is
still not fully working in the M2 Macbook.

It would appear that we need to run `brew shellenv` [for Bash to work properly](https://stackoverflow.com/a/65505326).

```sh
# i added these into my global .bash_profile
if [[ -f /opt/homebrew/bin/brew ]]; then
    eval $(/opt/homebrew/bin/brew shellenv)
fi
```

Bash should work now.

### How did you install Bash in MacOS before?

Seems pretty much the same, except:

- I have a symlink from `/usr/local/bin/bash` to `/usr/local/Cellar/bash/5.2.21/bin/bash`.
  I cannot recall if this was Homebrew's actions or mine.
- Because of the symlink, my `/etc/shells` only point to `/usr/local/bin/bash`.
- Maybe the `/opt/homebrew` path is Homebrew's way of handling the new ARM chips?

---

#### References

- https://brew.sh/
- https://stackoverflow.com/a/77052639
- https://stackoverflow.com/a/65505326
