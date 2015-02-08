# Devuan GNU/Linux

## Simple (package) development kit

### version 0.2

This set of scripts aid package maintainers to import sources from
Debian, verify signatures and stage them to be imported inside
Devuan's git repository.

# Requirements

Tested on Debian and Ubuntu, this SDK requires to have Zsh installed,
plus elinks, gnupg2 and curl.

# Quick start

Using Debian or Ubuntu, install `zsh` `elinks` `gnupg2` `schroot` `debootstrap`

Then clone the SDK repository:

```
git clone https://git.devuan.org/devuan/devuan-sdk.git
```

Then while running into Zsh step inside the sdk, "source" it and
initialize it:

```
cd devuan-sdk
source sdk
init
```

This will take a while to download Debian's keyring and import it,
then clone all Devuan repositories. The `source sdk` is in fact where
one loads the sdk: the running shell *will become the interactive sdk
console* providing command completion and online help. To exit the SDK
one has to close the running shell, or the terminal.

Once `init` is done go into the `stage` directory and you'll see all
the Devuan repositories will be there. One can work normally: commit
and push (you need write permissions) or `format-patch`. Our
continuous integration should be picking up from what is pushed in
master.

Pretty easy no? but this is only the basic usage. SDK has functions to
locally compile the packages into schroot of various architectures and
even serve them locally as a repository over http, to facilitate local
testing.

# Complete instructions

If you are new to Zsh look online for `oh-my-zsh` or `grml` for a
quick and comfortable configuration. Eventually, you may want to help
figuring out why Debian developers think is a good idea to write from
scratch a new shell called Dash.

Nevermind. To feel better at home enter the SDK source directory and
have a look at the `config` file, which mostly works with its
defaults.

As usual to activate the SDK in the current Zsh do `source sdk`.  In
case you get an error, this might be due to your particular setup of
Zsh: then try running the sdk from inside a pristine configuration,
for instance running `zsh --no-rcs`.

## Import a package from Debian

Let's say we want to import the latest version of `nethack-console`
package from Debian 8.0, verify its signature, unpack it and make it
into a git repository.

From inside devuan-sdk/ give the following commands:

```
source sdk
package nethack-console
package-import
```

The SDK output (excluding curl progress bars) will be then:

```
Devuan (*) Importing package: nethack-console
Devuan  .  Source package name: nethack
Devuan  .  Source file: nethack_3.4.3-15.dsc
Devuan  .  Source file: nethack_3.4.3.orig.tar.gz
Devuan  .  Source file: nethack_3.4.3-15.debian.tar.xz
Devuan (*) Package version: 3.4.3
Devuan  .  Package destination: nethack-3.4.3
Devuan  .  Package description: nethack_3.4.3-15.dsc
Devuan  .  Package orig source: nethack_3.4.3.orig.tar.gz
Devuan  .  Package deb  source: nethack_3.4.3-15.debian.tar.xz
```

Then let's verify signatures with

```
package-verify
```

And check the output carefully for the hashes to match:

```
Devuan (*) Importing package: nethack-console
Devuan  .  Compare SHA256 checksums
SHA256 desc checksum: bb39c3d2a9ee2df4a0c8fdde708fbc63740853a7608d2f4c560b488124866fe4 nethack_3.4.3.orig.tar.gz
SHA256 file checksum: bb39c3d2a9ee2df4a0c8fdde708fbc63740853a7608d2f4c560b488124866fe4 nethack_3.4.3.orig.tar.gz
Devuan  .  Compare SHA256 checksums
SHA256 desc checksum: 60af68a29e9c33aafd64d7f72c2cfe21536bb6bbf32c717d03112c24d048e5c0 nethack_3.4.3-15.debian.tar.xz
SHA256 file checksum: 60af68a29e9c33aafd64d7f72c2cfe21536bb6bbf32c717d03112c24d048e5c0 nethack_3.4.3-15.debian.tar.xz
gpg: Signature made Mon 12 May 2014 08:31:47 AM CEST using RSA key ID AA1F32FF
gpg: Good signature from "Vincent Cheng <vcheng@debian.org>"
Primary key fingerprint: D53A 815A 3CB7 659A F882  E395 8EED CC1B AA1F 32FF
```

Here they match and the signature is good: the package is authentic!
From here we can then decide to unpack it and to stage it into our git
repository:

```
package-unpack
```

Will print

```
Devuan (*) Importing package: nethack-console
Devuan  .  Decompressing orig source file
Devuan  .  Decompressing debian source file
```

And then staging it will copy the unpacked sources into the `stage/`
subdir and version them with git, also stamping them with a little
`.devuan` file to remember the package and version they came from in
Debian:

```
package-stage
```

Will output a list of files from the git import, but also SDK
messages:

```
Devuan (*) Importing package: nethack-console
Initialized empty Git repository in /home/jrml/devel/devuan-sdk/stage/nethack-console/.git/
[master (root-commit) 9e7a5b4] Import from Debian jessie/main package nethack-console-3.4.3
 749 files changed, 412550 insertions(+)
[...]
Devuan (*) Stage successfull: stage/nethack-console
Devuan  .  Remote: https://git.devuan.org/packages-base/nethack-console
Devuan (*) Importing package: nethack-console
[master d25e6e3] import stamp version 3.4.3
 1 file changed, 11 insertions(+)
 create mode 100644 .devuan
```

As all goes well you will see that the name and version of the package
is shown on the right prompt, as a reminder of the interactive shell
about which package we are working on.

You are done importing! one last thing: if a new version of the staged
package `nethack-console` would came out in Debian, `package-import`
will realize it. One can then run all the procedure again until
`package-stage` which will then create a git branch with the new
version inside the existing imported repository. This way maintainers
can use git to analyse differences and merge them manually into the
current Devuan package.

## Prepare the build chroots

Once we have staged a package and eventually extirpated any systemd
dependencies, we'll certainly want to test locally its build and
perhaps also install it on a test system to see if all works well. The
SDK build is facilitated by `schroot` (which must be installed) and
operated via the set of `chroot-` commands.

First of all choose the target architecture (we will use `i386`) and
then create the chroot (will ask for the super user password with
sudo):

```
chroot-create i386
```

Then wait since this will take a while the first time. The chroot will
be created using debootstrap in two steps, printing out the set of
packages being installed. Once done, it will stay and you will see
`i386` (or other architectures choosen) on the right prompt. Right now
only `i386` and `amd64` are supported.

Once this process is finished, one can "enter" the chroot and use it
from inside (also install packages and try out things). Just do:

```
chroot-enter devuan-i386
```

Or even from outside the chroot:

```
schroot -c devuan-i386 -u root -d /root -s /bin/zsh
```

Is the same really.  The `i386` is implicit from previous
commands. Chroots are nested operating systems stored as directory
structures into the sdk subdir `chroot`, everything being changed
while inside will be reflected in those directories, making it
possible to `sudo cp` files from the host to chroot.

## Build the package

TODO

# Caveat

This is an early release with limited functionality to facilitate the
import of some packages.

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

