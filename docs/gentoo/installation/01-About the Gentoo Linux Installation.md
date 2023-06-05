# About the Gentoo Linux Installation

## Introduction

### Welcome

Welcome to Gentoo! Gentoo is a free operating system based on Linux that can be automatically optimized and customized for just about any application or need. It is built on an ecosystem of free software and does not hide what is running beneath the hood from its users.

### Openness
Gentoo's premier tools are built from simple programming languages. [Portage](https://wiki.gentoo.org/wiki/Portage), Gentoo's package maintenance system, is [written in Python](https://gitweb.gentoo.org/proj/portage.git/). Ebuilds, which provide package definitions for Portage are [written in bash](https://gitweb.gentoo.org/repo/gentoo.git). Our users are encouraged to review, modify, and enhance the source code for all parts of Gentoo.

By default, packages are only patched when necessary to fix bugs or provide interoperability within Gentoo. They are installed to the system by compiling source code provided by upstream projects into binary format (although support for precompiled binary packages is included too). Configuring Gentoo happens through text files.

For the above reasons and others: *openness* is built-in as a *design principal*.

### Choice

*Choice* is another Gentoo *design principal*.

When installing Gentoo, choice is made clear throughout the Handbook. System administrators can choose two fully supported init systems (Gentoo's own [OpenRC](https://wiki.gentoo.org/wiki/OpenRC) and Freedesktop.org's [systemd](https://wiki.gentoo.org/wiki/Systemd)), partition structure for storage disk(s), what file systems to use on the disk(s), a target [system profile](https://wiki.gentoo.org/wiki/Profile), remove or add features on a global (system-wide) or package specific level via USE flags, bootloader, network management utility, and much, much more.

As a development philosophy, [Gentoo's authors](https://www.gentoo.org/inside-gentoo/developers/) try to avoid forcing users onto a specific system profile or desktop environment. If something is offered in the GNU/Linux ecosystem, it's likely available in Gentoo. If not, then we'd love to see it so. For new package requests please file a [bug report](https://bugs.gentoo.org/) or create your own ebuild repository.

### Power

Being a source-based operating system allows Gentoo to be ported onto new computer [instruction set architectures](https://en.wikipedia.org/wiki/instruction_set_architecture) and also allows all installed packages to be tuned. This strength surfaces another Gentoo *design principal: power.*

A system administrator who has successfully installed and customized Gentoo has compiled a tailored operating system from source code. The entire operating system can be tuned at a binary level via the mechanisms included in Portage's make.conf file. If so desired, adjustments can be made on a per-package basis, or a package group basis. In fact, entire sets of functionality can be added or removed using USE flags.

It is very important that the Handbook reader understands that these design principals are what makes Gentoo unique. With the principals of great power, many choices, and extreme openness highlighted, diligence, thought, and intentionality should be employed while using Gentoo.

### How the installation is structured

The Gentoo Installation can be seen as a 10-step procedure, corresponding to the next set of chapters. Each step results in a certain state:

|Step	|Result|
|1	|The user is in a working environment ready to install Gentoo.|
|2	|The Internet connection is ready to install Gentoo.|
|3	|The hard disks are initialized to host the Gentoo installation.|
|4	|The installation environment is prepared and the user is ready to chroot into the new environment.|
|5	|Core packages, which are the same on all Gentoo installations, are installed.|
|6	|The Linux kernel is installed.|
|7	|Most of the Gentoo system configuration files are created.|
|8	|The necessary system tools are installed.|
|9	|The proper boot loader has been installed and configured.|
|10	|The freshly installed Gentoo Linux environment is ready to be explored.|

Whenever a certain choice is presented the handbook will try to explain the pros and cons of each choice. Although the text then continues with a default choice (identified by "Default: " in the title), the other possibilities will be documented as well (marked by "Alternative: " in the title). Do not think that the default is what Gentoo recommends. It is, however, the choice that Gentoo believes most users will make.

Sometimes an optional step can be followed. Such steps are marked as "Optional: " and are therefore not needed to install Gentoo. However, some optional steps are dependent on a previously made decision. The instructions will inform the reader when this happens, both when the decision is made, and right before the optional step is described.

### Installation options for Gentoo
Gentoo can be installed in many different ways. It can be downloaded and installed from official Gentoo installation media such as our bootable ISO images. The installation media can be installed on a USB stick or accessed via a netbooted environment. Alternatively, Gentoo can be installed from non-official media such as an already installed distribution or a non-Gentoo bootable disk (such as [Knoppix](https://www.knopper.net/knoppix/index-en.html)).

This document covers the installation using official Gentoo Installation media or, in certain cases, netbooting.

!!! Note
```For help on the other installation approaches, including using non-Gentoo bootable media, please read our [Alternative installation guide](https://wiki.gentoo.org/wiki/Installation_alternatives).
```

We also provide a Gentoo installation tips and tricks document that might be useful.

Troubles
If a problem is found in the installation (or in the installation documentation), please visit our bug tracking system and check if the bug is known. If not, please create a bug report for it so we can take care of it. Do not be afraid of the developers who are assigned to the bugs - they (generally) don't eat people.

Although this document is architecture-specific, it may contain references to other architectures as well, because large parts of the Gentoo Handbook use text that is identical for all architectures (to avoid duplication of effort). Such references have been kept to a minimum, to avoid confusion.

If there is some uncertainty whether or not the problem is a user-problem (some error made despite having read the documentation carefully) or a software-problem (some error we made despite having tested the installation/documentation carefully) everybody is welcome to join the #gentoo (webchat) channel on irc.libera.chat. Of course, everyone is welcome otherwise too as our chat channel covers the broad Gentoo spectrum.

Speaking of which, if there are any additional questions regarding Gentoo, check out the Frequently Asked Questions article. There are also FAQs on the Gentoo Forums.