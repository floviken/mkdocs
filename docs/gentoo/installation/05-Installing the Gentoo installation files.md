# Installing the Gentoo installation files

## Installing a stage tarball

### Setting the date and time

Before installing Gentoo the clock must be set correctly. Due to Gentoo's web based services using security certificates, it might not be possible to download the installation files if the system clock is too far skewed. Also, files saved with a date in the future may cause strange errors after the initial installation has completed if the clock is corrected later.

Verify the current date and time by running the **date** command:

``` sh
root #date
Mon Oct  3 13:16:22 PDT 2021
```

If the date/time displayed is more than few minutes off, it should be updated in accuracy using one of the methods below.

#### Automatic

Most readers will desire to have their system update the time automatically using a time server.

!!! Important
Some motherboards do not include a Real-Time Clock (RTC), which will keep relatively accurate time even while the system is powered off. It is very important for these systems to automatically sync the system clock with a time server at every system start and on a regular internal thereafter. This is just as important for systems that do include a RTC, but have a failed battery.

Official Gentoo live environments include the **chronyd** command (available through the [net-misc/chrony](https://packages.gentoo.org/packages/net-misc/chrony) package) and a configuration file pointing to ntp.org time servers. It can be used to automatically sync the system clock to UTC time using a time server. Using this method requires a working network configuration and may not be available on all architectures.

!!! Warning
Automatic time sync comes at a price. It will reveal the system's IP address and related network information to a time server (in the case of the example below ntp.org). Users with privacy concerns should be aware of this before setting the system clock using the below method.

`root # chronyd -q`

#### Manual

For systems that do not have access to a time server, the **date** command can also be used to set the system clock. It will use the following format as an argument: MMDDhhmmYYYY syntax (**M**onth, **D**ay, **h**our, **m**inute and **Y**ear).

UTC time is recommended for all Linux systems. A timezone will be defined later in the installation which will modify the clock to display local time.

For instance, to set the date to October 3rd, 13:16 in the year 2021, issue:

`root # date 100313162021`

### Choosing a stage tarball

!!! Note
Not every architecture has a multilib option. Many only run with native code. Multilib is most commonly applied to **amd64**.

#### Multilib (32 and 64-bit)

Choosing a base tarball for the system can save a considerable amount of time later on in the installation process, specifically when it is time to choose a [system profile](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Choosing_the_right_profile). The selection of a stage tarball will directly impact future system configuration and can save a headache or two later on down the line. The multilib tarball uses 64-bit libraries when possible, and only falls back to the 32-bit versions when necessary for compatibility. This is an excellent option for the majority of installations because it provides a great amount of flexibility for customization in the future. Those who desire their systems to be capable of easily switching profiles should download the multilib tarball option for their respective processor architecture.

Most users should not use the 'advanced' tarballs options; they are for specific software or hardware configurations.

#### No-multilib (pure 64-bit)

Selecting a no-multilib tarball to be the base of the system provides a complete 64-bit operating system environment. This effectively renders the ability to switch to multilib profiles improbable, although still technically possible.

!!! Warning
Readers who are just starting out with Gentoo should not choose a no-multilib tarball unless it is absolutely necessary. Migrating from a no-multilib to a multilib system requires an extremely well-working knowledge of Gentoo and the lower-level toolchain (it may even cause our [Toolchain developers](https://wiki.gentoo.org/wiki/Project:Toolchain) to shudder a little). It is not for the faint of heart and is beyond the scope of this guide.

#### OpenRC

[OpenRC](https://wiki.gentoo.org/wiki/OpenRC) is a dependency-based init system (responsible for starting up system services once the kernel has booted) that maintains compatibility with the system provided init program, normally located in /sbin/init. It is Gentoo's native and original init system, but is also deployed by a few other Linux distributions and BSD systems.

OpenRC does not function as a replacement for the /sbin/init file by default and is 100% compatible with Gentoo init scripts. This means a solution can be found to run the dozens of daemons in the Gentoo ebuild repository.

systemd
systemd is a modern SysV-style init and rc replacement for Linux systems. It is used as the primary init system by a majority of Linux distributions. systemd is fully supported in Gentoo and works for its intended purpose. If something seems lacking in the Handbook for a systemd install path, review the [systemd article](https://wiki.gentoo.org/wiki/Systemd) before asking for support.

!!! Note
It is technically possible to switch a running Gentoo installation from OpenRC to systemd and back. However, switching requires some effort and is outside the scope of this installation manual. Before downloading a stage tarball, decide whether OpenRC or systemd will be used as the target init system and download the relevant stage tarball.

### Downloading the stage tarball

Go to the Gentoo mount point where the root file system is mounted (most likely /mnt/gentoo):

`root # cd /mnt/gentoo`

### Graphical browsers

Those using environments with fully graphical web browsers will have no problem copying a stage file URL from the main website's [download section](https://www.gentoo.org/downloads/#other-arches). Simply select the appropriate tab, right click the link to the stage file, then Copy Link to copy the link to the clipboard, then paste the link to the wget utility on the command-line to download the stage tarball:

`root # wget <PASTED_STAGE_URL>` 

#### Command-line browsers

More traditional readers or 'old timer' Gentoo users, working exclusively from command-line may prefer using links (www-client/links), a non-graphical, menu-driven browser. To download a stage, surf to the Gentoo mirror list like so:

`root # links https://www.gentoo.org/downloads/mirrors/`

To use an HTTP proxy with links, pass on the URL with the -http-proxy option:

`root # links -http-proxy proxy.server.com:8080 https://www.gentoo.org/downloads/mirrors/`

Next to links there is also the [lynx](https://packages.gentoo.org/packages/www-client/lynx) (www-client/lynx) browser. Like **links** it is a non-graphical browser but it is not menu-driven.

`root # lynx https://www.gentoo.org/downloads/mirrors/`

If a proxy needs to be defined, export the *http_proxy* and/or *ftp_proxy* variables:

```sh
root # export http_proxy="http://proxy.server.com:port"
root # export ftp_proxy="http://proxy.server.com:port"
```

On the mirror list, select a mirror close by. Usually HTTP mirrors suffice, but other protocols are available as well. Move to the releases/amd64/autobuilds/ directory. There all available stage files are displayed (they might be stored within subdirectories named after the individual sub-architectures). Select one and press d to download.

After the stage file download completes, it is possible to verify the integrity and validate the contents of the stage tarball. Those interested should proceed to the next section.

Those not interested in verifying and validating the stage file can close the command-line browser by pressing `q` and can move directly to the Unpacking the stage tarball section.

### Verifying and validating

!!! Note
Most stages are now explicitly [suffixed](https://www.gentoo.org/news/2021/07/20/more-downloads.html) with the init system type (openrc or systemd), although some architectures may still be missing these for now.

Like with the minimal installation CDs, additional downloads to verify and validate the stage file are available. Although these steps may be skipped, these files are provided for users who care about the legitimacy of the file(s) they just downloaded.

A .CONTENTS file that contains a list of all files inside the stage tarball.
A .DIGESTS file that contains checksums of the stage file in different algorithms.
Use **openssl** and compare the output with the checksums provided by the .DIGESTS file.

For instance, to validate the SHA512 checksum:

`root # openssl dgst -r -sha512 stage3-amd64-<release>-<init>.tar.xz`

Another way is to use the **sha512sum** command:

`root # sha512sum stage3-amd64-<release>-<init>.tar.xz`

To validate the Whirlpool checksum:

`root # openssl dgst -r -whirlpool stage3-amd64-<release>-<init>.tar.xz`

Compare the output of these commands with the value registered in the .DIGESTS files. The values need to match, otherwise the downloaded file might be corrupt (or the digests file is).

Just like with the ISO file, it is also possible to verify the cryptographic signature of the tar.xz file using gpg to make sure the tarball has not been tampered with:

`root # gpg --verify stage3-amd64-<release>-<init>.tar.xz{.asc,}`

The fingerprints of the OpenPGP keys used for signing release media can be found on the [release media signatures page](https://www.gentoo.org/downloads/signatures/) of the Gentoo webserver.

## Unpacking the stage tarball

Now unpack the downloaded stage onto the system. Use the tar utility to proceed:

`root # tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner`

Verify the same options (`xpf` and `--xattrs-include='*.*'`) are used in the command. The x stands for extract, the p for preserve permissions and the `f` to denote that we want to extract a file (not standard input). `--xattrs-include='*.*'` is to include preservation of the the extended attributes in all namespaces stored in the archive. Finally, `--numeric-owner` is used to ensure that the user and group IDs of the files being extracted from the tarball will remain the same as Gentoo's release engineering team intended (even if adventurous users are not using official Gentoo live environments).

Now that the stage file is unpacked, proceed with [Configuring compile options](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Stage#Configuring_compile_options).

## Configuring compile options

### Introduction

To optimize the system, it is possible to set variables which impact the behavior of Portage, Gentoo's officially supported package manager. All those variables can be set as environment variables (using export) but setting via export is not permanent.

 Note
Technically variables can be exported via the shell's profile or rc files, however that is not best practice for basic system administration.
Portage reads in the make.conf file when it runs, which will change runtime behavior depending on the values saved in the file. make.conf can be considered the primary configuration file for Portage, so treat its content carefully.

 Tip
A commented listing of all possible variables can be found in /mnt/gentoo/usr/share/portage/config/make.conf.example. Additional documentation on make.conf can be found by running man 5 make.conf.

For a successful Gentoo installation only the variables that are mentioned below need to be set.
Fire up an editor (in this guide we use nano) to alter the optimization variables we will discuss hereafter.

root #nano -w /mnt/gentoo/etc/portage/make.conf
From the make.conf.example file it is obvious how the file should be structured: commented lines start with #, other lines define variables using the VARIABLE="value" syntax. Several of those variables are discussed in the next section.

CFLAGS and CXXFLAGS
The CFLAGS and CXXFLAGS variables define the optimization flags for GCC C and C++ compilers respectively. Although those are defined generally here, for maximum performance one would need to optimize these flags for each program separately. The reason for this is because every program is different. However, this is not manageable, hence the definition of these flags in the make.conf file.

In make.conf one should define the optimization flags that will make the system the most responsive generally. Don't place experimental settings in this variable; too much optimization can make programs misbehave (crash, or even worse, malfunction).

We will not explain all possible optimization options. To understand them all, read the GNU Online Manual(s) or the gcc info page (info gcc - only works on a working Linux system). The make.conf.example file itself also contains lots of examples and information; don't forget to read it too.

A first setting is the -march= or -mtune= flag, which specifies the name of the target architecture. Possible options are described in the make.conf.example file (as comments). A commonly used value is native as that tells the compiler to select the target architecture of the current system (the one users are installing Gentoo on).

A second one is the -O flag (that is a capital O, not a zero), which specifies the gcc optimization class flag. Possible classes are s (for size-optimized), 0 (zero - for no optimizations), 1, 2 or even 3 for more speed-optimization flags (every class has the same flags as the one before, plus some extras). -O2 is the recommended default. -O3 is known to cause problems when used system-wide, so we recommend to stick to -O2.

Another popular optimization flag is -pipe (use pipes rather than temporary files for communication between the various stages of compilation). It has no impact on the generated code, but uses more memory. On systems with low memory, gcc might get killed. In that case, do not use this flag.

Using -fomit-frame-pointer (which doesn't keep the frame pointer in a register for functions that don't need one) might have serious repercussions on the debugging of applications.

When the CFLAGS and CXXFLAGS variables are defined, combine the several optimization flags in one string. The default values contained in the stage3 archive that is unpacked should be good enough. The following one is just an example:

CODE Example CFLAGS and CXXFLAGS variables
# Compiler flags to set for all languages
COMMON_FLAGS="-march=native -O2 -pipe"
# Use the same settings for both variables
CFLAGS="${COMMON_FLAGS}"
CXXFLAGS="${COMMON_FLAGS}"
 Tip
Although the GCC optimization article has more information on how the various compilation options can affect a system, the Safe CFLAGS article may be a more practical place for beginners to start optimizing their systems.
MAKEOPTS
The MAKEOPTS variable defines how many parallel compilations should occur when installing a package. As of Portage version 3.0.31[1], if left undefined, Portage's default behavior is to set the MAKEOPTS value to the same number of threads returned by nproc.

A good choice is the smaller of: the number of threads the CPU has, or the total amount of system RAM divided by 2 GiB.

 Warning
Using a large number of jobs can significantly impact memory consumption. A good recommendation is to have at least 2 GiB of RAM for every job specified (so, e.g. -j6 requires at least 12 GiB). To avoid running out of memory, lower the number of jobs to fit the available memory.
 Tip
When using parallel emerges (--jobs), the effective number of jobs run can grow exponentially (up to make jobs multiplied by emerge jobs). This can be worked around by running a localhost-only distcc configuration that will limit the number of compiler instances per host.
FILE /etc/portage/make.confExample MAKEOPTS declaration
# If left undefined, Portage's default behavior is to set the MAKEOPTS value to the same number of threads returned by `nproc` 
MAKEOPTS="-j4"
Search for MAKEOPTS in man 5 make.conf for more details.

Ready, set, go!
Update the /mnt/gentoo/etc/portage/make.conf file to match personal preference and save (nano users would hit Ctrl+o to write the change and then Ctrl+x to quit).

Then continue withInstalling the Gentoo base system.