# Installing the Gentoo base system

## Chrooting

### Optional: Selecting mirrors

#### Distribution files

!!! Tip
It is safe to skip this step when using non-Gentoo installation media. The [app-portage/mirrorselect](https://packages.gentoo.org/packages/app-portage/mirrorselect) package can be emerged later within the stage3 (after [Entering the new environment](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Entering_the_new_environment)) and the actions defined in this section can be performed at that point.

In order to download source code quickly it is recommended to select a fast mirror. Portage will look in the make.conf file for the *GENTOO_MIRRORS* variable and use the mirrors listed therein. It is possible to surf to the Gentoo mirror list and search for a mirror (or mirrors) that is close to the system's physical location (as those are most frequently the fastest ones). However, we provide a nice tool called **mirrorselect** which provides users with a nice interface to select the mirrors needed. Just navigate to the mirrors of choice and press `Spacebar` to select one or more mirrors.

`root # mirrorselect -i -o >> /mnt/gentoo/etc/portage/make.conf`

#### Gentoo ebuild repository

A second important step in selecting mirrors is to configure the Gentoo ebuild repository via the /etc/portage/repos.conf/gentoo.conf file. This file contains the sync information needed to update the package repository (the collection of ebuilds and related files containing all the information Portage needs to download and install software packages).

Configuring the repository can be done in a few simple steps. First, if it does not exist, create the [repos.conf](https://wiki.gentoo.org/wiki//etc/portage/repos.conf) directory:

`root # mkdir --parents /mnt/gentoo/etc/portage/repos.conf`

Next, copy the Gentoo repository configuration file provided by Portage to the (newly created) repos.conf directory:

`root # cp /mnt/gentoo/usr/share/portage/config/repos.conf /mnt/gentoo/etc/portage/repos.conf/gentoo.conf`

Take a peek with a text editor or by using the cat command. The inside of the file should be in .ini format and look like this:

```sh title="FILE /mnt/gentoo/etc/portage/repos.conf/gentoo.conf"
[DEFAULT]
main-repo = gentoo
 
[gentoo]
location = /var/db/repos/gentoo
sync-type = rsync
sync-uri = rsync://rsync.gentoo.org/gentoo-portage
auto-sync = yes
sync-rsync-verify-jobs = 1
sync-rsync-verify-metamanifest = yes
sync-rsync-verify-max-age = 24
sync-openpgp-key-path = /usr/share/openpgp-keys/gentoo-release.asc
sync-openpgp-key-refresh-retry-count = 40
sync-openpgp-key-refresh-retry-overall-timeout = 1200
sync-openpgp-key-refresh-retry-delay-exp-base = 2
sync-openpgp-key-refresh-retry-delay-max = 60
sync-openpgp-key-refresh-retry-delay-mult = 4
```

The default *sync-uri* variable value listed above will determine a mirror location based on a rotation. This will aid in easing bandwidth stress on Gentoo's infrastructure and will provide a fail-safe in case a specific mirror is offline. It is recommended the default URI is retained unless a local, private Portage mirror will be used.

!!! Tip

The specification for Portage's plug-in sync API can be found in the Portage Sync article.

### Copy DNS info

One thing still remains to be done before entering the new environment and that is copying over the DNS information in /etc/resolv.conf. This needs to be done to ensure that networking still works even after entering the new environment. /etc/resolv.conf contains the name servers for the network.

To copy this information, it is recommended to pass the `--dereference` option to the **cp** command. This ensures that, if /etc/resolv.conf is a symbolic link, that the link's target file is copied instead of the symbolic link itself. Otherwise in the new environment the symbolic link would point to a non-existing file (as the link's target is most likely not available inside the new environment).

`root # cp --dereference /etc/resolv.conf /mnt/gentoo/etc/`

### Mounting the necessary filesystems

In a few moments, the Linux root will be changed towards the new location.

The filesystems that need to be made available are:

- /proc/ is a pseudo-filesystem. It looks like regular files, but is generated on-the-fly by the Linux kernel
- /sys/ is a pseudo-filesystem, like /proc/ which it was once meant to replace, and is more structured than /proc/
- /dev/ is a regular file system which contains all device. It is partially managed by the Linux device manager (usually **udev**)
- /run/ is a temporary file system used for files generated at runtime, such as PID files or locks

The /proc/ location will be mounted on /mnt/gentoo/proc/ whereas the others are bind-mounted. The latter means that, for instance, /mnt/gentoo/sys/ will actually be /sys/ (it is just a second entry point to the same filesystem) whereas /mnt/gentoo/proc/ is a new mount (instance so to speak) of the filesystem.

``` sh
root #mount --types proc /proc /mnt/gentoo/proc
root #mount --rbind /sys /mnt/gentoo/sys
root #mount --make-rslave /mnt/gentoo/sys
root #mount --rbind /dev /mnt/gentoo/dev
root #mount --make-rslave /mnt/gentoo/dev
root #mount --bind /run /mnt/gentoo/run
root #mount --make-slave /mnt/gentoo/run
```

!!! Note
The `--make-rslave` operations are needed for systemd support later in the installation.

!!! Warning
When using non-Gentoo installation media, this might not be sufficient. Some distributions make /dev/shm a symbolic link to /run/shm/ which, after the chroot, becomes invalid. Making /dev/shm/ a proper tmpfs mount up front can fix this:

`root # test -L /dev/shm && rm /dev/shm && mkdir /dev/shm`
`root # mount --types tmpfs --options nosuid,nodev,noexec shm /dev/shm`

Also ensure that mode 1777 is set:

`root # chmod 1777 /dev/shm /run/shm`


### Entering the new environment

Now that all partitions are initialized and the base environment installed, it is time to enter the new installation environment by chrooting into it. This means that the session will change its root (most top-level location that can be accessed) from the current installation environment (installation CD or other installation medium) to the installation system (namely the initialized partitions). Hence the name, *change root* or *chroot*.

This chrooting is done in three steps:

1. The root location is changed from / (on the installation medium) to /mnt/gentoo/ (on the partitions) using chroot
2. Some settings (those in /etc/profile) are reloaded in memory using the source command
3. The primary prompt is changed to help us remember that this session is inside a chroot environment.

``` sh
root # chroot /mnt/gentoo /bin/bash
root # source /etc/profile
root # export PS1="(chroot) ${PS1}"
```

From this point, all actions performed are immediately on the new Gentoo Linux environment.

!!! Tip
If the Gentoo installation is interrupted anywhere after this point, it should be possible to 'resume' the installation at this step. There is no need to repartition the disks again! Simply mount the root partition and run the steps above starting with copying the DNS info to re-enter the working environment. This is also useful for fixing bootloader issues. More information can be found in the chroot article.

### Mounting the boot partition

Now that the new environment has been entered, it is necessary to mount the boot partition. This will be important when it is time to compile the kernel and install the bootloader:

`root #mount /dev/sda1 /boot`

## Configuring Portage

### Installing a Gentoo ebuild repository snapshot from the web

Next step is to install a snapshot of the Gentoo ebuild repository. This snapshot contains a collection of files that informs Portage about available software titles (for installation), which profiles the system administrator can select, package or profile specific news items, etc.

The use of **emerge-webrsync** is recommended for those who are behind restrictive firewalls (it uses HTTP/FTP protocols for downloading the snapshot) and saves network bandwidth. Readers who have no network or bandwidth restrictions can happily skip down to the next section.

This will fetch the latest snapshot (which is released on a daily basis) from one of Gentoo's mirrors and install it onto the system:

`root # emerge-webrsync`

!!! Note
During this operation, emerge-webrsync might complain about a missing /var/db/repos/gentoo/ location. This is to be expected and nothing to worry about - the tool will create the location.


From this point onward, Portage might mention that certain updates are recommended to be executed. This is because system packages installed through the stage file might have newer versions available; Portage is now aware of new packages because of the repository snapshot. Package updates can be safely ignored for now; updates can be delayed until after the Gentoo installation has finished.

### Optional: Updating the Gentoo ebuild repository

It is possible to update the Gentoo ebuild repository to the latest version. The previous **emerge-webrsync** command will have installed a very recent snapshot (usually recent up to 24h) so this step is definitely optional.

Suppose there is a need for the last package updates (up to 1 hour), then use **emerge --sync**. This command will use the rsync protocol to update the Gentoo ebuild repository (which was fetched earlier on through emerge-webrsync) to the latest state.

`root # emerge --sync`

On slow terminals, like some framebuffers or serial consoles, it is recommended to use the --quiet option to speed up the process:

`root # emerge --sync --quiet`

### Reading news items

When the Gentoo ebuild repository is synchronized, Portage may output informational messages similar to the following:

```sh 
* IMPORTANT: 2 news items need reading for repository 'gentoo'.
* Use eselect news to read news items.
```

News items were created to provide a communication medium to push critical messages to users via the Gentoo ebuild repository. To manage them, use **eselect news**. The **eselect** application is a Gentoo-specific utility that allows for a common management interface for system administration. In this case, **eselect** is asked to use its news module.

For the `news` module, three operations are most used:

With `list` an overview of the available news items is displayed.
With `read` the news items can be read.
With `purge` news items can be removed once they have been read and will not be reread anymore.

```
root #eselect news list
root #eselect news read
```

More information about the news reader is available through its manual page:

`root # man news.eselect`

### Choosing the right profile
 
!!! Tip
Desktop profiles are not exclusively for desktop environments. They are still suitable for minimal window managers like i3 or sway.

A *profile* is a building block for any Gentoo system. Not only does it specify default values for USE, CFLAGS, and other important variables, it also locks the system to a certain range of package versions. These settings are all maintained by Gentoo's Portage developers.

To see what profile the system is currently using, run **eselect** using the `profile` module:

```sh
root # eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/17.1 *
  [2]   default/linux/amd64/17.1/desktop
  [3]   default/linux/amd64/17.1/desktop/gnome
  [4]   default/linux/amd64/17.1/desktop/kde
```

!!! Note
The output of the command is just an example and evolves over time.

!!! Note
To use systemd, select a profile which has "systemd" in the name and vice versa, if not

There are also desktop subprofiles available for some architectures.

!!! Warning
Profile upgrades are not to be taken lightly. When selecting the initial profile, use the profile corresponding to the same version as the one initially used by stage3 (e.g. 17.1). Each new profile version is announced through a news item containing migration instructions. Follow the instructions before switching to a newer profile.

After viewing the available profiles for the amd64 architecture, users can select a different profile for the system:

`root # eselect profile set 2`

#### No-multilib

In order to select a pure 64-bit environment, with no 32-bit applications or libraries, use a no-multilib profile:

```sh
root # eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/17.1 *
  [2]   default/linux/amd64/17.1/desktop
  [3]   default/linux/amd64/17.1/desktop/gnome
  [4]   default/linux/amd64/17.1/desktop/kde
  [5]   default/linux/amd64/17.1/no-multilib
```
Next select the *no-multilib* profile:

```sh
root #eselect profile set 5
root #eselect profile list
Available profile symlink targets:
  [1]   default/linux/amd64/17.1
  [2]   default/linux/amd64/17.1/desktop
  [3]   default/linux/amd64/17.1/desktop/gnome
  [4]   default/linux/amd64/17.1/desktop/kde
  [5]   default/linux/amd64/17.1/no-multilib *
```

!!! Note
The `developer` subprofile is specifically for Gentoo Linux development and is not meant to be used by casual users.

### Updating the @world set

At this point, it is wise to update the system's [@world](https://wiki.gentoo.org/wiki/World_set_(Portage)) set so that a base can be established.

This following step is necessary so the system can apply any updates or USE flag changes which have appeared since the stage3 was built and from any profile selection:

`root #emerge --ask --verbose --update --deep --newuse @world`

!!! Tip
If a full scale desktop environment profile has been selected this process could greatly extend the amount of time necessary for the install process. Those in a time crunch can work by this 'rule of thumb': the shorter the profile name, the less specific the system's @world set; the less specific the [@world](https://wiki.gentoo.org/wiki/World_set_(Portage)) set, the fewer packages the system will require. In other words:
Selecting default/linux/amd64/17.1 will require very few packages to be updated, whereas
Selecting default/linux/amd64/17.1/desktop/gnome/systemd will require many packages to be installed since the init system is changing from OpenRC to systemd, and the GNOME desktop environment framework will be installed.

### Configuring the USE variable

*USE* is one of the most powerful variables Gentoo provides to its users. Several programs can be compiled with or without optional support for certain items. For instance, some programs can be compiled with support for GTK+ or with support for Qt. Others can be compiled with or without SSL support. Some programs can even be compiled with framebuffer support (svgalib) instead of X11 support (X-server).

Most distributions compile their packages with support for as much as possible, increasing the size of the programs and startup time, not to mention an enormous amount of dependencies. With Gentoo users can define what options a package should be compiled with. This is where *USE* comes into play.

In the *USE* variable users define keywords which are mapped onto compile-options. For instance, ssl will compile SSL support in the programs that support it. -X will remove X-server support (note the minus sign in front). gnome gtk -kde -qt5 will compile programs with GNOME (and GTK+) support, and not with KDE (and Qt) support, making the system fully tweaked for GNOME (if the architecture supports it).

The default USE settings are placed in the make.defaults files of the Gentoo profile used by the system. Gentoo uses a (complex) inheritance system for its profiles, which we will not dive into at this stage. The easiest way to check the currently active USE settings is to run **emerge --info** and select the line that starts with *USE*:

```sh
root $ emerge --info | grep ^USE
USE="X acl alsa amd64 berkdb bindist bzip2 cli cracklib crypt cxx dri ..."
```

!!! Note
The above example is truncated, the actual list of USE values is much, much larger.
A full description on the available USE flags can be found on the system in /var/db/repos/gentoo/profiles/use.desc.

`root $ less /var/db/repos/gentoo/profiles/use.desc`

Inside the **less** command, scrolling can be done using the ↑ and ↓ keys, and exited by pressing q.

As an example we show a USE setting for a KDE-based system with DVD, ALSA, and CD recording support:

root #nano -w /etc/portage/make.conf
FILE /etc/portage/make.confEnabling flags for a KDE/Plasma-based system with DVD, ALSA, and CD recording support
USE="-gtk -gnome qt5 kde dvd alsa cdr"
When a USE value is defined in /etc/portage/make.conf it is added to the system's USE flag list. USE flags can be globally removed by adding a - minus sign in front of the value in the the list. For example, to disable support for X graphical environments, -X can be set:

FILE /etc/portage/make.confIgnoring default USE flags
USE="-X acl alsa"
 Warning
Although possible, setting -* (which will disable all USE values except the ones specified in make.conf) is strongly discouraged and unwise. Ebuild developers choose certain default USE flag values in ebuilds in order to prevent conflicts, enhance security, and avoid errors, and other reasons. Disabling all USE flags will negate default behavior and may cause major issues.
CPU_FLAGS_*
Some architectures (including AMD64/X86, ARM, PPC) have a USE_EXPAND variable called CPU_FLAGS_ARCH (replace ARCH with the relevant system architecture as appropriate).

This is used to configure the build to compile in specific assembly code or other intrinsics, usually hand-written or otherwise extra, and is not the same as asking the compiler to output optimized code for a certain CPU feature (e.g. -march=).

Users should set this variable in addition to configuring their COMMON_FLAGS as desired.

A few steps are needed to set this up:

root #emerge --ask app-portage/cpuid2cpuflags
Inspect the output manually if curious:

root #cpuid2cpuflags
Then copy the output into package.use:

root #echo "*/* $(cpuid2cpuflags)" > /etc/portage/package.use/00cpu-flags
VIDEO_CARDS
The VIDEO_CARDS USE_EXPAND variable should be configured appropriately depending on the available GPU(s). The Xorg guide covers how to do this. Setting VIDEO_CARDS is not required for a console only install.

Optional: Configure the ACCEPT_LICENSE variable
The licenses of a Gentoo package are stored in the LICENSE variable in the ebuild. The accepted specific licenses or groups of licenses of a system are defined in the following files:

System wide in the selected profile.
System wide in the /etc/portage/make.conf file.
Per-package in a /etc/portage/package.license file.
Per-package in a /etc/portage/package.license/ directory of files.
Portage looks up in the ACCEPT_LICENSE which packages to allow for installation. In order to print the current system wide value run:

user $portageq envvar ACCEPT_LICENSE
@FREE
Optionally override the system wide accepted default in the profiles by changing /etc/portage/make.conf.

FILE /etc/portage/make.confExample how to accept licenses with ACCEPT_LICENSE system wide
ACCEPT_LICENSE="-* @FREE @BINARY-REDISTRIBUTABLE"
Optionally one can also define accepted licenses per-package as shown in the following directory of files example. Note that the package.license directory will need created if it does not already exist:

root #mkdir /etc/portage/package.license
FILE /etc/portage/package.license/kernelExample how to accept licenses per-package
app-arch/unrar unRAR
sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE
sys-firmware/intel-microcode intel-ucode
 Important
The LICENSE variable in an ebuild is only a guideline for Gentoo developers and users. It is not a legal statement, and there is no guarantee that it will reflect reality. So don't rely on it, but check the package itself in depth, including all files that have been installed to the system.
The license groups defined in the Gentoo repository, managed by the Gentoo Licenses project, are:

Group Name	Description
@GPL-COMPATIBLE	GPL compatible licenses approved by the Free Software Foundation [a_license 1]
@FSF-APPROVED	Free software licenses approved by the FSF (includes @GPL-COMPATIBLE)
@OSI-APPROVED	Licenses approved by the Open Source Initiative [a_license 2]
@MISC-FREE	Misc licenses that are probably free software, i.e. follow the Free Software Definition [a_license 3] but are not approved by either FSF or OSI
@FREE-SOFTWARE	Combines @FSF-APPROVED, @OSI-APPROVED and @MISC-FREE
@FSF-APPROVED-OTHER	FSF-approved licenses for "free documentation" and "works of practical use besides software and documentation" (including fonts)
@MISC-FREE-DOCS	Misc licenses for free documents and other works (including fonts) that follow the free definition [a_license 4] but are NOT listed in @FSF-APPROVED-OTHER
@FREE-DOCUMENTS	Combines @FSF-APPROVED-OTHER and @MISC-FREE-DOCS
@FREE	Metaset of all licenses with the freedom to use, share, modify and share modifications. Combines @FREE-SOFTWARE and @FREE-DOCUMENTS
@BINARY-REDISTRIBUTABLE	Licenses that at least permit free redistribution of the software in binary form. Includes @FREE
@EULA	License agreements that try to take away your rights. These are more restrictive than "all-rights-reserved" or require explicit approval
 https://www.gnu.org/licenses/license-list.html
 https://www.opensource.org/licenses
 https://www.gnu.org/philosophy/free-sw.html
 https://freedomdefined.org/

Optional: Using systemd as the init system
The remainder of the Gentoo handbook will provide systemd steps alongside OpenRC (the traditional Gentoo init system) where separate steps or recommendations are necessary. System administrators should also consult the systemd article for more details on managing systemd as the system and service manager.

Timezone
 Note
This step does not apply to users of the musl libc. Users who do not know what that means should perform this step.
Select the timezone for the system. Look for the available timezones in /usr/share/zoneinfo/:

root #ls /usr/share/zoneinfo
Suppose the timezone of choice is Europe/Brussels.

OpenRC
We write the timezone name into the /etc/timezone file.

root #echo "Europe/Brussels" > /etc/timezone
Please avoid the /usr/share/zoneinfo/Etc/GMT* timezones as their names do not indicate the expected zones. For instance, GMT-8 is in fact GMT+8.

Next, reconfigure the sys-libs/timezone-data package, which will update the /etc/localtime file for us, based on the /etc/timezone entry. The /etc/localtime file is used by the system C library to know the timezone the system is in.

root #emerge --config sys-libs/timezone-data
systemd
A slightly different approach is employed when using systemd. A symbolic link is generated:

root #ln -sf ../usr/share/zoneinfo/Europe/Brussels /etc/localtime
Later, when systemd is running, the timezone and related settings can be configured with the timedatectl command.

Configure locales
 Note
This step does not apply to users of the musl libc. Users who do not know what that means should perform this step.
Locale generation
Most users will want to use only one or two locales on their system.

Locales specify not only the language that the user should use to interact with the system, but also the rules for sorting strings, displaying dates and times, etc. Locales are case sensitive and must be represented exactly as described. A full listing of available locales can be found in the /usr/share/i18n/SUPPORTED file.

Supported system locales must be defined in the /etc/locale.gen file.

root #nano -w /etc/locale.gen
The following locales are an example to get both English (United States) and German (Germany/Deutschland) with the accompanying character formats (like UTF-8).

FILE /etc/locale.genEnabling US and DE locales with the appropriate character formats
en_US ISO-8859-1
en_US.UTF-8 UTF-8
de_DE ISO-8859-1
de_DE.UTF-8 UTF-8
 Warning
Many applications require least one UTF-8 locale to build properly.
The next step is to run the locale-gen command. This command generates all locales specified in the /etc/locale.gen file.

root #locale-gen
To verify that the selected locales are now available, run locale -a.

On systemd installs, localectl can be used, e.g. localectl set-locale ... or localectl list-locales.

Locale selection
Once done, it is now time to set the system-wide locale settings. Again we use eselect for this, now with the locale module.

With eselect locale list, the available targets are displayed:

root #eselect locale list
Available targets for the LANG variable:
  [1]  C
  [2]  C.utf8
  [3]  en_US
  [4]  en_US.iso88591
  [5]  en_US.utf8
  [6]  de_DE
  [7]  de_DE.iso88591
  [8]  de_DE.iso885915
  [9]  de_DE.utf8
  [10] POSIX
  [ ]  (free form)
With eselect locale set <NUMBER> the correct locale can be selected:

root #eselect locale set 9
Manually, this can still be accomplished through the /etc/env.d/02locale file and for Systemd the /etc/locale.conf file:

FILE /etc/env.d/02localeManually setting system locale definitions
LANG="de_DE.UTF-8"
LC_COLLATE="C.UTF-8"
Setting the locale will avoid warnings and errors during kernel and software compilations later in the installation.

Now reload the environment:

root #env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
For additional guidance through the locale selection process read also the Localization guide and the UTF-8 guide.