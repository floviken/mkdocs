# Working with Gentoo

## Welcome to Portage

Portage is one of Gentoo's most notable innovations in software management. With its high flexibility and enormous amount of features it is frequently seen as the best software management tool available for Linux.

Portage is completely written in [Python](https://www.python.org/) and [Bash](https://www.gnu.org/software/bash) and therefore fully visible to the users as both are scripting languages.

Most users will work with Portage through the **emerge** tool. This chapter is not meant to duplicate the information available from the emerge man page. For a complete rundown of emerge's options, please consult the man page:

``` shell
user $ man emerge
```

## Gentoo repository

### Ebuilds
When Gentoo's documentation talks about packages, it means software titles that are available to the Gentoo users through the Gentoo repository. This repository is a collection of ebuilds, files that contain all information Portage needs to maintain software (install, search, query, etc.). These ebuilds reside in `/var/db/repos/gentoo` by default.

Whenever someone asks Portage to perform some action regarding software titles, it will use the ebuilds on the system as a base. It is therefore important to regularly update the ebuilds on the system so Portage knows about new software, security updates, etc.

### Updating the Gentoo repository
The Gentoo repository is usually updated with **rsync**, a fast incremental file transfer utility. Updating is fairly simple as the **emerge** command provides a front-end for **rsync**:

``` shell
root # emerge --sync
```

Sometimes firewall restrictions apply that prevent **rsync** from contacting the mirrors. In this case, update the Gentoo repository through Gentoo's daily generated snapshots. The **emerge-webrsync** tool automatically fetches and installs the latest snapshot on the system:

``` shell
root # emerge-webrsync
```

An additional advantage of using emerge-webrsync is that it allows the administrator to only pull in Gentoo repository snapshots that are signed by the Gentoo release engineering GPG key. More information on this can be found in the Portage features section on [fetching validated Gentoo repository snapshots](##Portage features
).

## Maintaining software

### Searching for software

There are multiple ways to search through the Gentoo repository for software. One way is through **emerge** itself. By default,  **emerge --search** returns the names of packages whose title matches (either fully or partially) the given search term.

For instance, to search for all packages who have "pdf" in their name:

``` shell
user $ emerge --search pdf
```

To search through the descriptions as well, use the `--searchdesc` (or `-S`) option:

``` shell
user $ emerge --searchdesc pdf
```

Notice that the output returns a lot of information. The fields are clearly labelled so we won't go further into their meanings:

``` shell
CODE Example output for a search command
*  net-print/cups-pdf
      Latest version available: 1.5.2
      Latest version installed: [ Not Installed ]
      Size of downloaded files: 15 kB
      Homepage:    http://cip.physik.uni-wuerzburg.de/~vrbehr/cups-pdf/
      Description: Provides a virtual printer for CUPS to produce PDF files.
      License:     GPL-2
```

### Installing software

When a software title has been found, then the installation is just one **emerge** command away. For instance, to install gnumeric:

``` shell
root # emerge --ask app-office/gnumeric
```

Since many applications depend on each other, any attempt to install a certain software package might result in the installation of several dependencies as well. Don't worry, Portage handles dependencies well. To find out what Portage would install, add the `--pretend` option. For instance:

``` shell
root # emerge --pretend gnumeric
```
To do the same, but interactively choose whether or not to proceed with the installation, add the `--ask` flag:

``` shell
root # emerge --ask gnumeric
``` 

During the installation of a package, Portage will download the necessary source code from the Internet (if necessary) and store it by default in `/var/cache/distfiles/`. After this it will unpack, compile and install the package. To tell Portage to only download the sources without installing them, add the `--fetchonly` option to the emerge command:

``` shell
root # emerge --fetchonly gnumeric
```

### Finding installed package documentation

Many packages come with their own documentation. Sometimes, the doc USE flag determines whether the package documentation should   be installed or not. To see if the doc USE flag is used by a package, use **emerge -vp category/package**:

``` shell
root # emerge -vp media-libs/alsa-lib
These are the packages that would be merged, in order:
 
Calculating dependencies... done!
[ebuild   R    ] media-libs/alsa-lib-1.1.3::gentoo  USE="python -alisp -debug -doc" ABI_X86="(64) -32 (-x32)" PYTHON_TARGETS="python2_7" 0 KiB
```

The best way of enabling the `doc` USE flag is doing it on a per-package basis via `/etc/portage/package.use`, so that only the documentation for the wanted packages is installed. For more information read the [USE flags](/docs/gentoo/working/02-useflags.md) section.

Once the package installed, its documentation is generally found in a subdirectory named after the package in the `/usr/share/doc/` directory:

``` shell 
user $ ls -l /usr/share/doc/alsa-lib-1.1.3
total 16
-rw-r--r-- 1 root root 3098 Mar  9 15:36 asoundrc.txt.bz2
-rw-r--r-- 1 root root  672 Mar  9 15:36 ChangeLog.bz2
-rw-r--r-- 1 root root 1083 Mar  9 15:36 NOTES.bz2
-rw-r--r-- 1 root root  220 Mar  9 15:36 TODO.bz2
```

A more sure way to list installed documentation files is to use **equery**'s `--filter` option. **equery** is used to query Portage's database and comes as part of the [app-portage/gentoolkit](https://packages.gentoo.org/packages/app-portage/gentoolkit) package:

``` shell
user $ equery files --filter=doc alsa-lib
 * Searching for alsa-lib in media-libs ...
 * Contents of media-libs/alsa-lib-1.1.3:
/usr/share/doc/alsa-lib-1.1.3/ChangeLog.bz2
/usr/share/doc/alsa-lib-1.1.3/NOTES.bz2
/usr/share/doc/alsa-lib-1.1.3/TODO.bz2
/usr/share/doc/alsa-lib-1.1.3/asoundrc.txt.bz2
```

The `--filter` option can be used with other rules to view the install locations for many other types of files. Additional functionality can be reviewed in equery's man page: **man 1 equery**.

### Removing software

To safely remove software from a system, use `emerge --deselect`. This will tell Portage a package is no longer required and it is eligible for cleaning through `--depclean`.

``` shell
root #emerge --deselect gnumeric
```

When a package is no longer selected, the package and its dependencies that were installed automatically when it was installed are still left on the system. To have Portage locate all dependencies that can now be removed, use **emerge**'s `--depclean` functionality, which is documented later.

### Updating the system

To keep the system in perfect shape (and not to mention install the latest security updates) it is necessary to update the system regularly. Since Portage only checks the ebuilds in the Gentoo repository, the first thing to do is to update this repository using `emerge --sync`. Then the system can be updated using `emerge --deep --update @world`.

Portage will, with `--deep`, search for newer version of the applications that are installed. Without `--deep`, it will only verify the versions for the applications that are explicitly installed (the applications listed in `/var/lib/portage/world`) - it does not thoroughly check their dependencies. This option should almost always therefore be used:

``` shell
root # emerge --update --deep @world
```

The standard upgrade command should include `--changed-use` or `--newuse` because of possible changes within the repository's profiles, or if the USE settings of the system have been altered. Portage will then verify if the change requires the installation of new packages or recompilation of existing ones:

```shell
root #emerge --update --deep --newuse @world
```

### Metapackages

Some packages in the Gentoo repository don't have any real content but are used to install a collection of packages. For instance, the [kde-plasma/plasma-meta](https://packages.gentoo.org/packages/kde-plasma/plasma-meta) package will install the KDE Plasma desktop on the system by pulling in various Plasma-related packages as dependencies.

To remove such a package from the system, running **emerge --deselect** on the package will not have much effect since the dependencies for the package remain on the system.

Portage has the functionality to remove orphaned dependencies as well, but since the availability of software is dynamically dependent it is important to first update the entire system fully, including the new changes applied when changing USE flags. After this one can run **emerge --depclean** to remove the orphaned dependencies. When this is done, it might be necessary to rebuild the applications that were dynamically linked to the now-removed software titles but don't require them anymore, although recently support for this has been added to Portage.

All this is handled with the following two commands:

``` shell
root # emerge --update --deep --newuse @world
root # emerge --ask --depclean
```

## Licenses

Beginning with Portage version 2.1.7, it is possible to accept or reject software installation based on its license. All packages in the tree contain a *LICENSE* entry in their ebuilds. Running **emerge --search category/package** will show the package's license.

!!! Important
```markdown
As a disclaimer and limitation of liability, the LICENSE variable in an ebuild is merely a guideline for Gentoo developers and users. It is *not* a legal statement or a guarantee that it will reflect the license of every file installed by an ebuild. It should not be relied upon for a completely accurate legal representation of all files provided by a package. To gain assurance, system administrators should perform an in-depth check of each file installed by a package for proper licensing alignment and/or compliance. If a discrepancies is found in the ebuild, please file a bug to suggest a change to the value(s) assigned to the ebuild's LICENSE variable.
```

By default, Portage permits licenses that are explicitly approved by the [Free Software Foundation](https://www.gnu.org/licenses/license-list.html), the [Open Source Initiative](https://opensource.org/licenses), or that follow the [Free Software Definition](https://www.gnu.org/philosophy/free-sw.html).


The variable that controls permitted licenses is called *ACCEPT_LICENSE*, which can be set in the `/etc/portage/make.conf` file. In the next example, this default value is shown:

``` shell title="FILE /etc/portage/make.conf  The default ACCEPT_LICENSE setting"
ACCEPT_LICENSE="-* @FREE"
```

With this configuration, packages with a free software or documentation license will be installable. Non-free software will not be installable.

It is possible to set *ACCEPT_LICENSE* globally in /etc/portage/make.conf, or to specify it on a per-package basis in the /etc/portage/package.license file.

For example, to allow the google-chrome license for the www-client/google-chrome package, add the following to /etc/portage/package.license:

``` shell title="FILE /etc/portage/package.license  Accepting the google-chrome license for the google-chrome package"
www-client/google-chrome google-chrome
```

This permits the installation of the www-client/google-chrome package, but prohibits the installation of the www-plugins/chrome-binary-plugins package, even though it has the same license.


Or to allow the often-needed sys-kernel/linux-firmware:

``` shell title="FILE /etc/portage/package.licenseAccepting the licenses for the linux-firmware package"
# Accepting the license for linux-firmware
sys-kernel/linux-firmware linux-fw-redistributable

# Accepting any license that permits redistribution
sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE
```

```title="Important"
Licenses are stored in /var/db/repos/gentoo/licenses/ directory, and license groups are kept in /var/db/repos/gentoo/profiles/license_groups file. The first entry of each line in CAPITAL letters is the name of the license group, and every entry after that is an individual license.
```

License groups defined in the *ACCEPT_LICENSE* variable are prefixed with an @ sign. A possible setting (which was the previous Portage default) is to allow all licenses, except End User License Agreements (EULAs) that require reading and signing an acceptance agreement. To accomplish this, accept all licenses (using *) and then remove the licenses in the EULA group as follows:

```title="FILE /etc/portage/make.confAccept all licenses except EULAs"
ACCEPT_LICENSE="* -@EULA"
```

Note that this setting will also accept non-free software and documentation.

## When Portage is complaining

### Terminology
As stated before, Portage is extremely powerful and supports many features that other software management tools lack. To understand this, we explain a few aspects of Portage without going into too much detail.

With Portage different versions of a single package can coexist on a system. While other distributions tend to name their package to those versions (like gtk+2 and gtk+3) Portage uses a technology called *SLOT*s. An ebuild declares a certain SLOT for its version. Ebuilds with different SLOTs can coexist on the same system. For instance, the gtk+ package has ebuilds with SLOT="2" and SLOT="3".

There are also packages that provide the same functionality but are implemented differently. For instance, metalogd, sysklogd, and syslog-ng are all system loggers. Applications that rely on the availability of "a system logger" cannot depend on, for instance, metalogd, as the other system loggers are as good a choice as any. Portage allows for virtuals: each system logger is listed as an "exclusive" dependency of the logging service in the logger virtual package of the virtual category, so that applications can depend on the [virtual/logger](https://packages.gentoo.org/packages/virtual/logger) package. When installed, the package will pull in the first logging package mentioned in the package, unless a logging package was already installed (in which case the virtual is satisfied).

Software in the Gentoo repository can reside in different branches. By default the system only accepts packages that Gentoo deems stable. Most new software titles, when committed, are added to the testing branch, meaning more testing needs to be done before it is marked as stable. Although the ebuilds for those software are in the Gentoo repository, Portage will not update them before they are placed in the stable branch.

Some software is only available for a few architectures. Or the software doesn't work on the other architectures, or it needs more testing, or the developer that committed the software to the Gentoo repository is unable to verify if the package works on different architectures.

Each Gentoo installation also adheres to a certain profile which contains, amongst other information, the list of packages that are required for a system to function normally.

### Blocked packages

```title="CODE Portage warning about blocked packages"
[ebuild  N     ] x11-wm/i3-4.20.1  USE="-doc -test"
[blocks B      ] x11-wm/i3 ("x11-wm/i3" is soft blocking x11-wm/i3-gaps-4.20.1)

 * Error: The above package list contains packages which cannot be
 * installed at the same time on the same system.

  (x11-wm/i3-4.20.1:0/0::gentoo, ebuild scheduled for merge) pulled in by
    x11-wm/i3

  (x11-wm/i3-gaps-4.20.1-1:0/0::gentoo, installed) pulled in by
    x11-wm/i3-gaps required by @selected
```

Ebuilds contain specific fields that inform Portage about its dependencies. There are two possible dependencies: build dependencies, declared in the *DEPEND* variable and run-time dependencies, likewise declared in *RDEPEND*. When one of these dependencies explicitly marks a package or virtual as being not compatible, it triggers a blockage.

While recent versions of Portage are smart enough to work around minor blockages without user intervention, occasionally such blockages need to be resolved manually.

To fix a blockage, users can choose to not install the package or unmerge the conflicting package first. In the given example, one can opt not to install x11-wm/i3 or to remove x11-wm/i3-gaps first. It is usually best to simply tell Portage the package is no longer desired, with **emerge --deselect x11-wm/i3-gaps**, for example, to remove it from the world file rather than removing the package itself forcefully.

Sometimes there are also blocking packages with specific atoms, such as `<media-video/mplayer-1.0_rc1-r2`. In this case, updating to a more recent version of the blocking package could remove the block.

It is also possible that two packages that are yet to be installed are blocking each other. In this rare case, try to find out why both would need to be installed. In most cases it is sufficient to do with one of the packages alone. If not, please file a bug on [Gentoo's bug tracking system](https://bugs.gentoo.org/).

### Masked packages

```title="CODE Portage warning about masked packages"
!!! all ebuilds that could satisfy "bootsplash" have been masked.
```

```title="CODE Portage warning about masked packages - reason"

!!! possible candidates are:
  
- gnome-base/gnome-2.8.0_pre1 (masked by: ~x86 keyword)
- lm-sensors/lm-sensors-2.8.7 (masked by: -sparc keyword)
- sys-libs/glibc-2.3.4.20040808 (masked by: -* keyword)
- dev-util/cvsd-1.0.2 (masked by: missing keyword)
- games-fps/unreal-tournament-451 (masked by: package.mask)
- sys-libs/glibc-2.3.2-r11 (masked by: profile)
- net-im/skype-2.1.0.81 (masked by: skype-eula license(s))
```

When trying to install a package that isn't available for the system, this masking error occurs. Users should try installing a different application that is available for the system or wait until the package is marked as available. There is always a reason why a package is masked:


|Reason for mask	|Description |
|:---|---|
|~arch keyword	|The application is not tested sufficiently to be put in the stable branch. Wait a few days or weeks and try again.|
|-arch keyword or -* keyword	| The application does not work on the target architecture. If this is not the case, then please file a bug. |
|missing keyword	|The application has not yet been tested on the target architecture. Ask the architecture porting team to test the package or test it for them and report the findings on Gentoo's Bugzilla website. See /etc/portage/package.accept_keywords and Accepting a keyword for a single package.|
|package.mask	 |The package has been found corrupt, unstable or worse and has been deliberately marked as do-not-use.|
|profile	|The package has been found not suitable for the current profile. The application might break the system if it is installed or is just not compatible with the profile currently in use.|
|license	|The package's license is not compatible with the ACCEPT_LICENSE value. Permit its license or the right license group by setting it in /etc/portage/make.conf or in /etc/portage/package.license.|

### Necessary USE flag changes

```title="CODE Portage warning about USE flag change requirement"
The following USE changes are necessary to proceed:
#required by app-text/happypackage-2.0, required by happypackage (argument)
>=app-text/feelings-1.0.0 test```

The error message might also be displayed as follows, if `--autounmask` isn't set:

```title="CODE Portage error about USE flag change requirement
emerge: there are no ebuilds built with USE flags to satisfy "app-text/feelings[test]".
!!! One of the following packages is required to complete your request:
- app-text/feelings-1.0.0 (Change USE: +test)
(dependency required by "app-text/happypackage-2.0" [ebuild])
(dependency required by "happypackage" [argument])
```

Such warning or error occurs when a package is requested for installation which not only depends on another package, but also requires that that package is built with a particular USE flag (or set of USE flags). In the given example, the package app-text/feelings needs to be built with USE="test", but this USE flag is not set on the system.

To resolve this, either add the requested USE flag to the global USE flags in /etc/portage/make.conf, or set it for the specific package in /etc/portage/package.use.

### Missing dependencies

``` shell title="CODE Portage warning about missing dependency"
emerge: there are no ebuilds to satisfy ">=sys-devel/gcc-3.4.2-r4".
  
!!! Problem with ebuild sys-devel/gcc-3.4.2-r2
!!! Possibly a DEPEND/*DEPEND problem.
```

The application to install depends on another package that is not available for the system. Please check Bugzilla if the issue is known and if not, please report it. Unless the system is configured to mix branches, this should not occur and is therefore a bug.

### Ambiguous ebuild name

```shell title="CODE Portage warning about ambiguous ebuild names"
[ Results for search key : listen ]
[ Applications found : 2 ]
  
*  dev-tinyos/listen [ Masked ]
      Latest version available: 1.1.15
      Latest version installed: [ Not Installed ]
      Size of files: 10,032 kB
      Homepage:      http://www.tinyos.net/
      Description:   Raw listen for TinyOS
      License:       BSD
  
*  media-sound/listen [ Masked ]
      Latest version available: 0.6.3
      Latest version installed: [ Not Installed ]
      Size of files: 859 kB
      Homepage:      http://www.listen-project.org
      Description:   A Music player and management for GNOME
      License:       GPL-2
  
!!! The short ebuild name "listen" is ambiguous. Please specify
!!! one of the above fully-qualified ebuild names instead.
```

The application that is selected for installation has a name that corresponds with more than one package. Supply the category name as well to resolve this. Portage will inform the user about possible matches to choose from.

### Circular dependencies

``` shell title="CODE Portage warning about circular dependencies"
!!! Error: circular dependencies: 
  
ebuild / net-print/cups-1.1.15-r2 depends on ebuild / app-text/ghostscript-7.05.3-r1
ebuild / app-text/ghostscript-7.05.3-r1 depends on ebuild / net-print/cups-1.1.15-r2
```

Two (or more) packages to install depend on each other and can therefore not be installed. This is most likely a bug in one of the packages in the Gentoo repository. Please re-sync after a while and try again. It might also be beneficial to check [Bugzilla](https://bugs.gentoo.org/) to see if the issue is known and if not, report it.

### Fetch failed

``` shell title="CODE Portage warning about fetch failed"
!!! Fetch failed for sys-libs/ncurses-5.4-r5, continuing...
(...)
!!! Some fetch errors were encountered.  Please see above for details.
```

Portage was unable to download the sources for the given application and will try to continue installing the other applications (if applicable). This failure can be due to a mirror that has not synchronized correctly or because the ebuild points to an incorrect location. The server where the sources reside can also be down for some reason.

Retry after one hour to see if the issue still persists.

### System profile protection

```shell title="CODE Portage warning about profile-protected package"
!!! Trying to unmerge package(s) in system profile. 'sys-apps/portage'
!!! This could be damaging to your system.
```
The user has asked to remove a package that is part of the system's core packages. It is listed in the profile as required and should therefore not be removed from the system.

### Digest verification failure

``` shell title="CODE Digest verification failure"
>>> checking ebuild checksums
!!! Digest verification failed:
```

This is a sign that something is wrong with the Gentoo repository - often, caused by a mistake made when committing an ebuild to the Gentoo ebuild repository.

When the digest verification fails, do not try to re-digest the package personally. Running **ebuild foo manifest** will not fix the problem; it quite possibly could make it worse.

Instead, wait an hour or two for the repository to settle down. It is likely that the error was noticed right away, but it can take a little time for the fix to trickle down the rsync mirrors. Check [Bugzilla](https://bugs.gentoo.org/) and see if anyone has reported the problem yet or ask around on [#gentoo ](ircs://irc.libera.chat/#gentoo) [(webchat)](https://web.libera.chat/#gentoo) (IRC). If not, go ahead and file a bug for the broken ebuild.


Once the bug has been fixed, re-sync the Gentoo ebuild repository to pick up the fixed digest.

```markdown title="Important"
Be careful to not sync the Gentoo ebuild repository more than once a day. As stated in the official Gentoo netiquette policy (as well as when running emerge --sync), users who sync too often will be soft-banned from additional syncs for a time. Abusers who repeatedly fail to follow this policy may be hard-banned. Unless absolutely necessary it is often best to wait for a 24 hours period to sync so that re-synchronization does not overload Gentoo's rsync mirrors.
```