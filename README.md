# Devuan GNU/Linux

## Simple (package) development kit

### version 0.3

This set of scripts aid package maintainers to import sources from
Debian, verify signatures and stage them to be imported inside
Devuan's git repository. 

BEWARE this is still in development and does not addresses strictly
security issues nor wrong usage. USE AT YOUR OWN RISK and in any case
DON'T USE ON YOUR PERSONAL MACHINE.
If you try this fast and loose use a disposable system ;^)

# Requirements

This SDK is designed to be used interactively from a terminal as well
from shell scripts.

Using Debian or Ubuntu, install `sudo` `zsh` `gnupg2` `schroot`
`debootstrap` `dpkg-dev` `curl` `rsync`.

The last one it may be called `dpkg` or `dpkg-devtools` on other
systems like Arch and Parabola.

The Devuan SDK is a fresh take to old tasks :^) acting as a sort of
interactive shell extension. All the instructions below should be
followed while already running in ZSh. A clear advantage is having tab
completion on commands, when running it interactively.

Sudo will be used to elevate the sdk user to superuser privileges only
when needed. In some cases one needs to make sure the following
commands are autorized in your `/etc/sudoers` file. For instance
assuming your username is `luther` then it should have:

```
Cmnd_Alias  DEVUAN = /usr/sbin/debootstrap, /usr/bin/rsync, /usr/bin/test, /usr/bin/curl
luther  ALL= NOPASSWD: DEVUAN
```

# Quick start

Then clone the SDK repository:

```
git clone https://git.devuan.org/devuan/devuan-sdk.git
```

Then run ZSh. In case you have conflicting extensions on your zsh
configuration, it may be needed to run from a vanilla one, using:

```
sudo zsh --no-rcs
```

then step inside the sdk, "source" it and initialize it:

```
cd devuan-sdk
source sdk
init
```

Once `init` is done go into the `stage` directory and you'll see all
the Devuan repositories will be there.

To proceed further we need to build our software packages and toast
them into an installer iso. First of all, choose the architecture for
which we are building. At the moment one can choose between `amd64`
and `i386`, we will choose the latter:

```
arch i386
chroot-create
```

Beware, this will take long: will run debootstrap, download Debian's
netinst iso and customize it as needed.

Consider a single sdk can create more than one chroot for multiple
architectures and they keep existing between builds. Switching is done
via `arch`.

Now let's imagine we need to import a new package from Debian,
something that has not yet being staged in Devuan. Let's pick
`hasciicam`:


```
package hasciicam
version latest
import
verify
stage
```

Then the hasciicam sourcecode will be in stage/ and checked in git. If
the package `hasciicam` is already staged in Devuan, then an error
would be given while importing the same version, but different
versions would be checked in as branches.

Ok now if we have a package correctly staged then its name and version
are shown at right of prompt, as well the architecture we are
targeting the build. Then launch the build with:

```
build
```

If the sourcecode and the package are good, it will end up with
success and the packages will be found in the `builds/` subdirectory
inside the sdk.

Now to insert the hasciicam package inside our new installer iso lets
first download the current installer in Debian (quick way):

```
iso-import
```

Then add the hasciicam package to it:

```
iso-add-package hasciicam
```

And at last launch the chain of commands leading to the final iso

```
auto-iso
```

The results will be in the sdk subdir `iso/`.

Pretty easy no? This is the basic usage. SDK has also functions to
locally compile the packages into schroot of various architectures and
even serve the results locally as an apt repository over http, to
facilitate local testing.

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
init
```

The `source sdk` is in fact where one loads the sdk:
the running shell *will become the interactive sdk console* providing
command completion and online help. To exit the SDK one has to close
the running shell, or the terminal.

The `init` command needs to be executed only on the first run: it will
take a while to clone all Devuan repositories and download Debian's
keyring.

Once done, import the nethack-console package:

```
package nethack-console
package-import
```

The SDK output (excluding curl progress bars) will be then:

```
Devuan (*) Importing package: nethack
Devuan  .  Downloading sources from: http://ftp.debian.org/debian//pool/main/n/nethack
Devuan (*) Source version:  3.4.3-15
Devuan  .  Source destination: nethack-3.4.3
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
Initialized empty Git repository in /home/jrml/devel/devuan-sdk/stage/nethack-console/.git/
[master (root-commit) 9e7a5b4] Import from Debian jessie/main package nethack-console-3.4.3
 749 files changed, 412550 insertions(+)
[...]
Devuan (*) Stage successfull: stage/nethack-console
Devuan  .  Remote: https://git.devuan.org/packages-base/nethack-console
Devuan (*) Importing package: nethack-console
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

All `package-` steps can be listed by interactive completion, just
type `package-[tab]` for a reminder. They are also aliased to shorter
commands: simply omitt the `package-` prefix.

## Prepare to build in chroots

Once we have staged a package and eventually extirpated any systemd
dependencies, we'll certainly want to test locally its build and
perhaps also install it on a test system to see if all works well. The
SDK build is facilitated by `schroot` (which must be installed) and
operated via the set of `chroot-` commands.

First of all choose the target architecture (we will use `i386`) and
then create the chroot (will ask for the super user password with
sudo):

```
arch i386
chroot-create
```

Then wait since this will take a while the first time. The chroot will
be created using debootstrap in two steps, printing out the set of
packages being installed. Once done, it will stay and you will see
`i386` (or other architectures choosen) on the right prompt. Right now
only `i386` and `amd64` are supported.

Once this process is finished, one can "enter" the chroot and use it
from inside (also install packages and try out things). Just do:

```
chroot-enter
```

Chroots are nested operating systems stored as directory structures
into the sdk subdir `chroot`, everything being changed while inside
will be reflected in those directories, making it possible to `sudo
cp` files from the host to chroot.

## Build the package

Just type:

```
auto-build
```

If all goes well, results will be in the SDK subdir `builds/`.

To sign the package use:

```
build-sign
```

The `auto-build` command is really a wrapper among various steps, to
list them one can use again `build-[tab]`. All steps can be launched
standalone:

```
build-deps
build-clean
build-sync
build-pack-orig
build-pack-debian
build-start
build-finish
```

# Toast the iso

The installer ISO can be toasted into a ready to burn image with a
simple `auto-iso` command, here the breakdown of its various steps:

```
iso-import
iso-replace-packages
iso-prepare
iso-make
```

We just use to dowload Debian's netinst and change its contents for
now, but things may change rapidly in some future.

# Caveat

This is an early release with limited functionality to facilitate the
import and maintainance of some packages that are core to Devuan.

Some things may change in the future.

To support the development you are welcome to open issues on problems
and bugs you encounter, open merge requests of patches or simply
getting involved in other tasks evident on https://git.devuan.org

# Acknowledgments

The Devuan SDK was conceived during a period of residency at the
Schumacher college in Dartington UK, greatly inspired by the laborious
and mindful atmosphere of its wonderful premises.

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

