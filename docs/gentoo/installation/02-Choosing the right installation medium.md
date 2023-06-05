# Choosing the right installation medium

## Hardware requirements

Before we start, we first list what hardware requirements are needed to successfully install Gentoo on a amd64 box.


AMD64 livedisk hardware requirements
||Minimal CD	|LiveDVD|
|--|--|--|
|CPU|	|Any x86-64 CPU, both AMD64 and Intel 64|
|Memory|	|2 GB|
|Disk space|	8 GB (excluding swap space)|
|Swap space|	At least 2 GB|

The [AMD64 project](https://wiki.gentoo.org/wiki/Project:AMD64) is a good place to be for more information about Gentoo's **amd64** support.


## Gentoo Linux installation media
!!! Tip
```It is okay to use other, non-Gentoo installation media, although official media is recommended. Gentoo installation media ensures the necessary tools are around. When using non-Gentoo media, skip to Preparing the disks.
```

### Minimal installation CD

The Gentoo minimal installation CD is a bootable image: a self-contained Gentoo environment. It allows the user to boot Linux from the CD or other installation media. During the boot process the hardware is detected and the appropriate drivers are loaded. The image is maintained by Gentoo developers and allows anyone to install Gentoo if an active Internet connection is available.

The Minimal Installation CD is called install-amd64-minimal-<release>.iso.

### The occasional Gentoo LiveDVD

Occasionally, a special DVD image is crafted which can be used to install Gentoo. The instructions in this chapter target the Minimal Installation CD, so things might be a bit different when booting from the LiveDVD. However, the LiveDVD (or any other official Gentoo Linux environment) supports getting a root prompt by just invoking **sudo su -** or **sudo -i** in a terminal.

### What are stages then?

A stage3 tarball is an archive containing a profile specific minimal Gentoo environment. Stage3 tarballs are suitable to continue the Gentoo installation using the instructions in this handbook. Previously, the handbook described the installation using one of three [stage tarballs](https://wiki.gentoo.org/wiki/Stage_tarball). Gentoo does not offer stage1 and stage2 tarballs for download any more since these are mostly for internal use and for bootstrapping Gentoo on new architectures.

Stage3 tarballs can be downloaded from releases/amd64/autobuilds/ on any of the [official Gentoo mirrors](https://www.gentoo.org/downloads/mirrors/). Stage files update frequently and are not included in official installation images.

## Downloading

### Obtain the media

The default installation media that Gentoo Linux uses are the minimal installation CDs, which host a bootable, very small Gentoo Linux environment. This environment contains all the right tools to install Gentoo. The CD images themselves can be downloaded from the [downloads page](https://www.gentoo.org/downloads/) (recommended) or by manually browsing to the ISO location on one of the many [available mirrors](https://www.gentoo.org/downloads/mirrors/).

If downloading from a mirror, the minimal installation CDs can be found as follows:

1. Go to the releases/ directory.
2. Select the directory for the relevant target architecture (such as amd64/).
3. Select the autobuilds/ directory.
4. For amd64 and x86 architectures select either the current-install-amd64-minimal/ or current-install-x86-minimal/ directory (respectively). For all other architectures navigate to the current-iso/ directory.

!!! Note
```Some target architectures such as arm, mips, and s390 will not have minimal install CDs. At this time the Gentoo Release Engineering project does not support building .iso files for these targets.
```
Inside this location, the installation media file is the file with the .iso suffix. For instance, take a look at the following listing:

```sh title="CODE Example list of downloadable files at releases/amd64/autobuilds/current-iso/"
[DIR] hardened/                                          05-Dec-2014 01:42    -   
[   ] install-amd64-minimal-20141204.iso                 04-Dec-2014 21:04  208M  
[   ] install-amd64-minimal-20141204.iso.CONTENTS        04-Dec-2014 21:04  3.0K  
[   ] install-amd64-minimal-20141204.iso.DIGESTS         04-Dec-2014 21:04  740   
[TXT] install-amd64-minimal-20141204.iso.asc             05-Dec-2014 01:42  1.6K  
[   ] stage3-amd64-20141204.tar.bz2                      04-Dec-2014 21:04  198M  
[   ] stage3-amd64-20141204.tar.bz2.CONTENTS             04-Dec-2014 21:04  4.6M  
[   ] stage3-amd64-20141204.tar.bz2.DIGESTS              04-Dec-2014 21:04  720   
[TXT] stage3-amd64-20141204.tar.bz2.asc                  05-Dec-2014 01:42  1.5K
```

In the above example, the install-amd64-minimal-20141204.iso file is the minimal installation CD itself. But as can be seen, other related files exist as well:

- A .CONTENTS file which is a text file listing all files available on the installation media. This file can be useful to verify if particular firmware or drivers are available on the installation media before downloading it.
- A .DIGESTS file which contains the hash of the ISO file itself, in various hashing formats/algorithms. This file can be used to verify if the downloaded ISO file is corrupt or not.
- A .asc file which is a cryptographic signature of the ISO file. This can be used to both verify if the downloaded ISO file is corrupt or not, as well as verify that the download is indeed provided by the Gentoo Release Engineering team and has not been tampered with.

Ignore the other files available at this location for now - those will come back when the installation has proceeded further. Download the .iso file and, if verification of the download is wanted, download the .iso.asc file for the .iso file as well. The .CONTENTS file does not need to be downloaded as the installation instructions will not refer to this file anymore, and the .DIGESTS is not needed if the signature in the .iso.asc file is verified.

### Verifying the downloaded files

!!! Note
```This is an optional step and not necessary to install Gentoo Linux. However, it is recommended as it ensures that the downloaded file is not corrupt and has indeed been provided by the Gentoo Infrastructure team.```

The .asc file provides a cryptographic signature of the ISO. By validating it, one can make sure that the installation file is provided by the Gentoo Release Engineering team and is intact and unmodified.

### Microsoft Windows based verification
To first verify the cryptographic signature, tools such as [GPG4Win](https://www.gpg4win.org/) can be used. After installation, the public keys of the Gentoo Release Engineering team need to be imported. The list of keys is available on the [signatures page](https://www.gentoo.org/downloads/signatures/). Once imported, the user can then verify the signature in the .asc file.

### Linux based verification
On a Linux system, the most common method for verifying the cryptographic signature is to use the [app-crypt/gnupg](https://packages.gentoo.org/packages/app-crypt/gnupg) software. With this package installed, the following command can be used to verify the cryptographic signature in the .asc file.

First, download the right set of keys as made available on the [signatures page](https://www.gentoo.org/downloads/signatures/):

```sh 
user $ gpg --keyserver hkps://keys.gentoo.org --recv-keys 0xBB572E0E2D182910
gpg: requesting key 0xBB572E0E2D182910 from hkp server pool.sks-keyservers.net
gpg: key 0xBB572E0E2D182910: "Gentoo Linux Release Engineering (Automated Weekly Release Key) <releng@gentoo.org>" 1 new signature
gpg: 3 marginal(s) needed, 1 complete(s) needed, classic trust model
gpg: depth: 0  valid:   3  signed:  20  trust: 0-, 0q, 0n, 0m, 0f, 3u
gpg: depth: 1  valid:  20  signed:  12  trust: 9-, 0q, 0n, 9m, 2f, 0u
gpg: next trustdb check due at 2018-09-15
gpg: Total number processed: 1
gpg:         new signatures: 1
```
Alternatively you can use instead the [WKD](https://wiki.gentoo.org/wiki/WKD) to download the key:

```sh 
user $ gpg --auto-key-locate=clear,nodefault,wkd --locate-key releng@gentoo.org
gpg: key 0x9E6438C817072058: public key "Gentoo Linux Release Engineering (Gentoo Linux Release Signing Key) <releng@gentoo.org>" imported
gpg: key 0xBB572E0E2D182910: public key "Gentoo Linux Release Engineering (Automated Weekly Release Key) <releng@gentoo.org>" imported
gpg: Total number processed: 2
gpg:               imported: 2
gpg: public key of ultimately trusted key 0x58497EE51D5D74A5 not found
gpg: public key of ultimately trusted key 0x1F3D03348DB1A3E2 not found
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
pub   dsa1024/0x9E6438C817072058 2004-07-20 [SC] [expires: 2024-01-01]
      D99EAC7379A850BCE47DA5F29E6438C817072058
uid                   [ unknown] Gentoo Linux Release Engineering (Gentoo Linux Release Signing Key) <releng@gentoo.org>
sub   elg2048/0x0403710E1415B4ED 2004-07-20 [E] [expires: 2024-01-01]
```

Or if using official Gentoo release media, import the key from /usr/share/openpgp-keys/gentoo-release.asc (provided by [sec-keys/openpgp-keys-gentoo-release)](https://packages.gentoo.org/packages/sec-keys/openpgp-keys-gentoo-release):

``` sh
user $ gpg --import /usr/share/openpgp-keys/gentoo-release.asc
gpg: directory '/home/larry/.gnupg' created
gpg: keybox '/home/larry/.gnupg/pubring.kbx' created
gpg: key DB6B8C1F96D8BF6D: 2 signatures not checked due to missing keys
gpg: /home/larry/.gnupg/trustdb.gpg: trustdb created
gpg: key DB6B8C1F96D8BF6D: public key "Gentoo ebuild repository signing key (Automated Signing Key) <infrastructure@gentoo.org>" imported
gpg: key 9E6438C817072058: 3 signatures not checked due to missing keys
gpg: key 9E6438C817072058: public key "Gentoo Linux Release Engineering (Gentoo Linux Release Signing Key) <releng@gentoo.org>" imported
gpg: key BB572E0E2D182910: 1 signature not checked due to a missing key
gpg: key BB572E0E2D182910: public key "Gentoo Linux Release Engineering (Automated Weekly Release Key) <releng@gentoo.org>" imported
gpg: key A13D0EF1914E7A72: 1 signature not checked due to a missing key
gpg: key A13D0EF1914E7A72: public key "Gentoo repository mirrors (automated git signing key) <repomirrorci@gentoo.org>" imported
gpg: Total number processed: 4
gpg:               imported: 4
gpg: no ultimately trusted keys found
``` 

Next verify the cryptographic signature:

```
user $ gpg --verify install-amd64-minimal-20141204.iso.asc
gpg: Signature made Fri 05 Dec 2014 02:42:44 AM CET
gpg:                using RSA key 0xBB572E0E2D182910
gpg: Good signature from "Gentoo Linux Release Engineering (Automated Weekly Release Key) <releng@gentoo.org>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
```

Primary key fingerprint: 13EB BDBE DE7A 1277 5DFD  B1BA BB57 2E0E 2D18 2910
To be absolutely certain that everything is valid, verify the fingerprint shown with the fingerprint on the Gentoo signatures page.

Burning a disk
Of course, with just an ISO file downloaded, the Gentoo Linux installation cannot be started. The ISO file needs to be burned on a CD to boot from, and in such a way that its content is burned on the CD, not just the file itself. Below a few common methods are described - a more elaborate set of instructions can be found in Our FAQ on burning an ISO file.

Burning with Microsoft Windows 7 and above
Versions of Microsoft Windows 7 and above can both mount and burn ISO images to optical media without the requirement for third-party software. Simply insert a burnable disk, browse to the downloaded ISO files, right click the file in Windows Explorer, and select "Burn disk image".

Burning with Linux
The cdrecord utility from the package app-cdr/cdrtools can burn ISO images on Linux.

To burn the ISO file on the CD in the /dev/sr0 device (this is the first CD device on the system - substitute with the right device file if necessary):

user $cdrecord dev=/dev/sr0 install-amd64-minimal-20141204.iso
Users that prefer a graphical user interface can use K3B, part of the kde-apps/k3b package. In K3B, go to Tools and use Burn CD Image.

Booting
Booting the installation media
Once the installation media is ready, it is time to boot it. Insert the media in the system, reboot, and enter the motherboard's firmware user interface. This is usually performed by pressing a keyboard key such as DEL, F1, F10, or ESC during the Power-On Self-test (POST) process. The 'trigger' key varies depending on the system and motherboard. If it is not obvious use an internet search engine and do some research using the motherboard's model name as the search keyword. Results should be easy to determine. Once inside the motherboard's firmware menu, change the boot order so that the external bootable media (CD/DVD disks or USB drives) are tried before the internal disk devices. Without this change, the system will most likely reboot to the internal disk device, ignoring the external boot media.

 Important
When installing Gentoo with the purpose of using the UEFI interface instead of BIOS, it is recommended to boot with UEFI immediately. If not, then it might be necessary to create a bootable UEFI USB stick (or other medium) once before finalizing the Gentoo Linux installation.
If not yet done, ensure that the installation media is inserted or plugged into the system, and reboot. A boot prompt should be shown. At this screen, Enter will begin the boot process with the default boot options. To boot the installation media with custom boot options, specify a kernel followed by boot options and then hit Enter.

 Note
In all likelihood, the default gentoo kernel, as mentioned above, without specifying any of the optional parameters will work just fine. For boot troubleshooting and expert options, continue on with this section. Otherwise, just press Enter and skip ahead to Extra hardware configuration.
At the boot prompt, users get the option of displaying the available kernels (F1) and boot options (F2). If no choice is made within 15 seconds (either displaying information or using a kernel) then the installation media will fall back to booting from disk. This allows installations to reboot and try out their installed environment without the need to remove the CD from the tray (something well appreciated for remote installations).

Specifying a kernel was mentioned. On the Minimal installation media, only two predefined kernel boot options are provided. The default option is called gentoo. The other being the -nofb variant; this disables kernel framebuffer support.

The next section displays a short overview of the available kernels and their descriptions:

Kernel choices
gentoo
Default kernel with support for K8 CPUs (including NUMA support) and EM64T CPUs.
gentoo-nofb
Same as gentoo but without framebuffer support.
memtest86
Test the local RAM for errors.
Alongside the kernel, boot options help in tuning the boot process further.

Hardware options
acpi=on
This loads support for ACPI and also causes the acpid daemon to be started by the CD on boot. This is only needed if the system requires ACPI to function properly. This is not required for Hyperthreading support.
acpi=off
Completely disables ACPI. This is useful on some older systems and is also a requirement for using APM. This will disable any Hyperthreading support of your processor.
console=X
This sets up serial console access for the CD. The first option is the device, usually ttyS0, followed by any connection options, which are comma separated. The default options are 9600,8,n,1.
dmraid=X
This allows for passing options to the device-mapper RAID subsystem. Options should be encapsulated in quotes.
doapm
This loads APM driver support. This also requires that acpi=off.
dopcmcia
This loads support for PCMCIA and Cardbus hardware and also causes the pcmcia cardmgr to be started by the CD on boot. This is only required when booting from PCMCIA/Cardbus devices.
doscsi
This loads support for most SCSI controllers. This is also a requirement for booting most USB devices, as they use the SCSI subsystem of the kernel.
sda=stroke
This allows the user to partition the whole hard disk even when the BIOS is unable to handle large disks. This option is only used on machines with an older BIOS. Replace sda with the device that requires this option.
ide=nodma
This forces the disabling of DMA in the kernel and is required by some IDE chipsets and also by some CDROM drives. If the system is having trouble reading from the IDE CDROM, try this option. This also disables the default hdparm settings from being executed.
noapic
This disables the Advanced Programmable Interrupt Controller that is present on newer motherboards. It has been known to cause some problems on older hardware.
nodetect
This disables all of the autodetection done by the CD, including device autodetection and DHCP probing. This is useful for debugging a failing CD or driver.
nodhcp
This disables DHCP probing on detected network cards. This is useful on networks with only static addresses.
nodmraid
Disables support for device-mapper RAID, such as that used for on-board IDE/SATA RAID controllers.
nofirewire
This disables the loading of Firewire modules. This should only be necessary if your Firewire hardware is causing a problem with booting the CD.
nogpm
This disables gpm console mouse support.
nohotplug
This disables the loading of the hotplug and coldplug init scripts at boot. This is useful for debugging a failing CD or driver.
nokeymap
This disables the keymap selection used to select non-US keyboard layouts.
nolapic
This disables the local APIC on Uniprocessor kernels.
nosata
This disables the loading of Serial ATA modules. This is used if the system is having problems with the SATA subsystem.
nosmp
This disables SMP, or Symmetric Multiprocessing, on SMP-enabled kernels. This is useful for debugging SMP-related issues with certain drivers and motherboards.
nosound
This disables sound support and volume setting. This is useful for systems where sound support causes problems.
nousb
This disables the autoloading of USB modules. This is useful for debugging USB issues.
slowusb
This adds some extra pauses into the boot process for slow USB CDROMs, like in the IBM BladeCenter.
Logical volume/device management
dolvm
This enables support for Linux's Logical Volume Management.
Other options
debug
Enables debugging code. This might get messy, as it displays a lot of data to the screen.
docache
This caches the entire runtime portion of the CD into RAM, which allows the user to umount /mnt/cdrom and mount another CDROM. This option requires that there is at least twice as much available RAM as the size of the CD.
doload=X
This causes the initial ramdisk to load any module listed, as well as dependencies. Replace X with the module name. Multiple modules can be specified by a comma-separated list.
dosshd
Starts sshd on boot, which is useful for unattended installs.
passwd=foo
Sets whatever follows the equals as the root password, which is required for dosshd since the root password is by default scrambled.
noload=X
This causes the initial ramdisk to skip the loading of a specific module that may be causing a problem. Syntax matches that of doload.
nonfs
Disables the starting of portmap/nfsmount on boot.
nox
This causes an X-enabled LiveCD to not automatically start X, but rather, to drop to the command line instead.
scandelay
This causes the CD to pause for 10 seconds during certain portions the boot process to allow for devices that are slow to initialize to be ready for use.
scandelay=X
This allows the user to specify a given delay, in seconds, to be added to certain portions of the boot process to allow for devices that are slow to initialize to be ready for use. Replace X with the number of seconds to pause.
 Note
The bootable media will check for no* options before do* options, so that options can be overridden in the exact order specified.
Now boot the media, select a kernel (if the default gentoo kernel does not suffice) and boot options. As an example, we boot the gentoo kernel, with dopcmcia as a kernel parameter:

boot:gentoo dopcmcia
Next the user will be greeted with a boot screen and progress bar. If the installation is done on a system with a non-US keyboard, make sure to immediately press Alt+F1 to switch to verbose mode and follow the prompt. If no selection is made in 10 seconds the default (US keyboard) will be accepted and the boot process will continue. Once the boot process completes, the user is automatically logged in to the "Live" Gentoo Linux environment as the root user, the super user. A root prompt is displayed on the current console, and one can switch to other consoles by pressing Alt+F2, Alt+F3 and Alt+F4. Get back to the one started on by pressing Alt+F1.


Extra hardware configuration
When the Installation medium boots, it tries to detect all the hardware devices and loads the appropriate kernel modules to support the hardware. In the vast majority of cases, it does a very good job. However, in some cases it may not auto-load the kernel modules needed by the system. If the PCI auto-detection missed some of the system's hardware, the appropriate kernel modules have to be loaded manually.

In the next example the 8139too module (which supports certain kinds of network interfaces) is loaded:

root #modprobe 8139too
Optional: User accounts
If other people need access to the installation environment, or there is need to run commands as a non-root user on the installation medium (such as to chat using irssi without root privileges for security reasons), then an additional user account needs to be created and the root password set to a strong password.

To change the root password, use the passwd utility:

root #passwd
New password: (Enter the new password)
Re-enter password: (Re-enter the password)
To create a user account, first enter their credentials, followed by the account's password. The useradd and passwd commands are used for these tasks.

In the next example, a user called john is created:

root #useradd -m -G users john
root #passwd john
New password: (Enter john's password)
Re-enter password: (Re-enter john's password)
To switch from the (current) root user to the newly created user account, use the su command:

root #su - john
Optional: Viewing documentation while installing
TTYs
To view the Gentoo handbook during the installation, first create a user account as described above. Then press Alt+F2 to go to a new terminal (TTY).

During the installation, the links command can be used to browse the Gentoo handbook - of course only from the moment that the Internet connection is working.

user $links https://wiki.gentoo.org/wiki/Handbook:AMD64
To go back to the original terminal, press Alt+F1.

 Tip
When booted to the Gentoo minimal or Gentoo admin environments, seven TTYs will be available. They can be switched by pressing Alt then a function key between F1-F7. It can be useful to switch to a new terminal when waiting for job to complete, to open documentation, etc.
GNU Screen
The Screen utility is installed by default on official Gentoo installation media. It may be more efficient for the seasoned Linux enthusiast to use screen to view installation instructions via split panes rather than the multiple TTY method mentioned above.

Optional: Starting the SSH daemon
To allow other users to access the system during the installation (perhaps to support during an installation, or even do it remotely), a user account needs to be created (as was documented earlier on) and the SSH daemon needs to be started.

To fire up the SSH daemon on an OpenRC init, execute the following command:

root #rc-service sshd start
 Note
If users log on to the system, they will see a message that the host key for this system needs to be confirmed (through what is called a fingerprint). This behavior is typical and can be expected for initial connections to an SSH server. However, later when the system is set up and someone logs on to the newly created system, the SSH client will warn that the host key has been changed. This is because the user now logs on to - for SSH - a different server (namely the freshly installed Gentoo system rather than the live environment that the installation is currently using). Follow the instructions given on the screen then to replace the host key on the client system.
To be able to use sshd, the network needs to function properly. Continue with the chapter on Configuring the network.