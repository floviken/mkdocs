# Portage features
## Portage features

Portage has several additional features that make the Gentoo experience even better. Many of these features rely on certain software tools that improve performance, reliability, security, ...

To enable or disable certain Portage features, edit /etc/portage/make.conf and update or set the *FEATURES* variable which contains the various feature keywords, separated by white space. In several cases it will also be necessary to install the additional tool on which the feature relies.

Not all features that Portage supports are listed here. For a full overview, please consult the make.conf man page:

```sh
user $ man make.conf
```

To find out what *FEATURES* are set by default, run **emerge --info** and search for the *FEATURES* variable or *grep* it out:

```sh
user $ emerge --info | grep ^FEATURES=
```
## Distributed compiling
### Using distcc

**distcc** is a program to distribute compilations across several, not necessarily identical, machines on a network. The distcc client sends all necessary information to the available distcc servers (running distccd) so they can compile pieces of source code for the client. The net result is a faster compilation time.

More information about distcc (and how to have it work with Gentoo) can be found in the [Distcc](https://wiki.gentoo.org/wiki/Distcc) article.

### Installing distcc
Distcc ships with a graphical monitor to monitor tasks that the computer is sending away for compilation. This tool is automatically installed if `USE="gtk"` is set.

```sh
root # emerge --ask sys-devel/distcc
```

### Activating Portage distcc support

Add distcc to the *FEATURES* variable inside /etc/portage/make.conf. Next, edit the *MAKEOPTS* variable and increase the number of parallel build jobs that the system allows. A known guideline is to fill in `-jN` where `N` is the number of CPUs that run distccd (including the current host) plus one, but that is just a guideline.

Now run **distcc-config** and enter the list of available distcc servers. For a simple example assume that the available DistCC servers are 192.168.1.102 (the current host), 192.168.1.103 and 192.168.1.104 (two "remote" hosts):

```sh
root # distcc-config --set-hosts "192.168.1.102 192.168.1.103 192.168.1.104"
```
Don't forget to run the distccd daemon as well:

```sh
root # rc-update add distccd default
root # /etc/init.d/distccd start
```

## Caching compilation objects

### About ccache

**ccache** is a fast compiler cache. Whenever an application is compiled, it will cache intermediate results so that, whenever the same program and version is recompiled, the compilation time is greatly reduced. The first time ccache is run, it will be much slower than a normal compilation. Subsequent recompiles however should be faster. ccache is only helpful if the same application version will be recompiled many times; thus it is mostly only useful for software developers.

For more information about ccache, please visit its [homepage](https://ccache.dev/).

!!! Warning
```sh
ccache is known to cause numerous compilation failures. Sometimes ccache will retain stale code objects or corrupted files, which can lead to packages that cannot be emerged. If this happens (errors like "File not recognized: File truncated" come up in build logs), try recompiling the application with ccache disabled (FEATURES="-ccache" in /etc/portage/make.conf or one-shot from the commandline with the following) before reporting a bug:
```
```sh
root # FEATURES="-ccache" emerge --oneshot <category/package>
```
### Installing ccache

To install ccache run the following command:

```sh
root # emerge --ask dev-util/ccache
```

### Activating Portage ccache support

Open /etc/portage/make.conf and add `ccache` to any values defined in the *FEATURES* variable. If *FEATURES* does not exist, then create it. Next, add a new variable called *CCACHE_SIZE* and set it to `2G`:

```sh title="FILE /etc/portage/make.confEnabling Portage ccache support"
FEATURES="ccache"
CCACHE_SIZE="2G"
```

To check if ccache functions, ask ccache to provide its statistics. Because Portage uses a different ccache home directory, it is necessary to temporarily set the *CCACHE_DIR* variable:

```sh
root # CCACHE_DIR="/var/tmp/ccache" ccache -s
```

The /var/tmp/ccache/ location is Portage' default ccache home directory; it can be changed by setting the *CCACHE_DIR* variable in /etc/portage/make.conf.

When running **ccache** standalone, it would use the default location of ${HOME}/.ccache/, which is why the *CCACHE_DIR* variable needs to be set when asking for the (Portage) ccache statistics.

### Using ccache outside Portage

To use ccache for non-Portage compilations, add /usr/lib/ccache/bin/ to the beginning of the *PATH* variable (before /usr/bin). This can be accomplished by editing ~/.bash_profile in the user's home directory. Using ~/.bash_profile is one way to define *PATH* variables.

``` sh title="FILE ~/.bash_profileSetting the ccache location before any other PATH"
PATH="/usr/lib/ccache/bin:${PATH}"
```

## Binary package support

### Creating prebuilt packages

Portage supports the installation of prebuilt packages. Even though Gentoo does not provide prebuilt packages by itself Portage can be made fully aware of prebuilt packages.

To create a prebuilt package use the **quickpkg** command if the package is already installed on the system, or emerge with the `--buildpkg` or `--buildpkgonly` options.

To have Portage create prebuilt packages of every single package that gets installed, add buildpkg to the *FEATURES* variable.

More extended support for creating prebuilt package sets can be obtained with catalyst. For more information on catalyst please read the [Catalyst FAQ](https://wiki.gentoo.org/wiki/Project:Catalyst/FAQ).

### Installing prebuilt packages

Although Gentoo doesn't provide one, it is possible to create a central repository where prebuilt packages are stored. In order to use this repository, it is necessary to make Portage aware of it by having the *PORTAGE_BINHOST* variable point to it. For instance, if the prebuilt packages are on ftp://buildhost/gentoo:

``` sh title="FILE /etc/portage/make.conf Add PORTAGE_BINHOST location"
PORTAGE_BINHOST="ftp://buildhost/gentoo"
```

To install a prebuilt package, add the `--getbinpkg` option to the emerge command alongside of the `--usepkg` option. The former tells emerge to download the prebuilt package from the previously defined server while the latter asks emerge to try to install the prebuilt package first before fetching the sources and compiling it.

For instance, to install gnumeric with prebuilt packages:

`root # emerge --usepkg --getbinpkg gnumeric`

More information about emerge's prebuilt package options can be found in the emerge man page:

`user $ man emerge`


### Distributing prebuilt packages to others

If prebuilt packages are to be distributed to others, then make sure that this is permitted. Check the distribution terms of the upstream package for this. For example, for a package released under the GNU GPL, sources must be made available along with the binaries.

Ebuilds may define a `bindist` restriction in their *RESTRICT* variable if built binaries are not distributable. Sometimes this restriction is conditional on one or more USE flags.

By default, Portage will not mask any packages because of restrictions. This can be changed globally by setting the *ACCEPT_RESTRICT* variable in /etc/portage/make.conf. For example, to mask packages that have a `bindist` restriction, add the following line to make.conf:

``` sh title="FILE /etc/portage/make.confOnly accept binary distributable packages"
ACCEPT_RESTRICT="* -bindist"
```

It is also possible to override the *ACCEPT_RESTRICT* variable by passing the `--accept-restrict` option to the **emerge** command. For example, `--accept-restrict=-bindist` will temporarily mask packages with a `bindist` restriction.

Also consider setting the *ACCEPT_LICENSE* variable when distributing packages. See the Licenses section for this.

!!! Important

It is entirely the responsibility of each user to comply with packages' license terms and with laws of each user's country. The metadata variables defined by ebuilds (*RESTRICT* or *LICENSE*) can provide guidance when distribution of binaries is not permitted, however output from Portage or questions answered by the Gentoo developers are not legal statements and should not be relied upon as such. Be cautious to abide by the law of your physical location.

## Fetching files

### Userfetch

Portage is normally run as the root user. Setting `FEATURES="userfetch"` will allow Portage to drop root privileges while fetching package sources and run with user/group permissions of portage:portage. This is a small security improvement.

If `userfetch` is set in *FEATURES* be sure to change the owner of all the files beneath /var/db/repos/gentoo using the **chown** command with root privileges:

```sh 
root # chown --recursive --verbose portage:portage /var/db/repos/gentoo
```

### Verify distfiles

To re-verify the integrity and (potentially) re-download previously removed/corrupted distfiles for all currently installed packages, run:
``` sh
root # emerge --ask --fetchonly --emptytree @world
``` 
