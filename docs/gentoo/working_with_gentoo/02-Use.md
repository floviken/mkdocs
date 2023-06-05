# What are USE flags

## The idea behind USE flags

When installing Gentoo, users make choices depending on the environment they are working with. A setup for a server differs from a setup for a workstation. A gaming workstation differs from a 3D rendering workstation.

This is not only true for choosing what packages to install, but also what features a certain package should support. If there is no need for OpenGL, why would someone bother to install and maintain OpenGL and build OpenGL support in most of the packages? If someone doesn't want to use KDE, why would they bother compiling packages with KDE support if those packages work flawlessly without?

To help users in deciding what to install/activate and what not, Gentoo wanted the user to specify his/her environment in an easy way. This forces the user into deciding what they really want and eases the process for Portage to make useful decisions.


## Definition of a USE flag
Enter the USE flags. Such a flag is a keyword that embodies support and dependency-information for a certain concept. If a certain USE flag is set to enabled, then Portage will know the system administrator desires support for the chosen keyword. Of course this may alter the dependency information for a package. Depending on the USE flag, this may require pulling in *many* more dependencies in order to fulfill the requested dependency changes.

Take a look at a specific example: the `kde` USE flag. If this flag is not set in the *USE* variable (or if the value is prefixed with a minus sign: `-kde`), then all packages that have optional KDE support will be compiled *without* KDE support. All packages that have an optional KDE dependency will be installed *without* installing the KDE libraries (as dependency).

When the `kde` flag is set to enabled, then those packages will be compiled *with* KDE support, and the KDE libraries will be installed as dependency.

By correctly defining USE flags, the system will be tailored specifically to the needs of the system administrator.


## Using USE flags

### Declare permanent USE flags

All USE flags are declared inside the *USE* variable. To make it easy for users to search and pick USE flags, we already provide a default USE setting. This setting is a collection of USE flags we think are commonly used by the Gentoo users. This default setting is declared in the make.defaults files that are part of the selected profile.

The profile the system listens to is pointed to by the /etc/portage/make.profile symlink. Each profile works on top of other profiles, and the end result is therefore the sum of all profiles. The top profile is the base profile (/var/db/repos/gentoo/profiles/base).

To view the currently active USE flags (completely), use **emerge --info**:

``` shell
root # emerge --info | grep ^USE
USE="a52 aac acpi alsa branding cairo cdr dbus dts ..."
```

This variable already contains quite a lot of keywords. Do not alter any make.defaults file to tailor the *USE* variable to personal needs though: changes in these files will be undone when the Gentoo repository is updated!

To change this default setting, add or remove keywords to/from the USE variable. This is done globally by defining the *USE* variable in /etc/portage/make.conf. In this variable one can add the extra *USE* flags required, or remove the USE flags that are no longer needed. This latter is done by prefixing the keyword with the minus-sign (`-`).

For instance, to remove support for KDE and Qt but add support for LDAP, the following USE can be defined in /etc/portage/make.conf:

```shell title="FILE /etc/portage/make.confUpdating USE in make.conf"
USE="-kde -qt5 ldap"
```

### Declaring USE flags for individual packages

Sometimes users want to declare a certain USE flag for one (or a couple) of applications but not system-wide. To accomplish this, edit /etc/portage/package.use. package.use is typically a single file, however it can also be a directory filled with children files; see the tip below and then **man 5 portage** for more information on how to use this convention. The following examples assume package.use is a single file.

For instance, to only have Blu-ray support for the VLC media player package:

``` shell title="FILE /etc/portage/package.useEnabling Blu-ray support for VLC"
media-video/vlc bluray

```

```markdown title="Tip"
If package.use is pre-existing as a directory (opposed to a single file), packages can have their USE flags modified by simply creating files under the package.use/ directory. Any file naming convention can work, however it is wise to implement a coherent naming scheme. One convention is to simply use the package name as the title for the child file. For example, setting the bluray USE flag for the media-video/vlc package can be performed as follows:
``` 
``` shell
root #echo "media-video/vlc bluray" >> /etc/portage/package.use/vlc
```

Similarly it is possible to explicitly disable USE flags for a certain application. For instance, to disable bzip2 support in PHP (but have it for all other packages through the USE flag declaration in make.conf):

``` shell title="FILE /etc/portage/package.use"
Disable bzip2 support for PHP
dev-lang/php -bzip2
```

### Declaring temporary USE flags

Sometimes users need to set a USE flag for a brief moment. Instead of editing /etc/portage/make.conf twice (to do and undo the *USE* changes) just declare the USE variable as an environment variable. Remember that this setting only applies for the command entered; re-emerging or updating this application (either explicitly or as part of a system update) will undo the changes that were triggered through the (temporary) USE flag definition.

The following example temporarily removes the `pulseaudio` value from the USE variable during the installation of SeaMonkey:

```shell
root # USE="-pulseaudio" emerge www-client/seamonkey
```

### Precedence

Of course there is a certain precedence on what setting has priority over the USE setting. The precedence for the USE setting is, ordered by priority (first has lowest priority):

1. Default USE setting declared in the make.defaults files part of your profile
2. User-defined USE setting in /etc/portage/make.conf
3. User-defined USE setting in /etc/portage/package.use
4. User-defined USE setting as environment variable

To view the final USE setting as seen by Portage, run **emerge --info**. This will list all relevant variables (including the USE variable) with their current definition as known to Portage.

``` shell 
root # emerge --info
```

### Adapting the entire system to the new USE flags
After having altered USE flags, the system should be updated to reflect the necessary changes. To do so, use the `--newuse` option with **emerge**:

```shell 
root # emerge --update --deep --newuse @world
```

Next, run Portage's depclean to remove the conditional dependencies that were emerged on the "old" system but that have been obsoleted by the new USE flags.

!!! info title="Important"

    Double-check the provided list of "obsoleted" packages to make sure it does not remove packages that are needed. In the following example the --pretend (-p) switch to have depclean only list the packages without removing them:
    root # emerge --pretend --depclean


When depclean has finished, **emerge** may prompt to rebuild the applications that are dynamically linked against shared objects provided by possibly removed packages. Portage will preserve necessary libraries until this action is done to prevent breaking applications. It stores what needs to be rebuilt in the preserved-rebuild set. To rebuild the necessary packages, run:

```sh
root # emerge @preserved-rebuild
```

When all this is accomplished, the system is using the new USE flag settings.


## Package specific USE flags

### Viewing available USE flags
Let's take the example of seamonkey: what USE flags does it listen to? To find out, we use **emerge** with the `--pretend` and `--verbose` options:

```sh
root # emerge --pretend --verbose www-client/seamonkey
These are the packages that would be merged, in order:
 
Calculating dependencies... done!
[ebuild  N     ] www-client/seamonkey-2.48_beta1::gentoo  USE="calendar chatzilla crypt dbus gmp-autoupdate ipc jemalloc pulseaudio roaming skia startup-notification -custom-cflags -custom-optimization -debug -gtk3 -jack -minimal (-neon) (-selinux) (-system-cairo) -system-harfbuzz -system-icu -system-jpeg -system-libevent -system-libvpx -system-sqlite {-test} -wifi" L10N="-ca -cs -de -en-GB -es-AR -es-ES -fi -fr -gl -hu -it -ja -lt -nb -nl -pl -pt-PT -ru -sk -sv -tr -uk -zh-CN -zh-TW" 216,860 KiB
 
Total: 1 package (1 new), Size of downloads: 216,860 KiB
```

**emerge** isn't the only tool for this job. In fact, there is a tool dedicated to package information called **equery** which resides in the [app-portage/gentoolkit](https://packages.gentoo.org/packages/app-portage/gentoolkit) package

```sh
root # emerge --ask app-portage/gentoolkit
```

Now run **equery** with the uses argument to view the USE flags of a certain package. For instance, for the [app-portage/portage-utils](https://packages.gentoo.org/packages/app-portage/portage-utils) package:

```sh
user $ equery --nocolor uses =app-portage/portage-utils-0.93.3
[ Legend : U - final flag setting for installation]
[        : I - package is installed with flag     ]
[ Colors : set, unset                             ]
 * Found these USE flags for app-portage/portage-utils-0.93.3:
 U I
 + + nls       : Add Native Language Support (using gettext - GNU locale utilities)
 + + openmp    : Build support for the OpenMP (support parallel computing), requires >=sys-devel/gcc-4.2 built with USE="openmp"
 + + qmanifest : Build qmanifest applet, this adds additional dependencies for GPG, OpenSSL and BLAKE2B hashing
 + + qtegrity  : Build qtegrity applet, this adds additional dependencies for OpenSSL
 - - static    : !!do not set this during bootstrap!! Causes binaries to be statically linked instead of dynamically
```

## Satisfying REQUIRED_USE conditions

Some ebuilds require or forbid certain combinations of USE flags in order to work properly. This is expressed via a set of conditions placed in a *REQUIRED_USE* expression. This conditions ensure that all features and dependencies are complete and that the build will succeed and perform as expected. If any of these are not met, emerge will alert you and ask you to fix the issue.

|Example	|Description|
|---------|-----------|
|REQUIRED_USE=`"foo? ( bar )"`	|If foo is set, bar must be set.|
|REQUIRED_USE=`"foo? ( !bar )"`	|If foo is set, bar must not be set.|
|REQUIRED_USE=`"foo? ( || ( bar baz ) )"`	|If foo is set, bar or baz must be set.|
|REQUIRED_USE=`"^^ ( foo bar baz )"`	|Exactly one of foo bar or baz must be set.|
|REQUIRED_USE=`"|| ( foo bar baz )"`	|At least one of foo bar or baz must be set.|
|REQUIRED_USE=`"?? ( foo bar baz )"`	|No more than one of foo bar or baz may be set.|
|--|--|