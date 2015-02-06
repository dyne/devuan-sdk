# Devuan GNU/Linux

## Simple (package) development kit

### version 0.1

This set of scripts aid package maintainers to import sources from
Debian, verify signatures and stage them to be imported inside
Devuan's git repository.

# Requirements

Tested on Debian and Ubuntu, this SDK requires to have Zsh installed,
plus elinks, gnupg2 and curl.

# Usage

This SDK initializes the currently running shell with a set of tools
to manipulate packages. It works as a shell extension.

If you are new to Zsh look online for `oh-my-zsh` or `grml` for a
quick and comfortable configuration. Eventually, you may want to help
figuring out why Debian developers think is a good idea to write from
scratch a new shell called Dash.

Well, nevermind. Unpack and enter the SDK directory. Have a look at
the `config` file, which mostly works with its defaults.

To import the source packages we already have in devuan, make sure you
have a gitlab account and then do:

```
source sdk init
```

This will update the list of Devuan packages on the stage, cloning
those that are missing. It will also download and import the debian
keyring to verify the imported packages, which may require some time
the first time: please keep patient and make a tea.

Once done, think of a name of a Debian package to work on, for
instance `desktop-base`, then from inside devuan-sdk/ do:

```
source sdk desktop-base
```

If all goes well you will see that the name of the package is shown on
the right prompt with a `-?` following it, because we don't know yet
its latest version.

Start the download of the package with:

```
import
```

Curl will show its progress and the .dsc and .tar files will be found
in the `sources/` subdirectory relative to the SDK.

Once initialized the keyring, cryptographic verification of the
currently selected package is simply done with:

```
verify
```

And analyzing the output of the command, which will compute SHA256
sums and verify the .dsc signature.

To unpack all the sources, automatically overlaying the
.debian. separate source if present, do:

```
unpack
```

At last, to stage the source and import it as first git commit:

```
stage
```

Every command calls all the preceeding steps and fails on error. Hence
calling `stage` directly as first command will call in cascade of all
other steps: `get-sources` -> `verify` -> `unpack` -> `stage`.

# Caveat

This is an early release with limited functionality to facilitate the
import of some packages.

More functionalities will be added soon, to stage the packages for
Devuan and upload them, eventually to handle ChangeLogs and to compare
version between Debian and Devuan.

This SDK is designed to be used interactively from a terminal as well
from shell scripts.

# License

Devuan SDK is Copyright (C) 2015 by the Dyne.org Foundation

Devuan SDK is designed, written and maintained by Denis Roio <jaromil@dyne.org>
 
This source code is free software; you can redistribute it and/or
modify it under the terms of the GNU Public License as published by
the Free Software Foundation; either version 3 of the License, or (at
your option) any later version.

This source code is distributed in the hope that it will be useful,
but WITHOUT ANY WARRANTY; without even the implied warranty of
MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  Please refer to
the GNU Public License for more details.

You should have received a copy of the GNU Public License along with
this source code; if not, write to: Free Software Foundation, Inc.,
675 Mass Ave, Cambridge, MA 02139, USA.

