# Preparing the disks

## Introduction to block devices

### Block devices

Let's take a good look at disk-oriented aspects of Gentoo Linux and Linux in general, including block devices, partitions, and Linux filesystems. Once the ins and outs of disks are understood, partitions and filesystems can be established for installation.

To begin, let's look at block devices. SCSI and Serial ATA drives are both labeled under device handles such as: /dev/sda, /dev/sdb, /dev/sdc, etc. On more modern machines, PCI Express based NVMe solid state disks have device handles such as /dev/nvme0n1, /dev/nvme0n2, etc.

The following table will help readers determine where to find a certain type of block device on the system:

|Type of device	|Default device handle	|Editorial notes and considerations|
|---|---|---|
|SATA, SAS, SCSI, or USB flash	|/dev/sda	|Found on hardware from roughly 2007 until the present, this device handle is perhaps the most commonly used in Linux. These types of devices can be connected via the SATA bus, SCSI, USB bus as block storage. As example, the first partition on the first SATA device is called /dev/sda1.|
|NVM Express (NVMe)	|/dev/nvme0n1	|The latest in solid state technology, NVMe drives are connected to the PCI Express bus and have the fastest transfer block speeds on the market. Systems from around 2014 and newer may have support for NVMe hardware. The first partition on the first NVMe device is called /dev/nvme0n1p1.|
|MMC, eMMC, and SD	|/dev/mmcblk0|	embedded MMC devices, SD cards, and other types of memory cards can be useful for data storage. That said, many systems may not permit booting from these types of devices. It is suggested to not use these devices for active Linux installations; rather consider using them to transfer files, which is their design goal. Alternatively they could be useful for short-term backups.|

The block devices above represent an abstract interface to the disk. User programs can use these block devices to interact with the disk without worrying about whether the drives are SATA, SCSI, or something else. The program can simply address the storage on the disk as a bunch of contiguous, randomly-accessible 4096-byte (4K) blocks.


### Partition tables

Although it is theoretically possible to use a raw, unpartitioned disk to house a Linux system (when creating a btrfs RAID for example), this is almost never done in practice. Instead, disk block devices are split up into smaller, more manageable block devices. On **amd64** systems, these are called partitions. There are currently two standard partitioning technologies in use: MBR (sometimes also called DOS disklabel) and GPT; these are tied to the two boot process types: legacy BIOS boot and UEFI.

#### GUID Partition Table (GPT)
The GUID Partition Table (GPT) setup (also called GPT disklabel) uses 64-bit identifiers for the partitions. The location in which it stores the partition information is much bigger than the 512 bytes of the MBR partition table (DOS disklabel), which means there is practically no limit on the amount of partitions for a GPT disk. Also the size of a partition is bounded by a much greater limit (almost 8 ZiB - yes, zebibytes).

When a system's software interface between the operating system and firmware is UEFI (instead of BIOS), GPT is almost mandatory as compatibility issues will arise with DOS disklabel.

GPT also takes advantage of checksumming and redundancy. It carries CRC32 checksums to detect errors in the header and partition tables and has a backup GPT at the end of the disk. This backup table can be used to recover damage of the primary GPT near the beginning of the disk.

!!! Important

There are a few caveats regarding GPT:
- Using GPT on a BIOS-based computer works, but then one cannot dual-boot with a Microsoft Windows operating system. The reason is that Microsoft Windows will boot in UEFI mode if it detects a GPT partition label.
- Some buggy (old) motherboard firmware configured to boot in BIOS/CSM/legacy mode might also have problems with booting from GPT labeled disks.

#### Master boot record (MBR) or DOS boot sector

[The Master boot record](https://en.wikipedia.org/wiki/Master_boot_record) boot sector (also called DOS boot sector or DOS disk label) was first introduced in 1983 with PC DOS 2.x. MBR uses 32-bit identifiers for the start sector and length of the partitions, and supports three partition types: primary, extended, and logical. Primary partitions have their information stored in the master boot record itself - a very small (usually 512 bytes) location at the very beginning of a disk. Due to this small space, only four primary partitions are supported (for instance, /dev/sda1 to /dev/sda4).

In order to support more partitions, one of the primary partitions in the MBR can be marked as an extended partition. This partition can then contain additional logical partitions (partitions within a partition).

!!! Important

Although still supported by most motherboard manufacturers, MBR boot sectors and their associated partitioning limitations are considered legacy. Unless working with hardware that is pre-2010, it best to partition a disk with GUID Partition Table. Readers who must proceed with setup type should knowingly acknowledge the following information:
Most post-2010 motherboards consider using MBR boot sectors a legacy (supported, but not ideal) boot mode.
Due to using 32-bit identifiers, partition tables in the MBR cannot address storage space that is larger than 2 TiBs in size.
Unless an extended partition is created, MBR supports a maximum of four partitions.
This setup does not provide a backup boot sector, so if something overwrites the partition table, all partition information will be lost.
That said, MBR and BIOS boot is still frequently used in virtualized cloud environments such as AWS.

The Handbook authors suggest using GPT whenever possible for Gentoo installations.

### Advanced storage

The amd64 Installation CDs provide support for Logical Volume Manager (LVM). LVM increases the flexibility offered by the partitioning setup. It allows to combine partitions and disks into volume groups and define RAID groups or caches on fast SSDs for slow HDs. The installation instructions below will focus on "regular" partitions, but it is good to know LVM is supported if that route is desired. Visit the LVM article for more details. Newcomers beware: although fully supported, LVM is outside the scope of this guide.

Default partitioning scheme
Throughout the remainder of the handbook, we will discuss and explain two cases: 1) GPT partition table and UEFI boot, and 2) MBR partition table and legacy BIOS boot. While it is possible to mix and match, that goes beyond the scope of this manual. As already stated above, installations on modern hardware should use GPT partition table and UEFI boot; as an exception from this rule, MBR and BIOS boot is still frequently used in virtualized (cloud) environments.

The following partitioning scheme will be used as a simple example layout:

Partition	Filesystem	Size	Description
/dev/sda1	fat32 (UEFI) or ext4 (BIOS - aka Legacy boot)	256M	Boot/EFI system partition
/dev/sda2	(swap)	RAM size * 2	Swap partition
/dev/sda3	ext4	Rest of the disk	Root partition
If this suffices as information, the advanced reader can directly skip ahead to the actual partitioning.

Both fdisk and parted are partitioning utilities. fdisk is well known, stable, and recommended for the MBR partition layout. parted was one of the first Linux block device management utilities to support GPT partitions, and provides an alternative. Here, fdisk is used since it has a better text-based user interface.

Before going to the creation instructions, the first set of sections will describe in more detail how partitioning schemes can be created and mention some common pitfalls.


Designing a partition scheme
How many partitions and how big?
The design of disk partition layout is highly dependent on the demands of the system and the file system(s) applied to the device. If there are lots of users, then it is advised to have /home on a separate partition which will increase security and make backups and other types of maintenance easier. If Gentoo is being installed to perform as a mail server, then /var should be a separate partition as all mails are stored inside the /var directory. Game servers may have a separate /opt partition since most gaming server software is installed therein. The reason for these recommendations is similar to the /home directory: security, backups, and maintenance.

In most situations on Gentoo, /usr and /var should be kept relatively large in size. /usr hosts the majority of applications available on the system and the Linux kernel sources (under /usr/src). By default, /var hosts the Gentoo ebuild repository (located at /var/db/repos/gentoo) which, depending on the file system, generally consumes around 650 MiB of disk space. This space estimate excludes the /var/cache/distfiles and /var/cache/binpkgs directories, which will gradually fill with source files and (optionally) binary packages respectively as they are added to the system.

How many partitions and how big very much depends on considering the trade-offs and choosing the best option for the circumstance. Separate partitions or volumes have the following advantages:

Choose the best performing filesystem for each partition or volume.
The entire system cannot run out of free space if one defunct tool is continuously writing files to a partition or volume.
If necessary, file system checks are reduced in time, as multiple checks can be done in parallel (although this advantage is realized more with multiple disks than it is with multiple partitions).
Security can be enhanced by mounting some partitions or volumes read-only, nosuid (setuid bits are ignored), noexec (executable bits are ignored), etc.

However, multiple partitions have certain disadvantages as well:

If not configured properly, the system might have lots of free space on one partition and little free space on another.
A separate partition for /usr/ may require the administrator to boot with an initramfs to mount the partition before other boot scripts start. Since the generation and maintenance of an initramfs is beyond the scope of this handbook, we recommend that newcomers do not use a separate partition for /usr/.
There is also a 15-partition limit for SCSI and SATA unless the disk uses GPT labels.
 Note
Installations that intend to use systemd as the service and init system must have the /usr directory available at boot, either as part of the root filesystem or mounted via an initramfs.
What about swap space?
There is no perfect value for swap space size. The purpose of the space is to provide disk storage to the kernel when internal memory (RAM) is under pressure. A swap space allows for the kernel to move memory pages that are not likely to be accessed soon to disk (swap or page-out), which will free memory in RAM for the current task. Of course, if the pages swapped to disk are suddenly needed, they will need to be put back in memory (page-in) which will take considerably longer than reading from RAM (as disks are very slow compared to internal memory).

When a system is not going to run memory intensive applications or has lots of RAM available, then it probably does not need much swap space. However do note in case of hibernation that swap space is used to store the entire contents of memory (likely on desktop and laptop systems rather than on server systems). If the system requires support for hibernation, then swap space larger than or equal to the amount of memory is necessary.

As a general rule, the swap space size is recommended to be twice the internal memory (RAM). For systems with multiple hard disks, it is wise to create one swap partition on each disk so that they can be utilized for parallel read/write operations. The faster a disk can swap, the faster the system will run when data in swap space must be accessed. When choosing between rotational and solid state disks, it is better for performance to put swap on the SSD. Also, swap files can be used as an alternative to swap partitions; this is mostly interesting for systems with very limited disk space.


What is the EFI System Partition (ESP)?
When installing Gentoo on a system that uses UEFI to boot the operating system (instead of BIOS), then it is important that an EFI System Partition (ESP) is created. The instructions below contain the necessary pointers to correctly handle this operation. The EFI system partition is not required when booting in BIOS/Legacy mode.

The ESP must be a FAT variant (sometimes shown as vfat on Linux systems). The official UEFI specification denotes FAT12, 16, or 32 filesystems will be recognized by the UEFI firmware, although FAT32 is recommended for the ESP. After partitioning, format the ESP accordingly:

root #mkfs.fat -F 32 /dev/sda1
 Important
If the ESP is not formatted with a FAT variant, the system's UEFI firmware will not find the bootloader (or Linux kernel) and will most likely be unable to boot the system!

What is the BIOS boot partition?
A BIOS boot partition is only needed when combining a GPT partition layout with GRUB2 in BIOS/Legacy boot mode. It is not required when booting in EFI/UEFI mode, and also not required when using a MBR table. It is a very small (1 to 2 MB) partition in which boot loaders like GRUB2 can put additional data that doesn't fit in the allocated storage. It will not be used in this guide.

Partitioning the disk with GPT for UEFI
The following parts explain how to create the example partition layout for a GPT / UEFI boot installation using fdisk. The example partition layout was mentioned earlier:

Partition	Description
/dev/sda1	EFI system (and boot) partition
/dev/sda2	Swap partition
/dev/sda3	Root partition
Change the partition layout according to personal preference.

Viewing the current partition layout
fdisk is a popular and powerful tool to split a disk into partitions. Fire up fdisk against the disk (in our example, we use /dev/sda):

root #fdisk /dev/sda
Use the p key to display the disk's current partition configuration:

Command (m for help):p
Disk /dev/sda: 28.89 GiB, 31001149440 bytes, 60549120 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 21AAD8CF-DB67-0F43-9374-416C7A4E31EA
 
Device        Start      End  Sectors  Size Type
/dev/sda1      2048   526335   524288  256M EFI System
/dev/sda2    526336  2623487  2097152    1G Linux swap
/dev/sda3   2623488 19400703 16777216    8G Linux filesystem
/dev/sda4  19400704 60549086 41148383 19.6G Linux filesystem
This particular disk was configured to house two Linux filesystems (each with a corresponding partition listed as "Linux") as well as a swap partition (listed as "Linux swap").

Creating a new disklabel / removing all partitions
Type g to create a new GPT disklabel on the disk; this will remove all existing partitions.

Command (m for help):g
Created a new GPT disklabel (GUID: 87EA4497-2722-DF43-A954-368E46AE5C5F).
For an existing GPT disklabel (see the output of p above), alternatively consider removing the existing partitions one by one from the disk. Type d to delete a partition. For instance, to delete an existing /dev/sda1:

Command (m for help):d
Partition number (1-4): 1
The partition has now been scheduled for deletion. It will no longer show up when printing the list of partitions (p, but it will not be erased until the changes have been saved. This allows users to abort the operation if a mistake was made - in that case, type q immediately and hit Enter and the partition will not be deleted.

Repeatedly type p to print out a partition listing and then type d and the number of the partition to delete it. Eventually, the partition table will be empty:

Command (m for help):p
Disk /dev/sda: 28.89 GiB, 31001149440 bytes, 60549120 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 87EA4497-2722-DF43-A954-368E46AE5C5F
Now that the in-memory partition table is empty, we're ready to create the partitions.

Creating the EFI system partition (ESP)
First create a small EFI system partition, which will also be mounted as /boot. Type n to create a new partition, followed by 1 to select the first partition. When prompted for the first sector, make sure it starts from 2048 (which may be needed for the boot loader) and hit Enter. When prompted for the last sector, type +256M to create a partition 256 Mbyte in size:

Command (m for help):n
Partition number (1-128, default 1): 1
First sector (2048-60549086, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-60549086, default 60549086): +256M
 
Created a new partition 1 of type 'Linux filesystem' and of size 256 MiB.
Mark the partition as EFI system partition:

Command (m for help):t
Selected partition 1
Partition type (type L to list all types): 1
Changed type of partition 'Linux filesystem' to 'EFI System'.
Creating the swap partition
Next, to create the swap partition, type n to create a new partition, then type 2 to create the second partition, /dev/sda2. When prompted for the first sector, hit Enter. When prompted for the last sector, type +4G (or any other size needed for the swap space) to create a partition 4GB in size.

Command (m for help):n
Partition number (2-128, default 2): 
First sector (526336-60549086, default 526336): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (526336-60549086, default 60549086): +4G
 
Created a new partition 2 of type 'Linux filesystem' and of size 4 GiB.
After all this is done, type t to set the partition type, 2 to select the partition just created and then type in 19 to set the partition type to "Linux Swap".

Command (m for help):t
Partition number (1,2, default 2): 2
Partition type (type L to list all types): 19
 
Changed type of partition 'Linux filesystem' to 'Linux swap'.
Creating the root partition
Finally, to create the root partition, type n to create a new partition. Then type 3 to create the third partition, /dev/sda3. When prompted for the first sector, hit Enter. When prompted for the last sector, hit Enter to create a partition that takes up the rest of the remaining space on the disk. After completing these steps, typing p should display a partition table that looks similar to this:

Command (m for help):p
Disk /dev/sda: 28.89 GiB, 31001149440 bytes, 60549120 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 87EA4497-2722-DF43-A954-368E46AE5C5F
 
Device       Start      End  Sectors  Size Type
/dev/sda1     2048   526335   524288  256M EFI System
/dev/sda2   526336  8914943  8388608    4G Linux swap
/dev/sda3  8914944 60549086 51634143 24.6G Linux filesystem
Saving the partition layout
To save the partition layout and exit fdisk, type w.

Command (m for help):w
With the partitions created, it is now time to put filesystems on them.

Partitioning the disk with MBR for BIOS / legacy boot
The following explains how to create the example partition layout for a MBR / BIOS legacy boot installation. The example partition layout mentioned earlier is now:

Partition	Description
/dev/sda1	Boot partition
/dev/sda2	Swap partition
/dev/sda3	Root partition
Change the partition layout according to personal preference.

Viewing the current partition layout
Fire up fdisk against the disk (in our example, we use /dev/sda):

root #fdisk /dev/sda
Use the p key to display the disk's current partition configuration:

Command (m for help):p
Disk /dev/sda: 28.89 GiB, 31001149440 bytes, 60549120 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 21AAD8CF-DB67-0F43-9374-416C7A4E31EA
 
Device        Start      End  Sectors  Size Type
/dev/sda1      2048   526335   524288  256M EFI System
/dev/sda2    526336  2623487  2097152    1G Linux swap
/dev/sda3   2623488 19400703 16777216    8G Linux filesystem
/dev/sda4  19400704 60549086 41148383 19.6G Linux filesystem
This particular disk was until now configured to house two Linux filesystems (each with a corresponding partition listed as "Linux") as well as a swap partition (listed as "Linux swap"), using a GPT table.

Creating a new disklabel / removing all partitions
Type o to create a new MBR disklabel (here also named DOS disklabel) on the disk; this will remove all existing partitions.

Command (m for help):o
Created a new DOS disklabel with disk identifier 0xe04e67c4.
The device contains 'gpt' signature and it will be removed by a write command. See fdisk(8) man page and --wipe option for more details.
For an existing DOS disklabel (see the output of p above), alternatively consider removing the existing partitions one by one from the disk. Type d to delete a partition. For instance, to delete an existing /dev/sda1:

Command (m for help):d
Partition number (1-4): 1
The partition has now been scheduled for deletion. It will no longer show up when printing the list of partitions (p, but it will not be erased until the changes have been saved. This allows users to abort the operation if a mistake was made - in that case, type q immediately and hit Enter and the partition will not be deleted.

Repeatedly type p to print out a partition listing and then type d and the number of the partition to delete it. Eventually, the partition table will be empty:

Command (m for help):p
Disk /dev/sda: 28.89 GiB, 31001149440 bytes, 60549120 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe04e67c4
Now we're ready to create the partitions.

Creating the boot partition
First, create a small partition which will be mounted as /boot. Type n to create a new partition, followed by p for a primary partition and 1 to select the first primary partition. When prompted for the first sector, make sure it starts from 2048 (which may be needed for the boot loader) and hit Enter. When prompted for the last sector, type +256M to create a partition 256 Mbyte in size:

Command (m for help):n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-60549119, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-60549119, default 60549119): +256M
 
Created a new partition 1 of type 'Linux' and of size 256 MiB.
Creating the swap partition
Next, to create the swap partition, type n to create a new partition, then p, then type 2 to create the second primary partition, /dev/sda2. When prompted for the first sector, hit Enter. When prompted for the last sector, type +4G (or any other size needed for the swap space) to create a partition 4GB in size.

Command (m for help):n
Partition type
   p   primary (1 primary, 0 extended, 3 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (2-4, default 2): 2
First sector (526336-60549119, default 526336): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (526336-60549119, default 60549119): +4G
 
Created a new partition 2 of type 'Linux' and of size 4 GiB.
After all this is done, type t to set the partition type, 2 to select the partition just created and then type in 82 to set the partition type to "Linux Swap".

Command (m for help):t
Partition number (1,2, default 2): 2
Hex code (type L to list all codes): 82

<!--T:179-->
Changed type of partition 'Linux' to 'Linux swap / Solaris'.
Creating the root partition
Finally, to create the root partition, type n to create a new partition. Then type p and 3 to create the third primary partition, /dev/sda3. When prompted for the first sector, hit Enter. When prompted for the last sector, hit Enter to create a partition that takes up the rest of the remaining space on the disk. After completing these steps, typing p should display a partition table that looks similar to this:

Command (m for help):p
Disk /dev/sda: 28.89 GiB, 31001149440 bytes, 60549120 sectors
Disk model: DataTraveler 2.0
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe04e67c4
 
Device     Boot   Start      End  Sectors  Size Id Type
/dev/sda1          2048   526335   524288  256M 83 Linux
/dev/sda2        526336  8914943  8388608    4G 82 Linux swap / Solaris
/dev/sda3       8914944 60549119 51634176 24.6G 83 Linux
Saving the partition layout
To save the partition layout and exit fdisk, type w.

Command (m for help):w
Now it is time to put filesystems on the partitions.


Creating file systems
Introduction
Now that the partitions have been created, it is time to place a filesystem on them. In the next section the various file systems that Linux supports are described. Readers that already know which filesystem to use can continue with Applying a filesystem to a partition. The others should read on to learn about the available filesystems...

Filesystems
Linux supports several dozen filesystems, although many of them are only wise to deploy for specific purposes. Only certain filesystems may be found stable on the amd64 architecture - it is advised to read up on the filesystems and their support state before selecting a more experimental one for important partitions. ext4 is the recommended all-purpose, all-platform filesystem. The below is a non-exaustive list

btrfs
A next generation filesystem that provides many advanced features such as snapshotting, self-healing through checksums, transparent compression, subvolumes, and integrated RAID. Kernels prior to 5.4.y are not guaranteed to be safe to use with btrfs in production because fixes for serious issues are only present in the more recent releases of the LTS kernel branches. Filesystem corruption issues are common on older kernel branches, with anything older than 4.4.y being especially unsafe and prone to corruption. Corruption is more likely on older kernels (than 5.4.y) when compression is enabled. RAID 5/6 and quota groups unsafe on all versions of btrfs. Furthermore, btrfs can counter-intuitively fail filesystem operations with ENOSPC when df reports free space due to internal fragmentation (free space pinned by DATA + SYSTEM chunks, but needed in METADATA chunks). Additionally, a single 4K reference to a 128M extent inside btrfs can cause free space to be present, but unavailable for allocations. This can also cause btrfs to return ENOSPC when free space is reported by df. Installing sys-fs/btrfsmaintenance and configuring the scripts to run periodically can help to reduce the possibility of ENOSPC issues by rebalancing btrfs, but it will not eliminate the risk of ENOSPC when free space is present. Some workloads will never hit ENOSPC while others will. If the risk of ENOSPC in production is unacceptable, you should use something else. If using btrfs, be certain to avoid configurations known to have issues. With the exception of ENOSPC, information on the issues present in btrfs in the latest kernel branches is available at the btrfs status page.
ext4
Initially created as a fork of ext3, ext4 brings new features, performance improvements, and removal of size limits with moderate changes to the on-disk format. It can span volumes up to 1 EB and with maximum file size of 16TB. Instead of the classic ext2/3 bitmap block allocation, ext4 uses extents, which improve large file performance and reduce fragmentation. Ext4 also provides more sophisticated block allocation algorithms (delayed allocation and multiblock allocation) giving the filesystem driver more ways to optimize the layout of data on the disk. Ext4 is the recommended all-purpose all-platform filesystem.
f2fs
The Flash-Friendly File System was originally created by Samsung for the use with NAND flash memory. As of Q2, 2016, this filesystem is still considered immature, but it is a decent choice when installing Gentoo to microSD cards, USB drives, or other flash-based storage devices.
JFS
IBM's high-performance journaling filesystem. JFS is a light, fast, and reliable B+tree-based filesystem with good performance in various conditions.
XFS
A filesystem with metadata journaling which comes with a robust feature-set and is optimized for scalability. XFS seems to be less forgiving to various hardware problems, but has been continuously upgraded to include modern features.
VFAT
Also known as FAT32, is supported by Linux but does not support standard UNIX permission settings. It is mostly used for interoperability/interchange with other operating systems (Microsoft Windows or Apple's macOS) but is also a necessity for some system bootloader firmware (like UEFI). Users of UEFI systems will need an EFI System Partition formatted with VFAT in order to boot.
NTFS
This "New Technology" filesystem is the flagship filesystem of Microsoft Windows since Windows NT 3.1. Similarly to VFAT, it does not store UNIX permission settings or extended attributes necessary for BSD or Linux to function properly, therefore it should not be used as a filesystem for most cases. It should only be used for interoperability/interchange with Microsoft Windows systems (note the emphasis on only).
Applying a filesystem to a partition
To create a filesystem on a partition or volume, there are user space utilities available for each possible filesystem. Click the filesystem's name in the table below for additional information on each filesystem:

Filesystem	Creation command	On minimal CD?	Package
btrfs	mkfs.btrfs	 Yes	sys-fs/btrfs-progs
ext4	mkfs.ext4	 Yes	sys-fs/e2fsprogs
f2fs	mkfs.f2fs	 Yes	sys-fs/f2fs-tools
jfs	mkfs.jfs	 Yes	sys-fs/jfsutils
reiserfs (deprecated)	mkfs.reiserfs	 Yes	sys-fs/reiserfsprogs
xfs	mkfs.xfs	 Yes	sys-fs/xfsprogs
vfat	mkfs.vfat	 Yes	sys-fs/dosfstools
NTFS	mkfs.ntfs	 Yes	sys-fs/ntfs3g
For instance, to have the EFI system partition partition (/dev/sda1) as FAT32 and the root partition (/dev/sda3) as ext4 as used in the example partition structure, the following commands would be used:

root #mkfs.vfat -F 32 /dev/sda1
root #mkfs.ext4 /dev/sda3
When using ext4 on a small partition (less than 8 GiB), then the file system must be created with the proper options to reserve enough inodes. This can be done using one of the following commands, respectively:

root #mkfs.ext4 -T small /dev/<device>
This will generally quadruple the number of inodes for a given file system as its "bytes-per-inode" reduces from one every 16kB to one every 4kB.

Now create the filesystems on the newly created partitions (or logical volumes).

Activating the swap partition
mkswap is the command that is used to initialize swap partitions:

root #mkswap /dev/sda2
To activate the swap partition, use swapon:

root #swapon /dev/sda2
Create and activate the swap with the commands mentioned above.

Mounting the root partition
 Tip
Users of non-Gentoo installation media will need to create the mount point by running:
root #mkdir --parents /mnt/gentoo
Now that the partitions have been initialized and are housing a filesystem, it is time to mount those partitions. Use the mount command, but don't forget to create the necessary mount directories for every partition created. As an example we mount the root partition:

root #mount /dev/sda3 /mnt/gentoo
 Note
If /tmp/ needs to reside on a separate partition, be sure to change its permissions after mounting:
root #chmod 1777 /mnt/gentoo/tmp
This also holds for /var/tmp.
Later in the instructions the proc filesystem (a virtual interface with the kernel) as well as other kernel pseudo-filesystems will be mounted. But first we install the Gentoo installation files.