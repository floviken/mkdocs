# Configuring the Linux kernel

## Optional: Installing firmware and/or microcode

### Firmware

Before getting to configuring kernel sections, it is beneficial to be aware that some hardware devices require additional, sometimes non-FOSS compliant, firmware to be installed on the system before they will operate correctly. This is often the case for wireless network interfaces commonly found in both desktop and laptop computers. Modern video chips from vendors like AMD, Nvidia, and Intel, often also require external firmware files to be fully functional. Most firmware for modern hardware devices can be found within the [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware) package.

It is recommended to have the [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware) package installed before the initial system reboot in order to have the firmware available in the event that it is necessary:

`root # emerge --ask sys-kernel/linux-firmware`

!!! Note
Installing certain firmware packages often requires accepting the associated firmware licenses. If necessary, visit the license handling section of the Handbook for help on accepting licenses.

It is important to note that kernel symbols that are built as modules (M) will load their associated firmware files from the filesystem when they are loaded by the kernel. It is not necessary to include the device's firmware files into the kernel's binary image for symbols loaded as modules.

### Microcode

In addition to discrete graphics hardware and network interfaces, CPUs also can require firmware updates. Typically this kind of firmware is referred to as microcode. Newer revisions of microcode are sometimes necessary to patch instability, security concerns, or other miscellaneous bugs in CPU hardware.

Microcode updates for AMD CPUs are distributed within the aforementioned [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware) package. Microcode for Intel CPUs can be found within the [sys-firmware/intel-microcode](https://packages.gentoo.org/packages/sys-firmware/intel-microcode) package, which will need to be installed separately. See the [Microcode article](https://wiki.gentoo.org/wiki/Microcode) for more information on how to apply microcode updates.

## Kernel configuration and compilation

Now it is time to configure and compile the kernel sources. For the purposes of the installation, three approaches to kernel management will be presented, however at any point post-installation a new approach can be employed.

Ranked from least involved to most involved:

[Full automation approach: Distribution kernels](#distribution-kernels)

A [Distribution Kernel](https://wiki.gentoo.org/wiki/Project:Distribution_Kernel) is used to configure, automatically build, and install the Linux kernel, its associated modules, and (optionally, but enabled by default) an initramfs file. Future kernel updates are fully automated since they are handled through the package manager, just like any other system package. It is possible provide [a custom kernel configuration file](https://wiki.gentoo.org/wiki/Project:Distribution_Kernel) if customization is necessary. This is the least involved process and is perfect for new Gentoo users due to it working out-of-the-box and offering minimal involvement from the system administrator.

[Hybrid approach: Genkernel](#alternative-genkernel)

New kernel sources are installed via the system package manager. System administrators use Gentoo's **genkernel** tool to generically configure, automatically build and install the Linux kernel, its associated modules, and (optionally, but **not** enabled by default) an initramfs file. It is possible provide a custom kernel configuration file if customization is necessary. Future kernel configuration, compilation, and installation require the system administrator's involvement in the form of running **eselect kernel, genkernel**, and potentially other commands for each update.

[Full manual approach](#alternative-manual-configuration)

New kernel sources are installed via the system package manager. The kernel is manually configured, built, and installed using the **eselect kernel** and a slew of **make** commands. Future kernel updates repeat the manual process of configuring, building, and installing the kernel files. This is the most involved process, but offers maximum control over the kernel update process.
The core around which all distributions are built is the Linux kernel. It is the layer between the user's programs and the system hardware. Although the handbook provides its users several possible kernel sources, a more comprehensive listing with more detailed descriptions is available at the [Kernel overview page](https://wiki.gentoo.org/wiki/Kernel/Overview).

### Distribution kernels

[Distribution Kernels](https://wiki.gentoo.org/wiki/Project:Distribution_Kernel) are ebuilds that cover the complete process of unpacking, configuring, compiling, and installing the kernel. The primary advantage of this method is that the kernels are updated to new versions by the package manager as part of @world upgrade. This requires no more involvement than running an emerge command. Distribution kernels default to a configuration supporting the majority of hardware, however two mechanisms are offered for customization: savedconfig and config snippets. See the project page for [more details on configuration](https://wiki.gentoo.org/wiki/Project:Distribution_Kernel#Modifying_kernel_configuration).

#### Installing the correct installkernel package

Before using the distribution kernels, please verify that the correct installkernel package for the system has been installed. When using [systemd-boot](https://wiki.gentoo.org/wiki/Systemd-boot) (formerly gummiboot) as the bootloader, install:

`root # emerge --ask sys-kernel/installkernel-systemd-boot`

When using a traditional a /boot layout (e.g. GRUB, LILO, etc.), the *gentoo* variant should be installed by default. If in doubt:

`root # emerge --ask sys-kernel/installkernel-gentoo`

#### Installing a distribution kernel

To build a kernel with Gentoo patches from source, type:

`root # emerge --ask sys-kernel/gentoo-kernel`

System administrators who want to avoid compiling the kernel sources locally can instead use precompiled kernel images:

`root # emerge --ask sys-kernel/gentoo-kernel-bin`

#### Upgrading and cleaning up

Once the kernel is installed, the package manager will automatically update it to newer versions. The previous versions will be kept until the package manager is requested to clean up stale packages. To reclaim disk space, stale packages can be trimmed by periodically running emerge with the --depclean option:

`root # emerge --depclean`

Alternatively, to specifically clean up old kernel versions:

`root # emerge --prune sys-kernel/gentoo-kernel sys-kernel/gentoo-kernel-bin`

#### Post-install/upgrade tasks

Distribution kernels are capable of rebuilding kernel modules installed by other packages. linux-mod.eclass provides the `dist-kernel` USE flag which controls a subslot dependency on [virtual/dist-kernel](https://packages.gentoo.org/packages/virtual/dist-kernel).

Enabling this USE flag on packages like [sys-fs/zfs](https://packages.gentoo.org/packages/sys-fs/zfs) and [sys-fs/zfs-kmod](https://packages.gentoo.org/packages/sys-fs/zfs-kmod) allows them to automatically be rebuilt against a newly updated kernel and, if applicable, will re-generate the initramfs accordingly.

##### Manually rebuilding the initramfs

If required, manually trigger such rebuilds by, after a kernel upgrade, executing:

`root # emerge --ask @module-rebuild`

If any kernel modules (e.g. ZFS) are needed at early boot, rebuild the initramfs afterward via:

`root # emerge --config sys-kernel/gentoo-kernel`
`root # emerge --config sys-kernel/gentoo-kernel-bin`

### Installing the kernel sources

!!! Note
This section is only relevant when using the following genkernel (hybrid) or manual kernel management approach.
When installing and compiling the kernel for amd64-based systems, Gentoo recommends the sys-kernel/gentoo-sources package.

Choose an appropriate kernel source and install it using **emerge**:

`root # emerge --ask sys-kernel/gentoo-sources`

This will install the Linux kernel sources in /usr/src/ using the specific kernel version in the path. It will not create a symbolic link by itself without `USE=symlink` being enabled on the chosen kernel sources package.

It is conventional for a /usr/src/linux symlink to be maintained, such that it refers to whichever sources correspond with the currently running kernel. However, this symbolic link will not be created by default. An easy way to create the symbolic link is to utilize eselect's kernel module.

For further information regarding the purpose of the symlink, and how to manage it, please refer to [Kernel/Upgrade](https://wiki.gentoo.org/wiki/Kernel/Upgrade).

First, list all installed kernels:

```sh 
root # eselect kernel list
Available kernel symlink targets:
  [1]   linux-5.15.52-gentoo
```

In order to create a symbolic link called linux, use:

```sh 
root $ eselect kernel set 1
root $ ls -l /usr/src/linux
lrwxrwxrwx    1 root   root    12 Oct 13 11:04 /usr/src/linux -> linux-5.15.52-gentoo
```

### Alternative: Genkernel

If an entirely manual configuration looks too daunting, system administrators should consider using [genkernel](https://wiki.gentoo.org/wiki/Genkernel) as a hybrid approach to kernel maintenance.

Genkernel provides a generic kernel configuration file, automatically **gen**erates the **kernel**, initramfs, and associated modules, and then installs the resulting binaries to the appropriate locations. This results in minimal and generic hardware support for the system's first boot, and allows for additional update control and customization of the kernel's configuration in the future.

Be informed: while using **genkernel** to maintain the kernel provides system administrators with more update control over the system's kernel, initramfs, and other options, it will require a time and effort commitment to perform future kernel updates as new sources are released. Those looking for a hands-off approach to kernel maintenance should [use a distribution kernel](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Kernel#Alternative:_Using_distribution_kernels).

For additional clarity, it is a *misconception* to believe genkernel automatically generates a custom kernel configuration for the hardware on which it is run; it uses a predetermined kernel configuration that supports most generic hardware and automatically handles the **make** commands necessary to assemble and install the kernel, the associate modules, and the initramfs file.

#### Binary redistributable software license group

If the linux-firmware package has been previously installed, then skip onward to the to the [installation section](#installation).

As a prerequisite, due to the `firwmare` USE flag being enabled by default for the [sys-kernel/genkernel](https://packages.gentoo.org/packages/sys-kernel/genkernel) package, the package manager will also attempt to pull in the [sys-kernel/linux-firmware](https://packages.gentoo.org/packages/sys-kernel/linux-firmware) package. The binary redistributable software licenses are required to be accepted before the linux-firmware will install.

This license group can be accepted system-wide for any package by adding the `@BINARY-REDISTRIBUTABLE` as an *ACCEPT_LICENSE* value in the /etc/portage/make.conf file. It can be exclusively accepted for the linux-firmware package by adding a specific inclusion via a /etc/portage/package.license/linux-firmware file.

If necessary, review the [methods of accepting software licenses](https://wiki.gentoo.org/wiki/Handbook:AMD64/Installation/Base#Optional:_Configure_the_ACCEPT_LICENSE_variable) available in the Installing the base system chapter of the handbook, then make some changes for acceptable software licenses.

If in analysis paralysis, the following will do the trick:

`root # mkdir /etc/portage/package.license`

```sh title="FILE /etc/portage/package.license/linux-firmware Accept binary redistributable licenses for the linux-firmware package"
sys-kernel/linux-firmware @BINARY-REDISTRIBUTABLE
```

#### Installation

Explanations and prerequisites aside, install the sys-kernel/genkernel package:

`root # emerge --ask sys-kernel/genkernel`

#### Generation

Compile the kernel sources by running **genkernel all**. Be aware though, as **genkernel** compiles a kernel that supports a wide array of hardware for differing computer architectures, this compilation may take quite a while to finish.

!!! Note
If the root partition/volume uses a filesystem other than ext4, it may be necessary to manually configure the kernel using genkernel --menuconfig all to add built-in kernel support for the particular filesystem(s) (i.e. not building the filesystem as a module).

!!! Note
Users of LVM2 should add --lvm as an argument to the genkernel command below.


`root # genkernel --mountboot --install all`

Once genkernel completes, a kernel and an initial ram filesystem (initramfs) will be generated and installed into the /boot directory. Associated modules will be installed into the /lib/modules directory. The initramfs will be started immediately after loading the kernel to perform hardware auto-detection (just like in the live disk image environments).

```sh
root # ls /boot/vmlinu* /boot/initramfs*
root # ls /lib/modules
```

### Alternative: Manual configuration

#### Introduction

Manually configuring a kernel is often seen as the most difficult procedure a Linux user ever has to perform. Nothing is less true - after configuring a couple of kernels no one remembers that it was difficult!

However, one thing is true: it is vital to know the system when a kernel is configured manually. Most information can be gathered by emerging [sys-apps/pciutils](https://packages.gentoo.org/packages/sys-apps/pciutils) which contains the **lspci** command:

`root # emerge --ask sys-apps/pciutils`

!!! Note
Inside the chroot, it is safe to ignore any pcilib warnings (like pcilib: cannot open /sys/bus/pci/devices) that **lspci** might throw out.

Another source of system information is to run **lsmod** to see what kernel modules the installation CD uses as it might provide a nice hint on what to enable.

Now go to the kernel source directory and execute **make menuconfig**. This will fire up menu-driven configuration screen.

```sh
root # cd /usr/src/linux
root # make menuconfig
```

The Linux kernel configuration has many, many sections. Let's first list some options that must be activated (otherwise Gentoo will not function, or not function properly without additional tweaks). We also have a [Gentoo kernel configuration guide](https://wiki.gentoo.org/wiki/Kernel/Gentoo_Kernel_Configuration_Guide) on the Gentoo wiki that might help out further.

#### Enabling required options

When using [sys-kernel/gentoo-sources](https://packages.gentoo.org/packages/sys-kernel/gentoo-sources), it is strongly recommend the Gentoo-specific configuration options be enabled. These ensure that a minimum of kernel features required for proper functioning is available:

```sh title="KERNEL Enabling Gentoo-specific options"
Gentoo Linux --->
  Generic Driver Options --->
    [*] Gentoo Linux support
    [*]   Linux dynamic and persistent device naming (userspace devfs) support
    [*]   Select options required by Portage features
        Support for init systems, system and service managers  --->
          [*] OpenRC, runit and other script based systems and managers
          [*] systemd
```

Naturally the choice in the last two lines depends on the selected init system ([OpenRC](https://wiki.gentoo.org/wiki/OpenRC) vs. [systemd](https://wiki.gentoo.org/wiki/Systemd)). It does not hurt to have support for both init systems enabled.

When using [sys-kernel/vanilla-sources](https://packages.gentoo.org/packages/sys-kernel/vanilla-sources), the additional selections for init systems will be unavailable. Enabling support is possible, but goes beyond the scope of the handbook.

#### Enabling support for typical system components

Make sure that every driver that is vital to the booting of the system (such as SATA controllers, NVMe block device support, filesystem support, etc.) is compiled in the kernel and not as a module, otherwise the system may not be able to boot completely.

Next select the exact processor type. It is also recommended to enable MCE features (if available) so that users are able to be notified of any hardware problems. On some architectures (such as x86_64), these errors are not printed to dmesg, but to /dev/mcelog. This requires the app-admin/mcelog package.

Also select Maintain a devtmpfs file system to mount at /dev so that critical device files are already available early in the boot process (CONFIG_DEVTMPFS and CONFIG_DEVTMPFS_MOUNT):

KERNEL Enabling devtmpfs support (CONFIG_DEVTMPFS)
Device Drivers --->
  Generic Driver Options --->
    [*] Maintain a devtmpfs filesystem to mount at /dev
    [*]   Automount devtmpfs at /dev, after the kernel mounted the rootfs
Verify SCSI disk support has been activated (CONFIG_BLK_DEV_SD):

KERNEL Enabling SCSI disk support (CONFIG_SCSI, CONFIG_BLK_DEV_SD)
Device Drivers --->
  SCSI device support  ---> 
    <*> SCSI device support
    <*> SCSI disk support
KERNEL Enabling basic SATA and PATA support (CONFIG_ATA_ACPI, CONFIG_SATA_PMP, CONFIG_SATA_AHCI, CONFIG_ATA_BMDMA, CONFIG_ATA_SFF, CONFIG_ATA_PIIX)
Device Drivers --->
  <*> Serial ATA and Parallel ATA drivers (libata)  --->
    [*] ATA ACPI Support
    [*] SATA Port Multiplier support
    <*> AHCI SATA support (ahci)
    [*] ATA BMDMA support
    [*] ATA SFF support (for legacy IDE and PATA)
    <*> Intel ESB, ICH, PIIX3, PIIX4 PATA/SATA support (ata_piix)
Verify basic NVMe support has been enabled:

KERNEL Enable basic NVMe support for Linux 4.4.x (CONFIG_BLK_DEV_NVME)
Device Drivers  --->
  <*> NVM Express block device
KERNEL Enable basic NVMe support for Linux 5.x.x (CONFIG_DEVTMPFS)
Device Drivers --->
  NVME Support --->
    <*> NVM Express block device
It does not hurt to enable the following additional NVMe support:

KERNEL Enabling additional NVMe support (CONFIG_NVME_MULTIPATH, CONFIG_NVME_MULTIPATH, CONFIG_NVME_HWMON, CONFIG_NVME_FC, CONFIG_NVME_TCP, CONFIG_NVME_TARGET, CONFIG_NVME_TARGET_PASSTHRU, CONFIG_NVME_TARGET_LOOP, CONFIG_NVME_TARGET_FC, CONFIG_NVME_TARGET_FCLOOP, CONFIG_NVME_TARGET_TCP
[*] NVMe multipath support
[*] NVMe hardware monitoring
<M> NVM Express over Fabrics FC host driver
<M> NVM Express over Fabrics TCP host driver
<M> NVMe Target support
  [*]   NVMe Target Passthrough support
  <M>   NVMe loopback device support
  <M>   NVMe over Fabrics FC target driver
  < >     NVMe over Fabrics FC Transport Loopback Test driver (NEW)
  <M>   NVMe over Fabrics TCP target support
Now go to File Systems and select support for the filesystems that will be used by the system. Do not compile the file system that is used for the root filesystem as module, otherwise the system may not be able to mount the partition. Also select Virtual memory and /proc file system. Select one or more of the following options as needed by the system:

KERNEL Enable file system support (CONFIG_EXT2_FS, CONFIG_EXT3_FS, CONFIG_EXT4_FS, CONFIG_BTRFS_FS, CONFIG_XFS_FS, CONFIG_MSDOS_FS, CONFIG_VFAT_FS, CONFIG_PROC_FS, and CONFIG_TMPFS)
File systems --->
  <*> Second extended fs support
  <*> The Extended 3 (ext3) filesystem
  <*> The Extended 4 (ext4) filesystem
  <*> Btrfs filesystem support
  <*> XFS filesystem support
  DOS/FAT/NT Filesystems  --->
    <*> MSDOS fs support
    <*> VFAT (Windows-95) fs support
  Pseudo Filesystems --->
    [*] /proc file system support
    [*] Tmpfs virtual memory file system support (former shm fs)
If PPPoE is used to connect to the Internet, or a dial-up modem, then enable the following options (CONFIG_PPP, CONFIG_PPP_ASYNC, and CONFIG_PPP_SYNC_TTY):

KERNEL Enabling PPPoE support (PPPoE, CONFIG_PPPOE, CONFIG_PPP_ASYNC, CONFIG_PPP_SYNC_TTY
Device Drivers --->
  Network device support --->
    <*> PPP (point-to-point protocol) support
    <*> PPP over Ethernet
    <*> PPP support for async serial ports
    <*> PPP support for sync tty ports
The two compression options won't harm but are not definitely needed, neither does the PPP over Ethernet option, that might only be used by ppp when configured to do kernel mode PPPoE.

Don't forget to include support in the kernel for the network (Ethernet or wireless) cards.

Most systems also have multiple cores at their disposal, so it is important to activate Symmetric multi-processing support (CONFIG_SMP):

KERNEL Activating SMP support (CONFIG_SMP)
Processor type and features  --->
  [*] Symmetric multi-processing support
 Note
In multi-core systems, each core counts as one processor.
If USB input devices (like keyboard or mouse) or other USB devices will be used, do not forget to enable those as well:

KERNEL Enable USB and human input device support (CONFIG_HID_GENERIC, CONFIG_USB_HID, CONFIG_USB_SUPPORT, CONFIG_USB_XHCI_HCD, CONFIG_USB_EHCI_HCD, CONFIG_USB_OHCI_HCD, (CONFIG_HID_GENERIC, CONFIG_USB_HID, CONFIG_USB_SUPPORT, CONFIG_USB_XHCI_HCD, CONFIG_USB_EHCI_HCD, CONFIG_USB_OHCI_HCD, CONFIG_USB4)
Device Drivers --->
  HID support  --->
    -*- HID bus support
    <*>   Generic HID driver
    [*]   Battery level reporting for HID devices
      USB HID support  --->
        <*> USB HID transport layer
  [*] USB support  --->
    <*>     xHCI HCD (USB 3.0) support
    <*>     EHCI HCD (USB 2.0) support
    <*>     OHCI HCD (USB 1.1) support
  <*> Unified support for USB4 and Thunderbolt  --->

#### Architecture specific kernel configuration

Make sure to select IA32 Emulation if 32-bit programs should be supported (CONFIG_IA32_EMULATION). Gentoo installs a multilib system (mixed 32-bit/64-bit computing) by default, so unless a no-multilib profile is used, this option is required.

KERNEL Selecting processor types and features
Processor type and features  --->
   [ ] Machine Check / overheating reporting 
   [ ]   Intel MCE Features
   [ ]   AMD MCE Features
   Processor family (AMD-Opteron/Athlon64)  --->
      ( ) Opteron/Athlon64/Hammer/K8
      ( ) Intel P4 / older Netburst based Xeon
      ( ) Core 2/newer Xeon
      ( ) Intel Atom
      ( ) Generic-x86-64
Binary Emulations --->
   [*] IA32 Emulation
Enable GPT partition label support if that was used previously when partitioning the disk (CONFIG_PARTITION_ADVANCED and CONFIG_EFI_PARTITION):

KERNEL Enable support for GPT
-*- Enable the block layer --->
   Partition Types --->
      [*] Advanced partition selection
      [*] EFI GUID Partition support
Enable EFI stub support, EFI variables and EFI Framebuffer in the Linux kernel if UEFI is used to boot the system (CONFIG_EFI, CONFIG_EFI_STUB, CONFIG_EFI_MIXED, CONFIG_EFI_VARS, and CONFIG_FB_EFI):

KERNEL Enable support for UEFI
Processor type and features  --->
    [*] EFI runtime service support 
    [*]   EFI stub support
    [*]     EFI mixed-mode support
 
Device Drivers
    Firmware Drivers  --->
        EFI (Extensible Firmware Interface) Support  --->
            <*> EFI Variable Support via sysfs
    Graphics support  --->
        Frame buffer Devices  --->
            <*> Support for frame buffer devices  --->
                [*]   EFI-based Framebuffer Support

#### Compiling and installing

With the configuration now done, it is time to compile and install the kernel. Exit the configuration and start the compilation process:

root #make && make modules_install
 Note
It is possible to enable parallel builds using make -jX with X being an integer number of parallel tasks that the build process is allowed to launch. This is similar to the instructions about /etc/portage/make.conf earlier, with the MAKEOPTS variable.
When the kernel has finished compiling, copy the kernel image to /boot/. This is handled by the make install command:

root #make install
This will copy the kernel image into /boot/ together with the System.map file and the kernel configuration file.


#### Optional: Building an initramfs

In certain cases it is necessary to build an initramfs - an initial ram-based file system. The most common reason is when important file system locations (like /usr/ or /var/) are on separate partitions. With an initramfs, these partitions can be mounted using the tools available inside the initramfs.

Without an initramfs, there is a risk that the system will not boot properly as the tools that are responsible for mounting the file systems require information that resides on unmounted file systems. An initramfs will pull in the necessary files into an archive which is used right after the kernel boots, but before the control is handed over to the init tool. Scripts on the initramfs will then make sure that the partitions are properly mounted before the system continues booting.

 Important
If using genkernel, it should be used for both building the kernel and the initramfs. When using genkernel only for generating an initramfs, it is crucial to pass --kernel-config=/path/to/kernel.config to genkernel or the generated initramfs may not work with a manually built kernel. Note that manually built kernels go beyond the scope of support for the handbook. See the kernel configuration article for more information.
To install an initramfs, install sys-kernel/dracut first, then have it generate an initramfs:

root #emerge --ask sys-kernel/dracut
root #dracut --kver=5.15.52-gentoo
The initramfs will be stored in /boot/. The resulting file can be found by simply listing the files starting with initramfs:

root #ls /boot/initramfs*
Now continue with Kernel modules.

## Kernel modules

### Listing available kernel modules

!!! Note
Hardware modules are optional to be listed manually. udev will normally load all hardware modules that are detected to be connected in most cases. However, it is not harmful for modules that will be automatically loaded to be listed. Modules cannot be loaded twice; they are either loaded or unloaded. Sometimes exotic hardware requires help to load their drivers.
The modules that need to be loaded during each boot in can be added to /etc/modules-load.d/*.conf files in the format of one module per line. When extra options are needed for the modules, they should be set in /etc/modprobe.d/*.conf files instead.

To view all modules available for a specific kernel version, issue the following find command. Do not forget to substitute "<kernel version>" with the appropriate version of the kernel to search:

root #find /lib/modules/<kernel version>/ -type f -iname '*.o' -or -iname '*.ko' | less

### Force loading particular kernel modules

To force load the kernel to load the 3c59x.ko module (which is the driver for a specific 3Com network card family), edit the /etc/modules-load.d/network.conf file and enter the module name within it.

root #mkdir -p /etc/modules-load.d
root #nano -w /etc/modules-load.d/network.conf
Note that the module's .ko file suffix is insignificant to the loading mechanism and left out of the configuration file:

FILE /etc/modules-load.d/network.confForce loading 3c59x module
3c59x
Continue the installation with Configuring the system.

