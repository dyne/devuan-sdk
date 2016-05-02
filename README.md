# Devuan GNU/Linux

## Simple (package) development kit

### version 0.6

#### DISCLAIMER

This SDK version is not up to date with the Continuous Integration
workflow Devuan has in place since a few months now. Updates to this
SDK will follow, meanwhile be aware that this code will not produce
iso images valid for release, since these are done via CI now.

### Introduction

This set of scripts aid package maintainers to import sources from
Debian, verify signatures and stage them to be imported inside
Devuan's git repository.

The Devuan SDK is a fresh take to old tasks :^) acting as a sort of
interactive shell extension. All the instructions below should be
followed while already running in ZSh. A clear advantage is having tab
completion on commands, when running it interactively.

BEWARE this is still in development and does not addresses strictly
security issues nor wrong usage. USE AT YOUR OWN RISK and in any case
DON'T USE ON YOUR PERSONAL MACHINE.
If you try this fast and loose use a disposable system ;^)

# Requirements

This SDK is designed to be used interactively from a terminal as well
from shell scripts.

Using Debian or Ubuntu, install the following packages:

```
gnupg2 schroot debootstrap debhelper makedev curl rsync dpkg-dev \
gcc-arm-none-eabi parted kpartx qemu-user-static pinthread sudo
```

Please note that:
 - `dpkg-dev` may be called `dpkg` or `dpkg-devtools` on other systems like Arch and Parabola.
 - `pinthread` is Devuan software and may not exist in other distros
 - `sudo` is used to elevate the sdk user to superuser privileges and should be configured accordingly

# Quick start

This is a quick guide to build a Devuan ISO from it sources using our
SDK, it will require approximatel 4GB space and an Internet
connection. This guide is not meant for people willing to use Devuan,
but for package maintainers and Devuan developers.

First clone the SDK repository:

```
git clone https://git.devuan.org/devuan/devuan-sdk.git
```

Then run ZSh. In case you have conflicting extensions on your zsh
configuration, it may be needed to run from a vanilla one, using:

```
zsh --no-rcs
```

then step inside the sdk, "source" it:

```
cd devuan-sdk

source sdk
```

then initialise it (needs to be done only the first time)

```
sdk-init
```

from inside the sdk environment is possible to tab-complete available public commands using `sdk-[tab]`.

## Chroot build

To proceed further we need to build our software packages and toast
them into an installer iso. First of all, choose the architecture for
which we are building. One can choose between `amd64` or `i386` or
`armhf` or other architectures.

```
sdk-chroot-arch i386

sdk-chroot-build
```

Beware, this will take long.

Consider a single SDK can create more than one chroot for multiple
architectures and they keep existing between builds.
Switching is done via `sdk-chroot-arch`.

## ISO build

Now the next step: toast the iso with all the built packages inside.
First choose what kind of seed configuration is wanted, at the time of
writing the choices are `netinst` or `xfce` which are both installers
and not live CDs.

```
seed xfce
```

Then toast the iso automatically using:

```
auto-iso
```

Which will download a Debian iso as template and change it to become a
systemd-free Devuan.

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

Then as usual from inside devuan-sdk/ give the following commands:

```
source sdk
```

The running shell *will become the interactive sdk console* providing
command completion and online help. To exit the SDK one has to close
the running shell, or the terminal.  So you are inside the SDK
environment. We recommend using shells that are initialized this way
to be used exclusively for the Devuan SDK purposes: after `source sdk`
they are not anymore normal shells. To go back to normal just `exit`
the terminal or open a new one besides, as the SDK is only active in
the shell where it is launched.

If this is the first SDK run, or if one needs to update the staged
sources downloading new commits from our server, then run the `init`
command.


## Import a package from Debian


Now let's imagine we need to import a new package from Debian,
something that has not yet being staged in Devuan. Let's pick
`hasciicam` and give the following commands to download it, verify its
signature, unpack it and make it into a git repository.


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


As all goes well you will see that the name and version of the
`hasciicam` package (or any other you have choosen) is shown on the
right prompt, as a reminder of the interactive shell about which
package we are working on.

You are done importing! one last thing: if a new version of the staged
package `hasciicam` would came out in Debian, the `import` will
realize it. One can then run all the procedure again until
the `stage` command which will then create a git branch with the new
version inside the existing imported repository. This way maintainers
can use git to analyse differences and merge them manually into the
current Devuan package.

All the commands used here can be prefixed with `package-` and listed
by interactive completion: just type `package-[tab]` for a
reminder of the commands available.

## Prepare to build in chroots

Once we have staged a package and eventually extirpated any systemd
dependencies, we'll certainly want to test locally its build and
perhaps also install it on a test system to see if all works well. The
SDK build is facilitated by `schroot` (which must be installed) and
operated via the set of `sdk-chroot-` commands.

First of all choose the target architecture (we will use `i386`) and
then create the chroot (will ask for the super user password with
sudo):

```
sdk-chroot-arch armhf
sdk-chroot-build
```

Then wait since this will take a while the first time. The chroot will
be created using debootstrap in two steps, printing out the set of
packages being installed. Once done, it will stay and you will see
`i386` (or other architectures choosen) on the right prompt. Right now
only `i386` and `amd64` are supported.

Once this process is finished, one can "enter" the chroot and use it
from inside (also install packages and try out things). Just do:

```
sdk-chroot-do
```

Chroots are nested operating systems stored as directory structures
into the sdk subdir `chroot`, everything being changed while inside
will be reflected in those directories, making it possible to `sudo
cp` files from the host to chroot.

## Build the package

Just type:

```
build
```

If all goes well, results will be in the SDK subdir `builds/`.

The `build` command is really a wrapper among various steps, to
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

If the build fails in the middle, one can comfortably modify the
staged source, including the `debian/*` files, and then retry from
where it failed using:

```
build-retry
```

To build all your source packages:

```
build all
```

Also this command will take long and will put your computer to work.
If the sourcecode and the package are good, it will end up with
success and the packages will be found in the `builds/` subdirectory
inside the sdk.

# Toast the iso

The installer ISO can be toasted into a ready to burn image with a
simple `auto-iso` command (as shown in the quick start guide
above). Here the breakdown of various steps performed by `auto-iso`:

```
seed xfce
iso-import
iso-replace-packages
iso-seed
iso-index
iso-make
```

We select the xfce iso format, dowload the Debian image for it,
replace its contents with our packages in builds/, make sure loginkit
is present, add our preseed and toast it. The resulting iso will be
found in the sdk subdirectory `iso/`.


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
