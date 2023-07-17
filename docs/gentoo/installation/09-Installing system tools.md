# Installing system tools

## System logger

### OpenRC

Some tools are missing from the stage3 archive because several packages provide the same functionality. It is now up to the user to choose which ones to install.

The first tool to decision is a logging mechanism for the system. Unix and Linux have an excellent history of logging capabilities - if needed, everything that happens on the system can be logged in a log file.

Gentoo offers several system logger utilities. A few of these include:


- [app-admin/sysklogd](https://packages.gentoo.org/packages/app-admin/sysklogd) - Offers the traditional set of system logging daemons. The default logging configuration works well out of the box which makes this package a good option for beginners.

- [app-admin/syslog-ng](https://packages.gentoo.org/packages/app-admin/syslog-ng) - An advanced system logger. Requires additional configuration for anything beyond logging to one big file. More advanced users may choose this package based on its logging potential; be aware additional configuration is a necessity for any kind of smart logging.

- [app-admin/metalog](https://packages.gentoo.org/packages/app-admin/metalog) - A highly-configurable system logger.
There may be other system logging utilities available through the Gentoo ebuild repository as well, since the number of available packages increases on a daily basis.

 Tip
If syslog-ng is going to be used, it is recommended to install and configure logrotate. syslog-ng does not provide any rotation mechanism for the log files. Newer versions (>= 2.0) of sysklogd however handle their own log rotation.
To install the system logger of choice, emerge it. On OpenRC, add it to the default runlevel using rc-update. The following example installs and activates app-admin/sysklogd as the system's syslog utility:

root #emerge --ask app-admin/sysklogd
root #rc-update add sysklogd default

### systemd

While a selection of logging mechanisms are presented for OpenRC-based systems, systemd includes a built-in logger called the systemd-journald service. The systemd-journald service is capable of handling most of the logging functionality outlined in the previous system logger section. That is to say, the majority of installations that will run systemd as the system and service manager can safely skip adding a additional syslog utilities.

See man journalctl for more details on using journalctl to query and review the systems logs.

For a number of reasons, such as the case of forwarding logs to a central host, it may be important to include redundant system logging mechanisms on a systemd-based system. This is a irregular occurrence for the handbook's typical audience and considered an advanced use case. It is therefore not covered by the handbook.

## Optional: Cron daemon

### OpenRC

Although it is optional and not required for every system, it is wise to install a cron daemon.

A cron daemon executes commands on scheduled intervals. Internals could be daily, weekly, or monthly, once every Tuesday, once every other week, etc. A wise system administrator will leverage the cron daemon to automate routine system maintenance tasks.

All cron daemons support high levels of granularity for scheduled tasks, and generally include the ability to send an email or other form of notification if a scheduled task does not complete as expected.

Gentoo offers several possible cron daemons, including:

sys-process/cronie - cronie is based on the original cron and has security and configuration enhancements like the ability to use PAM and SELinux.
sys-process/dcron - This lightweight cron daemon aims to be simple and secure, with just enough features to stay useful.
sys-process/fcron - A command scheduler with extended capabilities over cron and anacron.
sys-process/bcron - A younger cron system designed with secure operations in mind. To do this, the system is divided into several separate programs, each responsible for a separate task, with strictly controlled communications between parts.

#### cronie
The following example uses sys-process/cronie:

root #emerge --ask sys-process/cronie
Add cronie to the default system runlevel, which will automatically start it on power up:

root #rc-update add cronie default

#### Alternative: dcron

root #emerge --ask sys-process/dcron
If dcron is the go forward cron agent, an additional initialization command needs to be executed:

root #crontab /etc/crontab

#### Alternative: fcron

root #emerge --ask sys-process/fcron
If fcron is the selected scheduled task handler, an additional emerge step is required:

root #emerge --config sys-process/fcron

#### Alternative: bcron

bcron is a younger cron agent with built-in privilege separation.

root # emerge --ask sys-process/bcron

### systemd

Similar to system logging, systemd-based systems include support for scheduled tasks out-of-the-box in the form of timers. systemd timers can run at a system-level or a user-level and include the same functionality that a traditional cron daemon would provide. Unless redundant capabilities are necessary, installing an additional task scheduler such as a cron daemon is generally unnecessary and can be safely skipped.

Optional: File indexing
In order to index the file system to provide faster file location capabilities, install sys-apps/mlocate.

root #emerge --ask sys-apps/mlocate
Optional: Remote shell access
 Tip
opensshd's default configuration does not allow root to login as a remote user. Please create a non-root user and configure it appropriately to allow access post-installation if required, or adjust /etc/ssh/sshd_config to allow root.
To be able to access the system remotely after installation, sshd must be configured to start on boot.

OpenRC
To add the sshd init script to the default runlevel on OpenRC:

root #rc-update add sshd default
If serial console access is needed (which is possible in case of remote servers), agetty must be configured.

Uncomment the serial console section in /etc/inittab:

root #nano -w /etc/inittab
# SERIAL CONSOLES
s0:12345:respawn:/sbin/agetty 9600 ttyS0 vt100
s1:12345:respawn:/sbin/agetty 9600 ttyS1 vt100
systemd
To enable the SSH server, run:

root #systemctl enable sshd
To enable serial console support, run:

root #systemctl enable getty@tty1.service
Optional: Shell completion
Bash
Bash is the default shell for Gentoo systems, and therefore installing completion extensions can aid in efficiency and convenience to managing the system. The app-shells/bash-completion package will install completions available for Gentoo specific commands, as well as many other common commands and utilities:

root #emerge --ask app-shells/bash-completion
Post installation, bash completion for specific commands can managed through eselect. See the Shell completion integrations section of the bash article for more details.

Time synchronization
It is important to use some method of synchronizing the system clock. This is usually done via the NTP protocol and software. Other implementations using the NTP protocol exist, like Chrony.

To set up Chrony, for example:

root #emerge --ask net-misc/chrony
OpenRC
On OpenRC, run:

root #rc-update add chronyd default
systemd
On systemd, run:

root #systemctl enable chronyd.service
Alternatively, systemd users may wish to use the simpler systemd-timesyncd SNTP client which is installed by default.

root #systemctl enable systemd-timesyncd.service
Filesystem tools
Depending on the filesystems used, it may be necessary to install the required file system utilities (for checking the filesystem integrity, (re)formatting file systems, etc.). Note that ext4 user space tools (sys-fs/e2fsprogs are already installed as a part of the @system set.

The following table lists the tools to install if a certain filesystem tools will be needed in the installed environment.

Filesystem	Package
XFS	sys-fs/xfsprogs
ext4	sys-fs/e2fsprogs
VFAT (FAT32, ...)	sys-fs/dosfstools
Btrfs	sys-fs/btrfs-progs
ZFS	sys-fs/zfs
JFS	sys-fs/jfsutils
ReiserFS	sys-fs/reiserfsprogs
It's recommended that sys-block/io-scheduler-udev-rules is installed for the correct scheduler behavior with e.g. nvme devices:

root #emerge --ask sys-block/io-scheduler-udev-rules
 Tip
For more information on filesystems in Gentoo see the filesystem article.
Networking tools
If networking was previously configured in the Configuring the system step and network setup is complete, then this 'networking tools' section can be safely skipped. In this case, proceed with the section on Configuring a bootloader.

Installing a DHCP client
 Important
Most users will need a DHCP client to connect to their network. If none was installed, then the system might not be able to get on the network thus making it impossible to download a DHCP client afterwards.
A DHCP client obtains automatically an IP address for one or more network interface(s) using netifrc scripts. We recommend the use of net-misc/dhcpcd (see also dhcpcd):

root #emerge --ask net-misc/dhcpcd
Optional: Installing a PPPoE client
If PPP is used to connect to the internet, install the net-dialup/ppp package:

root #emerge --ask net-dialup/ppp
Optional: Install wireless networking tools
If the system will be connecting to wireless networks, install the net-wireless/iw package for Open or WEP networks and/or the net-wireless/wpa_supplicant package for WPA or WPA2 networks. iw is also a useful basic diagnostic tool for scanning wireless networks.

root #emerge --ask net-wireless/iw net-wireless/wpa_supplicant
Now continue with Configuring the bootloader.

