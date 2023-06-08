# Configuring the network

## Automatic network detection
Maybe it just works?

If the system is plugged into an Ethernet network with a DHCP server, it is very likely that the networking configuration has already been set up automatically. If so, then the many included network-aware commands on the installation media such as ssh, scp, ping, irssi, wget, and links, among others, will work immediately.

### Determine interface names

#### ifconfig command

If networking has been configured, the ifconfig command should list one or more network interfaces (besides lo).
In the example below eth0 shows up :

```sh 
root # ifconfig
eth0      Link encap:Ethernet  HWaddr 00:50:BA:8F:61:7A
          inet addr:192.168.0.2  Bcast:192.168.0.255  Mask:255.255.255.0
          inet6 addr: fe80::50:ba8f:617a/10 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:1498792 errors:0 dropped:0 overruns:0 frame:0
          TX packets:1284980 errors:0 dropped:0 overruns:0 carrier:0
          collisions:1984 txqueuelen:100
          RX bytes:485691215 (463.1 Mb)  TX bytes:123951388 (118.2 Mb)
          Interrupt:11 Base address:0xe800 
```

As a result of the shift towards predictable network interface names, the interface name on the system can be quite different from the old eth0 naming convention. Recent installation media might show regular network interfaces names like eno0, ens1, or enp5s0. Look for the interface in the ifconfig output that has an IP address related to the local network.

!!! Tip
```If no interfaces are displayed when the standard ifconfig command is used, try using the same command with the -a option. This option forces the utility to show all network interfaces detected by the system whether they be in an up or down state. If ifconfig -a produces no results then the hardware is faulty or the driver for the interface has not been loaded into the kernel. Both situations reach beyond the scope of this Handbook. Contact #gentoo (webchat) for support.```

#### ip command

As an alternative to **ifconfig**, the **ip** command can be used to determine interface names. The following example shows the output of **ip addr** (of another system so the information shown is different from the previous example):

``` sh 
root # ip addr
2: eno1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether e8:40:f2:ac:25:7a brd ff:ff:ff:ff:ff:ff
    inet 10.0.20.77/22 brd 10.0.23.255 scope global eno1
       valid_lft forever preferred_lft forever
    inet6 fe80::ea40:f2ff:feac:257a/64 scope link 
       valid_lft forever preferred_lft forever
```

The interface name in the above example directly follows the number; it is eno1.

In the remainder of this document, the handbook will assume that the operating network interface is called eth0.

### Optional: Configure any proxies

If the Internet is accessed through a proxy, then it is necessary to set up proxy information during the installation. It is very easy to define a proxy: just define a variable which contains the proxy server information.

In most cases, it is sufficient to define the variables using the server hostname. As an example, we assume the proxy is called proxy.gentoo.org and the port is 8080.

To set up an HTTP proxy (for HTTP and HTTPS traffic):

`root # export http_proxy="http://proxy.gentoo.org:8080"`

To set up an FTP proxy:

`root # export ftp_proxy="ftp://proxy.gentoo.org:8080"`

To set up an RSYNC proxy:

`root # export RSYNC_PROXY="proxy.gentoo.org:8080"`

If the proxy requires a username and password, use the following syntax for the variable:

``` sh title="CODE Adding username/password to the proxy variable"
http://username:password@proxy.gentoo.org:8080
```

### Testing the network

Try pinging your ISP's DNS server (found in /etc/resolv.conf) and a web site of choice. This ensures that the network is functioning properly and that the network packets are reaching the net, DNS name resolution is working correctly, etc.

`root # ping -c 3 www.gentoo.org`

If this all works, then the remainder of this chapter can be skipped to jump right to the next step of the installation instructions ([Preparing the disks](docs/gentoo/installation/)).

## Automatic network configuration

If the network doesn't work immediately, some installation media allow the user to use **net-setup** (for regular or wireless networks), **pppoe-setup** (for ADSL users) or **pptp** (for PPTP users).

If the installation medium does not contain any of these tools, continue with the [Manual network configuration](## Manual network configuration).

- Regular Ethernet users should continue with [Default: Using net-setup](###Default: Using net-setup)
- ADSL users should continue with [Alternative: Using PPP](### Alternative: Using PPP)
- PPTP users should continue with [Alternative: Using PPTP](### Alternative: Using PPTP)

### Default: Using net-setup
The simplest way to set up networking if it didn't get configured automatically is to run the **net-setup script**:

` root # net-setup eth0`

**net-setup** will ask some questions about the network environment. When all is done, the network connection should work. Test the network connection as stated before. If the tests are positive, congratulations! Skip the rest of this section and continue with Preparing the disks.

If the network still doesn't work, continue with Manual network configuration.

### Alternative: Using PPP
Assuming PPPoE is needed to connect to the Internet, the installation CD (any version) has made things easier by including ppp. Use the provided pppoe-setup script to configure the connection. During the setup the Ethernet device that is connected to your ADSL modem, the username and password, the IPs of the DNS servers and if a basic firewall is needed or not will be asked.

root #pppoe-setup
root #pppoe-start
If something goes wrong, double-check that the username and password are correct by looking at etc/ppp/pap-secrets or /etc/ppp/chap-secrets and make sure to use the right Ethernet device. If the Ethernet device does not exist, the appropriate network modules need to be loaded. In that case continue with Manual network configuration as it will explain how to load the appropriate network modules there.

If everything worked, continue with Preparing the disks.

### Alternative: Using PPTP
If PPTP support is needed, use pptpclient which is provided by the installation CDs. But first make sure that the configuration is correct. Edit /etc/ppp/pap-secrets or /etc/ppp/chap-secrets so it contains the correct username/password combination:

root #nano -w /etc/ppp/chap-secrets
Then adjust /etc/ppp/options.pptp if necessary:

root #nano -w /etc/ppp/options.pptp
When all that is done, run pptp (along with the options that couldn't be set in options.pptp) to connect the server:

root #pptp <server ipv4 address>
Now continue with Preparing the disks.

## Manual network configuration
Loading the appropriate network kernel modules
When the Installation CD boots, it tries to detect all the hardware devices and loads the appropriate kernel modules (drivers) to support the hardware. In the vast majority of cases, it does a very good job. However, in some cases, it may not auto-load the kernel modules needed to communicate properly with the present network hardware.

If net-setup or pppoe-setup failed, then it is possible that the network card wasn't found immediately. This means users may have to load the appropriate kernel modules manually.

To find out what kernel modules are provided for networking, use the ls command:

root #ls /lib/modules/`uname -r`/kernel/drivers/net
If a driver is found for the network device, use modprobe to load the kernel module. For instance, to load the pcnet32 module:

root #modprobe pcnet32
To check if the network card is now detected, use ifconfig. A detected network card would result in something like this (again, eth0 here is just an example):

root #ifconfig eth0
eth0      Link encap:Ethernet  HWaddr FE:FD:00:00:00:00  
          BROADCAST NOARP MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
If however the following error is shown, the network card is not detected:

root #ifconfig eth0
eth0: error fetching interface information: Device not found
The available network interface names on the system can be listed through the /sys file system:

root #ls /sys/class/net
dummy0  eth0  lo  sit0  tap0  wlan0
In the above example, 6 interfaces are found. The eth0 one is most likely the (wired) Ethernet adapter whereas wlan0 is the wireless one.

Assuming that the network card is now detected, retry net-setup or pppoe-setup again (which should work now), but for the hardcore people we explain how to configure the network manually as well.

Select one of the following sections based on your network setup:

Using DHCP for automatic IP retrieval
Preparing for wireless access if a wireless network is used
Understanding network terminology explains the basics about networking
Using ifconfig and route explains how to set up networking manually
Using DHCP
DHCP (Dynamic Host Configuration Protocol) makes it possible to automatically receive networking information (IP address, netmask, broadcast address, gateway, nameservers etc.). This only works if a DHCP server is in the network (or if the ISP provider provides a DHCP service). To have a network interface receive this information automatically, use dhcpcd:

root #dhcpcd eth0
Some network administrators require that the hostname and domainname provided by the DHCP server is used by the system. In that case, use:

root #dhcpcd -HD eth0
If this works (try pinging some Internet server, like Google's 8.8.8.8 or Cloudflare's 1.1.1.1), then everything is set and ready to continue. Skip the rest of this section and continue with Preparing the disks.

Preparing for wireless access
 Note
Support for the iw command might be architecture-specific. If the command is not available see if the net-wireless/iw package is available for the current architecture. The iw command will be unavailable unless the net-wireless/iw package has been installed.
When using a wireless (802.11) card, the wireless settings need to be configured before going any further. To see the current wireless settings on the card, one can use iw. Running iw might show something like:

root #iw dev wlp9s0 info
Interface wlp9s0
	ifindex 3
	wdev 0x1
	addr 00:00:00:00:00:00
	type managed
	wiphy 0
	channel 11 (2462 MHz), width: 20 MHz (no HT), center1: 2462 MHz
	txpower 30.00 dBm
To check for a current connection:

root #iw dev wlp9s0 link
Not connected.
or

root #iw dev wlp9s0 link
Connected to 00:00:00:00:00:00 (on wlp9s0)
	SSID: GentooNode
	freq: 2462
	RX: 3279 bytes (25 packets)
	TX: 1049 bytes (7 packets)
	signal: -23 dBm
	tx bitrate: 1.0 MBit/s
 Note
Some wireless cards may have a device name of wlan0 or ra0 instead of wlp9s0. Run ip link to determine the correct device name.
For most users, there are only two settings needed to connect, the ESSID (aka wireless network name) and, optionally, the WEP key.

First, ensure the interface is active:
root #ip link set dev wlp9s0 up
To connect to an open network with the name GentooNode:
root #iw dev wlp9s0 connect -w GentooNode
To connect with a hex WEP key, prefix the key with d::
root #iw dev wlp9s0 connect -w GentooNode key 0:d:1234123412341234abcd
To connect with an ASCII WEP key:
root #iw dev wlp9s0 connect -w GentooNode key 0:some-password
 Note
If the wireless network is set up with WPA or WPA2, then wpa_supplicant needs to be used. For more information on configuring wireless networking in Gentoo Linux, please read the Wireless networking chapter in the Gentoo Handbook.
Confirm the wireless settings by using iw dev wlp9s0 link. Once wireless is working, continue configuring the IP level networking options as described in the next section (Understanding network terminology) or use the net-setup tool as described previously.

Understanding network terminology
 Note
If the IP address, broadcast address, netmask and nameservers are known, then skip this subsection and continue with Using ifconfig and route.
If all of the above fails, the network will need to be configured manually. This is not difficult at all. However, some knowledge of network terminology and basic concepts might be necessary. After reading this section, users will know what a gateway is, what a netmask serves for, how a broadcast address is formed and why systems need nameservers.

In a network, hosts are identified by their IP address (Internet Protocol address). Such an address is perceived as a combination of four numbers between 0 and 255. Well, at least when using IPv4 (IP version 4). In reality, such an IPv4 address consists of 32 bits (ones and zeros). Let's view an example:

CODE Example of an IPv4 address
IP Address (numbers):   192.168.0.2
IP Address (bits):      11000000 10101000 00000000 00000010
                        -------- -------- -------- --------
                           192      168       0        2
 Note
The successor of IPv4, IPv6, uses 128 bits (ones and zeros). In this section, the focus is on IPv4 addresses.
Such an IP address is unique to a host as far as all accessible networks are concerned (i.e. every host that one wants to be able to reach must have a unique IP address). In order to distinguish between hosts inside and outside a network, the IP address is divided in two parts: the network part and the host part.

The separation is written down with the netmask, a collection of ones followed by a collection of zeros. The part of the IP that can be mapped on the ones is the network-part, the other one is the host-part. As usual, the netmask can be written down as an IP address.

CODE Example of network/host separation
IP address:    192      168      0         2
            11000000 10101000 00000000 00000010
Netmask:    11111111 11111111 11111111 00000000
               255      255     255        0
           +--------------------------+--------+
                    Network              Host
In other words, 192.168.0.14 is part of the example network, but 192.168.1.2 is not.

The broadcast address is an IP address with the same network-part as the network, but with only ones as host-part. Every host on the network listens to this IP address. It is truly meant for broadcasting packets.

CODE Broadcast address
IP address:    192      168      0         2
            11000000 10101000 00000000 00000010
Broadcast:  11000000 10101000 00000000 11111111
               192      168      0        255
           +--------------------------+--------+
                     Network             Host
To be able to surf on the Internet, each computer in the network must know which host shares the Internet connection. This host is called the gateway. Since it is a regular host, it has a regular IP address (for instance 192.168.0.1).

Previously we stated that every host has its own IP address. To be able to reach this host by a name (instead of an IP address) we need a service that translates a name (such as dev.gentoo.org) to an IP address (such as 64.5.62.82). Such a service is called a name service. To use such a service, the necessary name servers need to be defined in /etc/resolv.conf.

In some cases, the gateway also serves as a nameserver. Otherwise the nameservers provided by the ISP need to be entered in this file.

To summarize, the following information is needed before continuing:

Network item	Example
The system IP address	192.168.0.2
Netmask	255.255.255.0
Broadcast	192.168.0.255
Gateway	192.168.0.1
Nameserver(s)	195.130.130.5, 195.130.130.133
Using ifconfig and route
Employing tools from the sys-apps/net-tools package, setting up the network manually generally consists of three steps:

Assign an IP address using the ifconfig command.
Set up routing to the gateway using the route command.
Finish up by placing valid nameserver IPs in the /etc/resolv.conf file.
To assign an IP address, the IP address, broadcast address, and netmask are needed. Execute the following command, substituting ${IP_ADDR} with the target IP address, ${BROADCAST} with the target broadcast address, and ${NETMASK} with the target netmask:

root #ifconfig eth0 ${IP_ADDR} broadcast ${BROADCAST} netmask ${NETMASK} up
To configure routing using route, substitute the ${GATEWAY} value with the appropriate gateway IP address:

root #route add default gw ${GATEWAY}
Now open the /etc/resolv.conf file using a text editor:

root #nano -w /etc/resolv.conf
Fill in the nameserver(s) using the following as a template substituting ${NAMESERVER1} and ${NAMESERVER2} with nameserver IP addresses as necessary. More than one nameserver can be added:

FILE /etc/resolv.confDefault resolv.conf template
nameserver ${NAMESERVER1}
nameserver ${NAMESERVER2}
Now test the network by pinging an Internet server (like Google's 8.8.8.8 or Cloudflare's 1.1.1.1). Once connected, continue with Preparing the disks.