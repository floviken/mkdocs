# Configuring the bootloader

## Selecting a boot loader

With the Linux kernel configured, system tools installed and configuration files edited, it is time to install the last important piece of a Linux installation: the boot loader.

The boot loader is responsible for firing up the Linux kernel upon boot - without it, the system would not know how to proceed when the power button has been pressed.

For amd64, we document how to configure either GRUB or LILO for BIOS based systems, and GRUB or efibootmgr for UEFI systems.

In this section of the Handbook a delineation has been made between emerging the boot loader's package and installing a boot loader to a system disk. Here the term emerge will be used to ask Portage to make the software package available to the system. The term install will signify the boot loader copying files or physically modifying appropriate sections of the system's disk drive in order to render the boot loader activated and ready to operate on the next power cycle.

Default: GRUB
By default, the majority of Gentoo systems now rely upon GRUB (found in the sys-boot/grub package), which is the direct successor to GRUB Legacy. With no additional configuration, GRUB gladly supports older BIOS ("pc") systems. With a small amount of configuration, necessary before build time, GRUB can support more than a half a dozen additional platforms. For more information, consult the Prerequisites section of the GRUB article.

Emerge
When using an older BIOS system supporting only MBR partition tables, no additional configuration is needed in order to emerge GRUB:

root #emerge --ask --verbose sys-boot/grub
A note for UEFI users: running the above command will output the enabled GRUB_PLATFORMS values before emerging. When using UEFI capable systems, users will need to ensure GRUB_PLATFORMS="efi-64" is enabled (as it is the case by default). If that is not the case for the setup, GRUB_PLATFORMS="efi-64" will need to be added to the /etc/portage/make.conf file before emerging GRUB so that the package will be built with EFI functionality:

root #echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
root #emerge --ask sys-boot/grub
If GRUB was somehow emerged without enabling GRUB_PLATFORMS="efi-64", the line (as shown above) can be added to make.conf and then dependencies for the world package set can be re-calculated by passing the --update --newuse options to emerge:
root #emerge --ask --update --newuse --verbose sys-boot/grub
The GRUB software has now been merged to the system, but not yet installed.

Install
Next, install the necessary GRUB files to the /boot/grub/ directory via the grub-install command. Presuming the first disk (the one where the system boots from) is /dev/sda, one of the following commands will do:

When using BIOS:
root #grub-install /dev/sda
When using UEFI:
 Important
Make sure the EFI system partition has been mounted before running grub-install. It is possible for grub-install to install the GRUB EFI file (grubx64.efi) into the wrong directory without providing any indication the wrong directory was used.
root #grub-install --target=x86_64-efi --efi-directory=/boot
 Note
Modify the --efi-directory option to the root of the EFI System Partition. This is necessary if the /boot partition was not formatted as a FAT variant.
 Important
If grub-install returns an error like Could not prepare Boot variable: Read-only file system, it may be necessary to remount the efivars special mount as read-write in order to succeed:
root #mount -o remount,rw,nosuid,nodev,noexec --types efivarfs efivarfs /sys/firmware/efi/efivars
Some motherboard manufacturers seem to only support the /efi/boot/ directory location for the .EFI file in the EFI System Partition (ESP). The GRUB installer can perform this operation automatically with the --removable option. Verify the ESP is mounted before running the following commands. Presuming the ESP is mounted at /boot (as suggested earlier), execute:

root #grub-install --target=x86_64-efi --efi-directory=/boot --removable
This creates the default directory defined by the UEFI specification, and then copies the grubx64.efi file to the 'default' EFI file location defined by the same specification.

Configure
Next, generate the GRUB configuration based on the user configuration specified in the /etc/default/grub file and /etc/grub.d scripts. In most cases, no configuration is needed by users as GRUB will automatically detect which kernel to boot (the highest one available in /boot/) and what the root file system is. It is also possible to append kernel parameters in /etc/default/grub using the GRUB_CMDLINE_LINUX variable.

To generate the final GRUB configuration, run the grub-mkconfig command:

root #grub-mkconfig -o /boot/grub/grub.cfg
Generating grub.cfg ...
Found linux image: /boot/vmlinuz-6.1.38-gentoo
Found initrd image: /boot/initramfs-genkernel-amd64-6.1.38-gentoo
done
The output of the command must mention that at least one Linux image is found, as those are needed to boot the system. If an initramfs is used or genkernel was used to build the kernel, the correct initrd image should be detected as well. If this is not the case, go to /boot/ and check the contents using the ls command. If the files are indeed missing, go back to the kernel configuration and installation instructions.

 Tip
The os-prober utility can be used in conjunction with GRUB to detect other operating systems from attached drives. Windows 7, 8.1, 10, and other distributions of Linux are detectable. Those desiring dual boot systems should emerge the sys-boot/os-prober package then re-run the grub-mkconfig command (as seen above). If detection problems are encountered be sure to read the GRUB article in its entirety before asking the Gentoo community for support.
Alternative 1: LILO
Emerge
LILO, the LInuxLOader, is the tried and true workhorse of Linux boot loaders. However, it lacks features when compared to GRUB. LILO is still used because, on some systems, GRUB does not work and LILO does. Of course, it is also used because some people know LILO and want to stick with it. Either way, Gentoo supports both bootloaders.

Installing LILO is a breeze; just use emerge.

root #emerge --ask sys-boot/lilo
Configure
To configure LILO, first create /etc/lilo.conf:

root #nano -w /etc/lilo.conf
In the configuration file, sections are used to refer to the bootable kernel. Make sure that the kernel files (with kernel version) and initramfs files are known, as they need to be referred to in this configuration file.

 Note
If the root filesystem is JFS, add an append="ro" line after each boot item since JFS needs to replay its log before it allows read-write mounting.
FILE /etc/lilo.confExample LILO configuration
boot=/dev/sda             # Install LILO in the MBR
prompt                    # Give the user the chance to select another section
timeout=50                # Wait 5 (five) seconds before booting the default section
default=gentoo            # When the timeout has passed, boot the "gentoo" section
compact                   # This drastically reduces load time and keeps the map file smaller; may fail on some systems
  
image=/boot/vmlinuz-6.1.38-gentoo
  label=gentoo            # Name we give to this section
  read-only               # Start with a read-only root. Do not alter!
  root=/dev/sda3          # Location of the root filesystem
  
image=/boot/vmlinuz-6.1.38-gentoo
  label=gentoo.rescue     # Name we give to this section
  read-only               # Start with a read-only root. Do not alter!
  root=/dev/sda3         # Location of the root filesystem
  append="init=/bin/bb"   # Launch the Gentoo static rescue shell
  
# The next two lines are for dual booting with a Windows system.
# In this example, Windows is hosted on /dev/sda6.
other=/dev/sda6
  label=windows
 Note
If a different partitioning scheme and/or kernel image is used, adjust accordingly.
If an initramfs is necessary, then change the configuration by referring to this initramfs file and telling the initramfs where the root device is located:

FILE /etc/lilo.confAdding initramfs information to a boot entry
image=/boot/vmlinuz-6.1.38-gentoo
  label=gentoo
  read-only
  append="root=/dev/sda3"
  initrd=/boot/initramfs-genkernel-amd64-6.1.38-gentoo
If additional options need to be passed to the kernel, use an append statement. For instance, to add the video statement to enable framebuffer:

FILE /etc/lilo.confAdding video parameter to the boot options
image=/boot/vmlinuz-6.1.38-gentoo
  label=gentoo
  read-only
  root=/dev/sda3
  append="video=uvesafb:mtrr,ywrap,1024x768-32@85"
Users that used genkernel should know that their kernels use the same boot options as is used for the installation CD. For instance, if SCSI device support needs to be enabled, add doscsi as kernel option.

Now save the file and exit.

Install
To finish up, run the /sbin/lilo executable so LILO can apply the /etc/lilo.conf settings to the system (i.e. install itself on the disk). Keep in mind that /sbin/lilo must be executed each time a new kernel is installed or a change has been made to the lilo.conf file in order for the system to boot if the filename of the kernel has changed.

root #/sbin/lilo
Alternative 2: efibootmgr
On UEFI based systems, the UEFI firmware on the system (in other words the primary bootloader), can be directly manipulated to look for UEFI boot entries. Such systems do not need to have additional (also known as secondary) bootloaders like GRUB in order to help boot the system. With that being said, the reason EFI-based bootloaders such as GRUB exist is to extend the functionality of UEFI systems during the boot process. Using efibootmgr is really for those who desire to take a minimalist (although more rigid) approach to booting their system; using GRUB (see above) is easier for the majority of users because it offers a flexible approach when booting UEFI systems.

Remember sys-boot/efibootmgr application is not a bootloader; it is a tool to interact with the UEFI firmware and update its settings, so that the Linux kernel that was previously installed can be booted with additional options (if necessary), or to allow multiple boot entries. This interaction is done through the EFI variables (hence the need for kernel support of EFI vars).

Be sure to read through the EFI stub kernel article before continuing. The kernel must have specific options enabled to be directly bootable by the system's UEFI firmware. It might be necessary to recompile the kernel. It is also a good idea to take a look at the efibootmgr article.

 Note
To reiterate, efibootmgr is not a requirement to boot an UEFI system. The Linux kernel itself can be booted immediately, and additional kernel command-line options can be built-in to the Linux kernel (there is a kernel configuration option called CONFIG_CMDLINE that allows the user to specify boot parameters as command-line options. Even an initramfs can be 'built-in' to the kernel.
Those that have decided to take this approach must install the software:

root #emerge --ask sys-boot/efibootmgr
Then, create the /boot/efi/boot/ location, and then copy the kernel into this location, calling it bootx64.efi:

root #mkdir -p /boot/efi/boot
root #cp /boot/vmlinuz-* /boot/efi/boot/bootx64.efi
Next, tell the UEFI firmware that a boot entry called "Gentoo" is to be created, which has the freshly compiled EFI stub kernel:

root #efibootmgr --create --disk /dev/sda --part 2 --label "Gentoo" --loader "\efi\boot\bootx64.efi"
If an initial RAM file system (initramfs) is used, add the proper boot option to it:

root #efibootmgr -c -d /dev/sda -p 2 -L "Gentoo" -l "\efi\boot\bootx64.efi" initrd='\initramfs-genkernel-amd64-6.1.38-gentoo'
 Note
The use of \ as directory separator is mandatory when using UEFI definitions.
With these changes done, when the system reboots, a boot entry called "Gentoo" will be available.

Alternative 3: Syslinux
Syslinux is yet another bootloader alternative for the amd64 architecture. It supports MBR and, as of version 6.00, it supports EFI boot. PXE (network) boot and lesser-known options are also supported. Although Syslinux is a popular bootloader for many it is unsupported by the Handbook. Readers can find information on emerging and then installing this bootloader in the Syslinux article.


Rebooting the system
Exit the chrooted environment and unmount all mounted partitions. Then type in that one magical command that initiates the final, true test: reboot.

root #exit
cdimage ~#cd
cdimage ~#umount -l /mnt/gentoo/dev{/shm,/pts,}
cdimage ~#umount -R /mnt/gentoo
cdimage ~#reboot
Do not forget to remove the bootable CD, otherwise the CD might be booted again instead of the new Gentoo system.

Once rebooted in the freshly installed Gentoo environment, finish up with Finalizing the Gentoo installation.