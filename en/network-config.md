# Networking

Networks consist of two or more devices, such as computer systems, printers,
and related equipment which are connected by either physical cabling or
wireless links for the purpose of sharing and distributing information among
the connected devices.

This section provides general and specific information pertaining to
networking, including an overview of network concepts and detailed discussion
of popular network protocols.

# Network Configuration

Ubuntu ships with a number of graphical utilities to configure your network
devices. This document is geared toward server administrators and will focus
on managing your network on the command line.

## Ethernet Interfaces

Ethernet interfaces are identified by the system using the naming convention
of *ethX*, where *X* represents a numeric value. The first Ethernet interface
is typically identified as *eth0*, the second as *eth1*, and all others should
move up in numerical order.

### Identify Ethernet Interfaces

To quickly identify all available Ethernet interfaces, you can use the
ifconfig command as shown below.

    ifconfig -a | grep eth
    eth0      Link encap:Ethernet  HWaddr 00:15:c5:4a:16:5a

Another application that can help identify all network interfaces available to
your system is the lshw command. In the example below, lshw shows a single
Ethernet interface with the logical name of *eth0* along with bus information,
driver details and all supported capabilities.

    sudo lshw -class network
      *-network
           description: Ethernet interface
           product: BCM4401-B0 100Base-TX
           vendor: Broadcom Corporation
           physical id: 0
           bus info: pci@0000:03:00.0
           logical name: eth0
           version: 02
           serial: 00:15:c5:4a:16:5a
           size: 10MB/s
           capacity: 100MB/s
           width: 32 bits
           clock: 33MHz
           capabilities: (snipped for brevity)
           configuration: (snipped for brevity)
           resources: irq:17 memory:ef9fe000-ef9fffff

### Ethernet Interface Logical Names {#ethernet-interface-names}

Interface logical names are configured in the file
`/etc/udev/rules.d/70-persistent-net.rules.` If you would like control which
interface receives a particular logical name, find the line matching the
interfaces physical MAC address and modify the value of *NAME=ethX* to the
desired logical name. Reboot the system to commit your changes.

### Ethernet Interface Settings

ethtool is a program that displays and changes Ethernet card settings such as
auto-negotiation, port speed, duplex mode, and Wake-on-LAN. It is not
installed by default, but is available for installation in the repositories.

    sudo apt install ethtool

The following is an example of how to view supported features and configured
settings of an Ethernet interface.

    sudo ethtool eth0
    Settings for eth0:
            Supported ports: [ TP ]
            Supported link modes:   10baseT/Half 10baseT/Full 
                                    100baseT/Half 100baseT/Full 
                                    1000baseT/Half 1000baseT/Full 
            Supports auto-negotiation: Yes
            Advertised link modes:  10baseT/Half 10baseT/Full 
                                    100baseT/Half 100baseT/Full 
                                    1000baseT/Half 1000baseT/Full 
            Advertised auto-negotiation: Yes
            Speed: 1000Mb/s
            Duplex: Full
            Port: Twisted Pair
            PHYAD: 1
            Transceiver: internal
            Auto-negotiation: on
            Supports Wake-on: g
            Wake-on: d
            Current message level: 0x000000ff (255)
            Link detected: yes

Changes made with the ethtool command are temporary and will be lost after a
reboot. If you would like to retain settings, simply add the desired ethtool
command to a *pre-up* statement in the interface configuration file
`/etc/network/interfaces`.

The following is an example of how the interface identified as *eth0* could be
permanently configured with a port speed of 1000Mb/s running in full duplex
mode.

    auto eth0
    iface eth0 inet static
    pre-up /sbin/ethtool -s eth0 speed 1000 duplex full

> **Note**
>
> Although the example above shows the interface configured to use the
> *static* method, it actually works with other methods as well, such as DHCP.
> The example is meant to demonstrate only proper placement of the *pre-up*
> statement in relation to the rest of the interface configuration.

## IP Addressing

The following section describes the process of configuring your systems IP
address and default gateway needed for communicating on a local area network
and the Internet.

### Temporary IP Address Assignment {#temp-ip-assignment}

For temporary network configurations, you can use standard commands such as
ip, ifconfig and route, which are also found on most other GNU/Linux operating
systems. These commands allow you to configure settings which take effect
immediately, however they are not persistent and will be lost after a reboot.

To temporarily configure an IP address, you can use the ifconfig command in
the following manner. Just modify the IP address and subnet mask to match your
network requirements.

    sudo ifconfig eth0 10.0.0.100 netmask 255.255.255.0

To verify the IP address configuration of eth0, you can use the ifconfig
command in the following manner.

    ifconfig eth0
    eth0      Link encap:Ethernet  HWaddr 00:15:c5:4a:16:5a  
              inet addr:10.0.0.100  Bcast:10.0.0.255  Mask:255.255.255.0
              inet6 addr: fe80::215:c5ff:fe4a:165a/64 Scope:Link
              UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
              RX packets:466475604 errors:0 dropped:0 overruns:0 frame:0
              TX packets:403172654 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:1000 
              RX bytes:2574778386 (2.5 GB)  TX bytes:1618367329 (1.6 GB)
              Interrupt:16 

To configure a default gateway, you can use the route command in the following
manner. Modify the default gateway address to match your network requirements.

    sudo route add default gw 10.0.0.1 eth0

To verify your default gateway configuration, you can use the route command in
the following manner.

    route -n
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    10.0.0.0        0.0.0.0         255.255.255.0   U     1      0        0 eth0
    0.0.0.0         10.0.0.1        0.0.0.0         UG    0      0        0 eth0

If you require DNS for your temporary network configuration, you can add DNS
server IP addresses in the file `/etc/resolv.conf`. In general, editing
`/etc/resolv.conf` directly is not recommanded, but this is a temporary and
non-persistent configuration. The example below shows how to enter two DNS
servers to `/etc/resolv.conf`, which should be changed to servers appropriate
for your network. A more lengthy description of the proper persistent way to
do DNS client configuration is in a following section.

    nameserver 8.8.8.8
    nameserver 8.8.4.4

If you no longer need this configuration and wish to purge all IP
configuration from an interface, you can use the ip command with the flush
option as shown below.

    ip addr flush eth0

> **Note**
>
> Flushing the IP configuration using the ip command does not clear the
> contents of `/etc/resolv.conf`. You must remove or modify those entries
> manually, or re-boot which should also cause `/etc/resolv.conf`, which is
> actually now a symlink to `/run/resolvconf/resolv.conf`, to be re-written.

### Dynamic IP Address Assignment (DHCP Client) {#dynamic-ip-addressing}

To configure your server to use DHCP for dynamic address assignment, add the
*dhcp* method to the inet address family statement for the appropriate
interface in the file `/etc/network/interfaces`. The example below assumes you
are configuring your first Ethernet interface identified as *eth0*.

    auto eth0
    iface eth0 inet dhcp

By adding an interface configuration as shown above, you can manually enable
the interface through the ifup command which initiates the DHCP process via
dhclient.

    sudo ifup eth0

To manually disable the interface, you can use the ifdown command, which in
turn will initiate the DHCP release process and shut down the interface.

    sudo ifdown eth0

### Static IP Address Assignment {#static-ip-addressing}

To configure your system to use a static IP address assignment, add the
*static* method to the inet address family statement for the appropriate
interface in the file `/etc/network/interfaces`. The example below assumes you
are configuring your first Ethernet interface identified as *eth0*. Change the
*address*, *netmask*, and *gateway* values to meet the requirements of your
network.

    auto eth0
    iface eth0 inet static
    address 10.0.0.100
    netmask 255.255.255.0
    gateway 10.0.0.1

By adding an interface configuration as shown above, you can manually enable
the interface through the ifup command.

    sudo ifup eth0

To manually disable the interface, you can use the ifdown command.

    sudo ifdown eth0

### Loopback Interface

The loopback interface is identified by the system as *lo* and has a default
IP address of 127.0.0.1. It can be viewed using the ifconfig command.

    ifconfig lo
    lo        Link encap:Local Loopback  
              inet addr:127.0.0.1  Mask:255.0.0.0
              inet6 addr: ::1/128 Scope:Host
              UP LOOPBACK RUNNING  MTU:16436  Metric:1
              RX packets:2718 errors:0 dropped:0 overruns:0 frame:0
              TX packets:2718 errors:0 dropped:0 overruns:0 carrier:0
              collisions:0 txqueuelen:0 
              RX bytes:183308 (183.3 KB)  TX bytes:183308 (183.3 KB)

By default, there should be two lines in `/etc/network/interfaces` responsible
for automatically configuring your loopback interface. It is recommended that
you keep the default settings unless you have a specific purpose for changing
them. An example of the two default lines are shown below.

    auto lo
    iface lo inet loopback

## Name Resolution

Name resolution as it relates to IP networking is the process of mapping IP
addresses to hostnames, making it easier to identify resources on a network.
The following section will explain how to properly configure your system for
name resolution using DNS and static hostname records.

### DNS Client Configuration

Traditionally, the file `/etc/resolv.conf` was a static configuration file
that rarely needed to be changed or automatically changed via DCHP client
hooks. Nowadays, a computer can switch from one network to another quite often
and the *resolvconf* framework is now being used to track these changes and
update the resolver's configuration automatically. It acts as an intermediary
between programs that supply nameserver information and applications that need
nameserver information. Resolvconf gets populated with information by a set of
hook scripts related to network interface configuration. The most notable
difference for the user is that any change manually done to `/etc/resolv.conf`
will be lost as it gets overwritten each time something triggers resolvconf.
Instead, resolvconf uses DHCP client hooks, and `/etc/network/interfaces` to
generate a list of nameservers and domains to put in `/etc/resolv.conf`, which
is now a symlink:

    /etc/resolv.conf -> ../run/resolvconf/resolv.conf

To configure the resolver, add the IP addresses of the nameservers that are
appropriate for your network in the file `/etc/network/interfaces`. You can
also add an optional DNS suffix search-lists to match your network domain
names. For each other valid resolv.conf configuration option, you can include,
in the stanza, one line beginning with that option name with a **dns-**
prefix. The resulting file might look like the following:

    iface eth0 inet static
        address 192.168.3.3
        netmask 255.255.255.0
        gateway 192.168.3.1
        dns-search example.com
        dns-nameservers 192.168.3.45 192.168.8.10

The *search* option can also be used with multiple domain names so that DNS
queries will be appended in the order in which they are entered. For example,
your network may have multiple sub-domains to search; a parent domain of
*example.com*, and two sub-domains, *sales.example.com* and *dev.example.com*.

If you have multiple domains you wish to search, your configuration might look
like the following:

    iface eth0 inet static
        address 192.168.3.3
        netmask 255.255.255.0
        gateway 192.168.3.1
        dns-search example.com sales.example.com dev.example.com
        dns-nameservers 192.168.3.45 192.168.8.10

If you try to ping a host with the name of *server1*, your system will
automatically query DNS for its Fully Qualified Domain Name (FQDN) in the
following order:

1.  server1**.example.com**

2.  server1**.sales.example.com**

3.  server1**.dev.example.com**

If no matches are found, the DNS server will provide a result of *notfound*
and the DNS query will fail.

### Static Hostnames

Static hostnames are locally defined hostname-to-IP mappings located in the
file `/etc/hosts`. Entries in the `hosts` file will have precedence over DNS
by default. This means that if your system tries to resolve a hostname and it
matches an entry in /etc/hosts, it will not attempt to look up the record in
DNS. In some configurations, especially when Internet access is not required,
servers that communicate with a limited number of resources can be
conveniently set to use static hostnames instead of DNS.

The following is an example of a `hosts` file where a number of local servers
have been identified by simple hostnames, aliases and their equivalent Fully
Qualified Domain Names (FQDN's).

    127.0.0.1   localhost
    127.0.1.1   ubuntu-server
    10.0.0.11   server1 server1.example.com vpn
    10.0.0.12   server2 server2.example.com mail
    10.0.0.13   server3 server3.example.com www
    10.0.0.14   server4 server4.example.com file

> **Note**
>
> In the above example, notice that each of the servers have been given
> aliases in addition to their proper names and FQDN's. *Server1* has been
> mapped to the name *vpn*, *server2* is referred to as *mail*, *server3* as
> *www*, and *server4* as *file*.

### Name Service Switch Configuration {#name-service-switch-config}

The order in which your system selects a method of resolving hostnames to IP
addresses is controlled by the Name Service Switch (NSS) configuration file
`/etc/nsswitch.conf`. As mentioned in the previous section, typically static
hostnames defined in the systems `/etc/hosts` file have precedence over names
resolved from DNS. The following is an example of the line responsible for
this order of hostname lookups in the file `/etc/nsswitch.conf`.

    hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4

-   **files** first tries to resolve static hostnames located in `/etc/hosts`.

-   **mdns4\_minimal** attempts to resolve the name using Multicast DNS.

-   **\[NOTFOUND=return\]** means that any response of *notfound* by the
    preceding *mdns4\_minimal* process should be treated as authoritative and
    that the system should not try to continue hunting for an answer.

-   **dns** represents a legacy unicast DNS query.

-   **mdns4** represents a Multicast DNS query.

To modify the order of the above mentioned name resolution methods, you can
simply change the *hosts:* string to the value of your choosing. For example,
if you prefer to use legacy Unicast DNS versus Multicast DNS, you can change
the string in `/etc/nsswitch.conf` as shown below.

    hosts:          files dns [NOTFOUND=return] mdns4_minimal mdns4

## Bridging

Bridging multiple interfaces is a more advanced configuration, but is very
useful in multiple scenarios. One scenario is setting up a bridge with
multiple network interfaces, then using a firewall to filter traffic between
two network segments. Another scenario is using bridge on a system with one
interface to allow virtual machines direct access to the outside network. The
following example covers the latter scenario.

Before configuring a bridge you will need to install the bridge-utils package.
To install the package, in a terminal enter:

    sudo apt install bridge-utils

Next, configure the bridge by editing `/etc/network/interfaces`:

    auto lo
    iface lo inet loopback

    auto br0
    iface br0 inet static
            address 192.168.0.10
            network 192.168.0.0
            netmask 255.255.255.0
            broadcast 192.168.0.255
            gateway 192.168.0.1
            bridge_ports eth0
            bridge_fd 9
            bridge_hello 2
            bridge_maxage 12
            bridge_stp off

> **Note**
>
> Enter the appropriate values for your physical interface and network.

Now bring up the bridge:

    sudo ifup br0

The new bridge interface should now be up and running. The brctl provides
useful information about the state of the bridge, controls which interfaces
are part of the bridge, etc. See `man brctl` for more information.

## Resources {#network-config-resources}

-   The [Ubuntu Wiki Network page] has links to articles covering more
    advanced network configuration.

-   The [resolvconf man page] has more information on resolvconf.

-   The [interfaces man page] has details on more options for
    `/etc/network/interfaces`.

-   The [dhclient man page] has details on more options for configuring DHCP
    client settings.

-   For more information on DNS client configuration see the [resolver man
    page]. Also, Chapter 6 of O'Reilly's [Linux Network Administrator's Guide]
    is a good source of resolver and name service configuration information.

-   For more information on *bridging* see the [brctl man page] and the Linux
    Foundation's [Networking-Bridge] page.

# TCP/IP

The Transmission Control Protocol and Internet Protocol (TCP/IP) is a standard
set of protocols developed in the late 1970s by the Defense Advanced Research
Projects Agency (DARPA) as a means of communication between different types of
computers and computer networks. TCP/IP is the driving force of the Internet,
and thus it is the most popular set of network protocols on Earth.

## TCP/IP Introduction

The two protocol components of TCP/IP deal with different aspects of computer
networking. *Internet Protocol*, the "IP" of TCP/IP is a connectionless
protocol which deals only with network packet routing using the *IP Datagram*
as the basic unit of networking information. The IP Datagram consists of a
header followed by a message. The *Transmission Control Protocol* is the "TCP"
of TCP/IP and enables network hosts to establish connections which may be used
to exchange data streams. TCP also guarantees that the data between
connections is delivered and that it arrives at one network host in the same
order as sent from another network host.

## TCP/IP Configuration

The TCP/IP protocol configuration consists of several elements which must be
set by editing the appropriate configuration files, or deploying solutions
such as the Dynamic Host Configuration Protocol (DHCP) server which in turn,
can be configured to provide the proper TCP/IP configuration settings to
network clients automatically. These configuration values must be set
correctly in order to facilitate the proper network operation of your Ubuntu
system.

The common configuration elements of TCP/IP and their purposes are as follows:

-   **IP address** The IP address is a unique identifying string expressed as
    four decimal numbers ranging from zero (0) to two-hundred and fifty-five
    (255), separated by periods, with each of the four numbers representing
    eight (8) bits of the address for a total length of thirty-two (32) bits
    for the whole address. This format is called *dotted quad notation*.

-   **Netmask** The Subnet Mask (or simply, *netmask*) is a local bit mask, or
    set of flags which separate the portions of an IP address significant to
    the network from the bits significant to the *subnetwork*. For example, in
    a Class C network, the standard netmask is 255.255.255.0 which masks the
    first three bytes of the IP address and allows the last byte of the IP
    address to remain available for specifying hosts on the subnetwork.

-   **Network Address** The Network Address represents the bytes comprising
    the network portion of an IP address. For example, the host 12.128.1.2 in
    a Class A network would use 12.0.0.0 as the network address, where twelve
    (12) represents the first byte of the IP address, (the network part) and
    zeroes (0) in all of the remaining three bytes to represent the potential
    host values. A network host using the private IP address 192.168.1.100
    would in turn use a Network Address of 192.168.1.0, which specifies the
    first three bytes of the Class C 192.168.1 network and a zero (0) for all
    the possible hosts on the network.

-   **Broadcast Address** The Broadcast Address is an IP address which allows
    network data to be sent simultaneously to all hosts on a given subnetwork
    rather than specifying a particular host. The standard general broadcast
    address for IP networks is 255.255.255.255, but this broadcast address
    cannot be used to send a broadcast message to every host on the Internet
    because routers block it. A more appropriate broadcast address is set to
    match a specific subnetwork. For example, on the private Class C IP
    network, 192.168.1.0, the broadcast address is 192.168.1.255. Broadcast
    messages are typically produced by network protocols such as the Address
    Resolution Protocol (ARP) and the Routing Information Protocol (RIP).

-   **Gateway Address** A Gateway Address is the IP address through which a
    particular network, or host on a network, may be reached. If one network
    host wishes to communicate with another network host, and that host is not
    located on the same network, then a *gateway* must be used. In many cases,
    the Gateway Address will be that of a router on the same network, which
    will in turn pass traffic on to other networks or hosts, such as
    Internet hosts. The value of the Gateway Address setting must be correct,
    or your system will not be able to reach any hosts beyond those on the
    same network.

-   **Nameserver Address** Nameserver Addresses represent the IP addresses of
    Domain Name Service (DNS) systems, which resolve network hostnames into
    IP addresses. There are three levels of Nameserver Addresses, which may be
    specified in order of precedence: The *Primary* Nameserver, the
    *Secondary* Nameserver, and the *Tertiary* Nameserver. In order for your
    system to be able to resolve network hostnames into their corresponding IP
    addresses, you must specify valid Nameserver Addresses which you are
    authorized to use in your system's TCP/IP configuration. In many cases
    these addresses can and will be provided by your network service provider,
    but many free and publicly accessible nameservers are available for use,
    such as the Level3 (Verizon) servers with IP addresses from 4.2.2.1
    to 4.2.2.6.

    > **Tip**
    >
    > The IP address, Netmask, Network Address, Broadcast Address, Gateway
    > Address, and Nameserver Addresses are typically specified via the
    > appropriate directives in the file `/etc/network/interfaces`. For more
    > information, view the system manual page for `interfaces`, with the
    > following command typed at a terminal prompt:

    Access the system manual page for `interfaces` with the following command:

        man interfaces

## IP Routing

IP routing is a means of specifying and discovering paths in a TCP/IP network
along which network data may be sent. Routing uses a set of *routing tables*
to direct the forwarding of network data packets from their source to the
destination, often via many intermediary network nodes known as *routers*.
There are two primary forms of IP routing: *Static Routing* and *Dynamic
Routing.*

Static routing involves manually adding IP routes to the system's routing
table, and this is usually done by manipulating the routing table with the
route command. Static routing enjoys many advantages over dynamic routing,
such as simplicity of implementation on smaller networks, predictability (the
routing table is always computed in advance, and thus the route is precisely
the same each time it is used), and low overhead on other routers and network
links due to the lack of a dynamic routing protocol. However, static routing
does present some disadvantages as well. For example, static routing is
limited to small networks and does not scale well. Static routing also fails
completely to adapt to network outages and failures along the route due to the
fixed nature of the route.

Dynamic routing depends on large networks with multiple possible IP routes
from a source to a destination and makes use of special routing protocols,
such as the Router Information Protocol (RIP), which handle the automatic
adjustments in routing tables that make dynamic routing possible. Dynamic
routing has several advantages over static routing, such as superior
scalability and the ability to adapt to failures and outages along network
routes. Additionally, there is less manual configuration of the routing
tables, since routers learn from one another about their existence and
available routes. This trait also eliminates the possibility of introducing
mistakes in the routing tables via human error. Dynamic routing is not
perfect, however, and presents disadvantages such as heightened complexity and
additional network overhead from router communications, which does not
immediately benefit the end users, but still consumes network bandwidth.

## TCP and UDP

TCP is a connection-based protocol, offering error correction and guaranteed
delivery of data via what is known as *flow control*. Flow control determines
when the flow of a data stream needs to be stopped, and previously sent data
packets should to be re-sent due to problems such as *collisions*, for
example, thus ensuring complete and accurate delivery of the data. TCP is
typically used in the exchange of important information such as database
transactions.

The User Datagram Protocol (UDP), on the other hand, is a *connectionless*
protocol which seldom deals with the transmission of important data because it
lacks flow control or any other method to ensure reliable delivery of the
data. UDP is commonly used in such applications as audio and video streaming,
where it is considerably faster than TCP due to the lack of error correction
and flow control, and where the loss of a few packets is not generally
catastrophic.

## ICMP

The Internet Control Messaging Protocol (ICMP) is an extension to the Internet
Protocol (IP) as defined in the Request For Comments (RFC) \#792 and supports
network packets containing control, error, and informational messages. ICMP is
used by such network applications as the ping utility, which can determine the
availability of a network host or device. Examples of some error messages
returned by ICMP which are useful to both network hosts and devices such as
routers, include *Destination Unreachable* and *Time Exceeded*.

## Daemons

Daemons are special system applications which typically execute continuously
in the background and await requests for the functions they provide from other
applications. Many daemons are network-centric; that is, a large number of
daemons executing in the background on an Ubuntu system may provide
network-related functionality. Some examples of such network daemons include
the *Hyper Text Transport Protocol Daemon* (httpd), which provides web server
functionality; the *Secure SHell Daemon* (sshd), which provides secure remote
login shell and file transfer capabilities; and the *Internet Message Access
Protocol Daemon* (imapd), which provides E-Mail services.

## Resources {#tcpip-resources}

-   There are man pages for [TCP] and [IP] that contain more
    useful information.

-   Also, see the [TCP/IP Tutorial and Technical Overview] IBM Redbook.

-   Another resource is O'Reilly's [TCP/IP Network Administration].

# Dynamic Host Configuration Protocol (DHCP) {#dhcp}

The Dynamic Host Configuration Protocol (DHCP) is a network service that
enables host computers to be automatically assigned settings from a server as
opposed to manually configuring each network host. Computers configured to be
DHCP clients have no control over the settings they receive from the DHCP
server, and the configuration is transparent to the computer's user.

The most common settings provided by a DHCP server to DHCP clients include:

-   IP address and netmask

-   IP address of the default-gateway to use

-   IP adresses of the DNS servers to use

However, a DHCP server can also supply configuration properties such as:

-   Host Name

-   Domain Name

-   Time Server

-   Print Server

The advantage of using DHCP is that changes to the network, for example a
change in the address of the DNS server, need only be changed at the DHCP
server, and all network hosts will be reconfigured the next time their DHCP
clients poll the DHCP server. As an added advantage, it is also easier to
integrate new computers into the network, as there is no need to check for the
availability of an IP address. Conflicts in IP address allocation are also
reduced.

A DHCP server can provide configuration settings using the following methods:

Manual allocation (MAC address)
:   This method entails using DHCP to identify the unique hardware address of
    each network card connected to the network and then continually supplying
    a constant configuration each time the DHCP client makes a request to the
    DHCP server using that network device. This ensures that a particular
    address is assigned automatically to that network card, based on it's
    MAC address.

Dynamic allocation (address pool)
:   In this method, the DHCP server will assign an IP address from a pool of
    addresses (sometimes also called a range or scope) for a period of time or
    lease, that is configured on the server or until the client informs the
    server that it doesn't need the address anymore. This way, the clients
    will be receiving their configuration properties dynamically and on a
    "first come, first served" basis. When a DHCP client is no longer on the
    network for a specified period, the configuration is expired and released
    back to the address pool for use by other DHCP Clients. This way, an
    address can be leased or used for a period of time. After this period, the
    client has to renegociate the lease with the server to maintain use of
    the address.

Automatic allocation
:   Using this method, the DHCP automatically assigns an IP address
    permanently to a device, selecting it from a pool of available addresses.
    Usually DHCP is used to assign a temporary address to a client, but a DHCP
    server can allow an infinite lease time.

The last two methods can be considered "automatic" because in each case the
DHCP server assigns an address with no extra intervention needed. The only
difference between them is in how long the IP address is leased, in other
words whether a client's address varies over time. Ubuntu is shipped with both
DHCP server and client. The server is dhcpd (dynamic host configuration
protocol daemon). The client provided with Ubuntu is dhclient and should be
installed on all computers required to be automatically configured. Both
programs are easy to install and configure and will be automatically started
at system boot.

## Installation {#dhcp-installation}

At a terminal prompt, enter the following command to install dhcpd:

    sudo apt install isc-dhcp-server

You will probably need to change the default configuration by editing
/etc/dhcp/dhcpd.conf to suit your needs and particular configuration.

You also may need to edit /etc/default/isc-dhcp-server to specify the
interfaces dhcpd should listen to.

NOTE: dhcpd's messages are being sent to syslog. Look there for diagnostics
messages.

## Configuration {#dhcp-configuration}

The error message the installation ends with might be a little confusing, but
the following steps will help you configure the service:

Most commonly, what you want to do is assign an IP address randomly. This can
be done with settings as follows:

    # minimal sample /etc/dhcp/dhcpd.conf
    default-lease-time 600;
    max-lease-time 7200;

    subnet 192.168.1.0 netmask 255.255.255.0 {
     range 192.168.1.150 192.168.1.200;
     option routers 192.168.1.254;
     option domain-name-servers 192.168.1.1, 192.168.1.2;
     option domain-name "mydomain.example";
    } 

This will result in the DHCP server giving clients an IP address from the
range 192.168.1.150-192.168.1.200. It will lease an IP address for 600 seconds
if the client doesn't ask for a specific time frame. Otherwise the maximum
(allowed) lease will be 7200 seconds. The server will also "advise" the client
to use 192.168.1.254 as the default-gateway and 192.168.1.1 and 192.168.1.2 as
its DNS servers.

After changing the config file you have to restart the dhcpd:

    sudo systemctl restart isc-dhcp-server.service

## References {#dhcp-references}

-   The [dhcp3-server Ubuntu Wiki] page has more information.

-   For more `/etc/dhcp/dhcpd.conf` options see the [dhcpd.conf man page].

-   [ISC dhcp-server]

# Time Synchronisation with NTP {#NTP}

NTP is a TCP/IP protocol for synchronising time over a network. Basically a
client requests the current time from a server, and uses it to set its own
clock.

Behind this simple description, there is a lot of complexity - there are tiers
of NTP servers, with the tier one NTP servers connected to atomic clocks, and
tier two and three servers spreading the load of actually handling requests
across the Internet. Also the client software is a lot more complex than you
might think - it has to factor out communication delays, and adjust the time
in a way that does not upset all the other processes that run on the server.
But luckily all that complexity is hidden from you!

Ubuntu uses ntpdate and ntpd.

## timedatectl

In recent Ubuntu releases *timedatectl* replaces *ntpdate*. By default
*timedatectl* syncs the time once on boot and later on uses socket activation
to recheck once network connections become active.

If *ntpdate / ntp* is installed *timedatectl* steps back to let you keep your
old setup. That shall ensure that no two time syncing services are fighting
and also to retain any kind of old behaviour/config that you had through an
upgrade. But it also implies that on an upgrade from a former release
ntp/ntpdate might still be installed and therefore renders the new systemd
based services disabled.

## timesyncd

In recent Ubuntu releases *timesyncd* replaces the client portion of *ntpd*.
By default *timesyncd* regularly checks and keeps the time in sync. It also
stores time updates locally, so that after reboots monotonically advances if
applicable.

The current status of time and time configuration via *timedatectl* and
*timesyncd* can be checked with *timedatectl status*.

    timedatectl status
          Local time: Fri 2016-04-29 06:32:57 UTC
      Universal time: Fri 2016-04-29 06:32:57 UTC
            RTC time: Fri 2016-04-29 07:44:02
           Time zone: Etc/UTC (UTC, +0000)
     Network time on: yes
    NTP synchronized: no
     RTC in local TZ: no

If NTP is installed and replaces the activity of *timedatectl* the line "NTP
synchronized" is set to yes.

The nameserver to fetch time for *timedatectl* and *timesyncd* from can be
specified in /etc/systemd/timesyncd.conf and with flexible additional config
files in /etc/systemd/timesyncd.conf.d/.

## ntpdate

*ntpdate* is considered deprecated in favour of *timedatectl* and thereby no
more installed by default. If installed it will run once at boot time to set
up your time according to Ubuntu's NTP server. Later on anytime a new
interface comes up it retries to update the time - while doing so it will try
to slowly drift time as long as the delta it has to cover isn't too big. That
behaviour can be controlled with the *-B/-b* switches.

    ntpdate ntp.ubuntu.com

## timeservers

By default the systemd based tools request time information at ntp.ubuntu.com.
In classic ntpd based service uses the pool of \[0-3\].ubuntu.pool.ntp.org Of
the pool number 2.ubuntu.pool.ntp.org as well as ntp.ubuntu.com also support
ipv6 if needed. If one needs to force ipv6 there also is ipv6.ntp.ubuntu.com
which is not configured by default.

## ntpd

The ntp daemon ntpd calculates the drift of your system clock and continuously
adjusts it, so there are no large corrections that could lead to inconsistent
logs for instance. The cost is a little processing power and memory, but for a
modern server this is negligible.

## Installation {#ntp-installation}

To install ntpd, from a terminal prompt enter:

    sudo apt install ntp

## Configuration {#timeservers-conf}

Edit `/etc/ntp.conf` to add/remove server lines. By default these servers are
configured:

    # Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
    # on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
    # more information.
    server 0.ubuntu.pool.ntp.org
    server 1.ubuntu.pool.ntp.org
    server 2.ubuntu.pool.ntp.org
    server 3.ubuntu.pool.ntp.org

After changing the config file you have to reload the ntpd:

    sudo systemctl reload ntp.service

## View status {#ntp-status}

Use ntpq to see more info:

    # sudo ntpq -p
         remote           refid      st t when poll reach   delay   offset  jitter
    ==============================================================================
    +stratum2-2.NTP. 129.70.130.70    2 u    5   64  377   68.461  -44.274 110.334
    +ntp2.m-online.n 212.18.1.106     2 u    5   64  377   54.629  -27.318  78.882
    *145.253.66.170  .DCFa.           1 u   10   64  377   83.607  -30.159  68.343
    +stratum2-3.NTP. 129.70.130.70    2 u    5   64  357   68.795  -68.168 104.612
    +europium.canoni 193.79.237.14    2 u   63   64  337   81.534  -67.968  92.792

## PPS Support {#ntp-pps}

Since 16.04 ntp supports PPS discipline which can be used to augment ntp with
local timesources for better accuracy. For more details on configuration see
the external pps ressource listed below.

## References {#ntp-references}

-   See the [Ubuntu Time] wiki page for more information.

-   [ntp.org, home of the Network Time Protocol project]

-   [ntp.org faq on configuring PPS]

# Data Plane Development Kit {#DPDK}

The DPDK is a set of libraries and drivers for fast packet processing and runs
mostly in Linux userland. It is a set of libraries that provide the so called
"Environment Abstraction Layer" (EAL). The EAL hides the details of the
environment and provides a standard programming interface. Common use cases
are around special solutions for instance network function virtualization and
advanced high-throughput network switching. The DPDK uses a run-to-completion
model for fast data plane performance and accesses devices via polling to
eliminate the latency of interrupt processing at the tradeoff of higher cpu
consumption. It was designed to run on any processors. The first supported CPU
was Intel x86 and it is now extended to IBM Power 8, EZchip TILE-Gx and ARM.

Ubuntu currently supports DPDK version 2.2 and provides some infrastructure to
ease its usability.

## Prerequisites {#dpdk-prerequisites}

This package is currently compiled for the lowest possible CPU requirements.
Which still requires at least SSE3 to be supported by the CPU.

The list of upstream DPDK supported network cards can be found at [supported
NICs]. But a lot of those are disabled by default in the upstream Project as
they are not yet in a stable state. The subset of network cards that DPDK has
enabled in the package as available in Ubuntu 16.04 is:

Intel

-   [e1000] (82540, 82545, 82546)

-   [e1000e] (82571..82574, 82583, ICH8..ICH10, PCH..PCH2)

-   [igb][e1000e] (82575..82576, 82580, I210, I211, I350, I354, DH89xx)

-   [ixgbe] (82598..82599, X540, X550)

-   [i40e] (X710, XL710, X722)

-   [fm10k] (FM10420)

Chelsio

-   [cxgbe] (Terminator 5)

Cisco

-   [enic] (UCS Virtual Interface Card)

Paravirtualization

-   [virtio-net] (QEMU)

-   [vmxnet3]

Others

-   [af\_packet] (Linux AF\_PACKET socket)

-   [ring] (memory)

On top it experimentally enables the following two PMD drivers as they
represent (virtual) devices that are very accessible to end users.

Paravirtualization

-   [xenvirt] (Xen)

Others

-   [pcap] (file or kernel driver)

Cards have to be unassigned from their kernel driver and instead be assigned
to uio\_pci\_generic of vfio-pci. uio\_pci\_generic is older and usually
getting to work more easily.

The newer vfio-pci requires that you activate the following kernel parameters
to enable iommu.

    iommu=pt intel_iommu=on
              

On top for vfio-pci you then have to configure and assign the iommu groups
accordingly.

Note: In virtio based environment it is enough to "unassign" devices from the
kernel driver. Without that DPDK will reject to use the device to avoid issues
with kernel and DPDK working on the device at the same time. Since DPDK can
work directly on virtio devices it is not required to assign e.g.
uio\_pci\_generic to those devices.

Manual configuration and status checks can be done via sysfs or with the tool
dpdk\_nic\_bind

    dpdk_nic_bind --help

    Usage:
    ------

         dpdk_nic_bind [options] DEVICE1 DEVICE2 ....

         where DEVICE1, DEVICE2 etc, are specified via PCI "domain:bus:slot.func" syntax
         or "bus:slot.func" syntax. For devices bound to Linux kernel drivers, they may
         also be referred to by Linux interface name e.g. eth0, eth1, em0, em1, etc.

         Options:
             --help, --usage:
             Display usage information and quit

         -s, --status:
                 Print the current status of all known network interfaces.
                 For each device, it displays the PCI domain, bus, slot and function,
                 along with a text description of the device. Depending upon whether the
                 device is being used by a kernel driver, the igb_uio driver, or no
                 driver, other relevant information will be displayed:
                 * the Linux interface name e.g. if=eth0
                 * the driver being used e.g. drv=igb_uio
                 * any suitable drivers not currently using that device
                     e.g. unused=igb_uio
             NOTE: if this flag is passed along with a bind/unbind option, the status
             display will always occur after the other operations have taken place.

         -b driver, --bind=driver:
                 Select the driver to use or "none" to unbind the device

             -u, --unbind:
             Unbind a device (Equivalent to "-b none")

         --force:
                 By default, devices which are used by Linux - as indicated by having
                 routes in the routing table - cannot be modified. Using the --force
                 flag overrides this behavior, allowing active links to be forcibly
                 unbound.
                 WARNING: This can lead to loss of network connection and should be used
                 with caution.

         Examples:
         ---------

         To display current device status:
                 dpdk_nic_bind --status

         To bind eth1 from the current driver and move to use igb_uio
                 dpdk_nic_bind --bind=igb_uio eth1

         To unbind 0000:01:00.0 from using any driver
                 dpdk_nic_bind -u 0000:01:00.0

         To bind 0000:02:00.0 and 0000:02:00.1 to the ixgbe kernel driver
                 dpdk_nic_bind -b ixgbe 02:00.0 02:00.

## DPDK Device configuration {#dpdk-config-dev}

The package *dpdk* provides init scripts that ease configuration of device
assignment and huge pages. It also makes them persistent accross reboots.

The following is an example of the file /etc/dpdk/interfaces configuring two
ports of a network card. One with uio\_pci\_generic and the other one with
vfio-pci

    # <bus>         Currently only "pci" is supported
    # <id>          Device ID on the specified bus
    # <driver>      Driver to bind against (vfio-pci or uio_pci_generic)
    #
    # Be aware that the two DPDK compatible drivers uio_pci_generic and vfio-pci are
    # part of linux-image-extra-<VERSION> package.
    # This package is not always installed by default - for example in cloud-images.
    # So please install it in case you run into missing module issues.
    #
    # <bus> <id>     <driver>
    pci 0000:04:00.0 uio_pci_generic
    pci 0000:04:00.1 vfio-pci
              

Cards are identified by their PCI-ID. If you are unsure you might use the tool
dpdk\_nic\_bind to show the current available devices and the drivers they are
assigned to.

    dpdk_nic_bind --status

    Network devices using DPDK-compatible driver
    ============================================
    0000:04:00.0 'Ethernet Controller 10-Gigabit X540-AT2' drv=uio_pci_generic unused=ixgbe

    Network devices using kernel driver
    ===================================
    0000:02:00.0 'NetXtreme BCM5719 Gigabit Ethernet PCIe' if=eth0 drv=tg3 unused=uio_pci_generic *Active*
    0000:02:00.1 'NetXtreme BCM5719 Gigabit Ethernet PCIe' if=eth1 drv=tg3 unused=uio_pci_generic 
    0000:02:00.2 'NetXtreme BCM5719 Gigabit Ethernet PCIe' if=eth2 drv=tg3 unused=uio_pci_generic 
    0000:02:00.3 'NetXtreme BCM5719 Gigabit Ethernet PCIe' if=eth3 drv=tg3 unused=uio_pci_generic 
    0000:04:00.1 'Ethernet Controller 10-Gigabit X540-AT2' if=eth5 drv=ixgbe unused=uio_pci_generic 

    Other network devices
    =====================
    <none>

              

## DPDK HugePage configuration {#dpdk-config-hp}

DPDK makes heavy use of huge pages to eliminate pressure on the TLB. Therefore
hugepages have to be configured in your system.

The *dpdk* package has a config file and scripts that try to ease hugepage
configuration for DPDK in the form of */etc/dpdk/dpdk.conf*. If you have more
consumers of hugepages than just DPDK in your system or very special
requirements how your hugepages are going to be set up you likely want to
allocate/control them by yourself. If not this can be a great simplification
to get DPDK configured for your needs.

Here an example configuring 1024 Hugepages of 2M each and 4 1G pages.

    NR_2M_PAGES=1024
    NR_1G_PAGES=4
              

As shown this supports configuring 2M and the larger 1G hugepages (or a mix of
both). It will make sure there are proper hugetlbfs mountpoints for DPDK to
find both sizes no matter what your default huge page size is. The config file
itself holds more details on certain corner cases and a few hints if you want
to allocate hugepages manually via a kernel parameter.

It depends on your needs which size you want - 1G pages are certainly more
effective regarding TLB pressure. But there were reports of them fragmenting
inside the DPDK memory alloactions. Also it can be harder to grab enough free
space to set up a certain amount of 1G pages later in the lifecycle of a
system.

## Compile DPDK Applications {#dpdk-apps}

Currently there are not a lot consumers of the DPDK library that are stable
and released. OpenVswitch-DPDK being an exception to that (see below), but in
general it is very likely that you might want / have to compile an app against
the library.

You will often find guides that tell you to fetch the DPDK sources, build them
to your needs and eventually build your application based on DPDK by setting
values RTE\_\* for the build system. Since Ubunutu provides an already
compiled DPDK for you can can skip all that. To simplify setting the proper
variables you can source the file /usr/share/dpdk/dpdk-sdk-env.sh before
building your application. Here an excerpt building the l2fwd example
application delivered with the dpdk-doc package.

    sudo apt-get install dpdk-dev libdpdk-dev
    . /usr/share/dpdk/dpdk-sdk-env.sh
    make -C /usr/share/dpdk/examples/l2fwd
              

Depending on what you build it might be a good addition to install all of DPDK
build dependencies before the make.

    sudo apt-get install build-dep dpdk
              

## OpenVswitch-DPDK {#dpdk-openvswitch}

Being a library it doesn't do a lot on its own, so it depends on emerging
projects making use of it. One consumer of the library that already is bundled
in the Ubuntu 16.04 release is OpenVswitch with DPDK support in the package
openvswitch-switch-dpdk.

Here an example how to install and configure a basic OpenVswitch using DPDK
for later use via libvirt/qemu-kvm.

    sudo apt-get install openvswitch-switch-dpdk
    sudo update-alternatives --set ovs-vswitchd /usr/lib/openvswitch-switch-dpdk/ovs-vswitchd-dpdk
    echo "DPDK_OPTS='--dpdk -c 0x1 -n 4 -m 2048 --vhost-owner libvirt-qemu:kvm --vhost-perm 0664'" | sudo tee -a /etc/default/openvswitch-switch
    sudo service openvswitch-switch restart
              

Please remember that you have to assign devices to DPDK compatible drivers
(see above) before restarting.

The section *--vhost-owner libvirt-qemu:kvm --vhost-perm 0664* will set
vhost\_user ports up with owner/permissions to be compatible with Ubuntus way
of running qemu-kvm/libvirt with reduced privileges for more security.

Please note that the section *-m 2048* is the most basic numa setup for a
single socket system. If you have multiple sockets you might want to define
how to split your memory among them, for example *-m 1024, 1024*. Please be
aware that DPDK will try to work only with local memory to the network cards
it works with (for performance reasons). That said if you have multiple nodes,
but all network cards on one, you should consider spreading your cards. If not
at least allocate your memory to the node where the cards reside, for example
in a two node all to node \#2: *-m 0, 2048*. You can use the tool *lstopo*
from the package *hwloc-nox* to see on which socket your cards are located.

The OpenVswitch you now started supports all port types OpenVswitch usually
does, plus DPDK port types. Here an example how to create a bridge and -
instead of a normal external port - add an external DPDK port to it.

    ovs-vsctl add-br ovsdpdkbr0 -- set bridge ovsdpdkbr0 datapath_type=netdev
    ovs-vsctl add-port ovsdpdkbr0 dpdk0 -- set Interface dpdk0 type=dpdk
              

> **Note**
>
> The enablement of DPDK in Open vSwitch has changed in version 2.6. So for
> users of releases &gt;=16.10, but also for users of the [Ubuntu Cloud
> Archive] &gt;=neutron the enablement has changed compared to that for users
> of Ubuntu 16.04. The options formerly passed via *DPDK\_OPTS* are now
> configured via ovs-vsctl into the Open vSwitch configuration database.
>
> The same example as above would in the new way look like:
>
>     # Enable DPDK
>     ovs-vsctl set Open_vSwitch . "other_config:dpdk-init=true"
>     # run on core 0
>     ovs-vsctl set Open_vSwitch . "other_config:dpdk-lcore-mask=0x1"
>     # Allocate 2G huge pages (not Numa node aware)
>     ovs-vsctl set Open_vSwitch . "other_config:dpdk-alloc-mem=2048"
>     # group/permissions for vhost-user sockets (required to work with libvirt/qemu)
>     ovs-vsctl set Open_vSwitch . \
>        "other_config:dpdk-extra=--vhost-owner libvirt-qemu:kvm --vhost-perm 0666"
>               
>
> Please see the associated upstream documentation and the man page of the
> vswitch configuration as provided by the package for more details:
>
> -   `/usr/share/doc/openvswitch-common/INSTALL.DPDK.md.gz`
>
> -   `/usr/share/doc/openvswitch-common/INSTALL.DPDK-ADVANCED.md.gz`
>
> -   `man ovs-vswitchd.conf.db`
>
## OpenVswitch DPDK to KVM Guests {#dpdk-openvswitch-guest}

If you are not building some sort of SDN switch or NFV on top of DPDK it is
very likely that you want to forward traffic to KVM guests. The good news is,
that with the new qemu/libvirt/dpdk/openvswitch versions in Ubuntu 16.04 this
is no more about manually appending commandline string. This chapter covers a
basic configuration how to connect a KVM guest to a OpenVswitch-DPDK instance.

The Guest has to be backed by shared hugepages for DPDK/vhost\_user to work.
To ensure in general that libvirt/qemu-kvm finds a proper hugepage mountpoint
you can just enable KVM\_HUGEPAGES in /etc/default/qemu-kvm. Afterwards
restart the service to pick up the changed configuration.

    sed -ri -e 's,(KVM_HUGEPAGES=).*,\11,' /etc/default/qemu-kvm
    service qemu-kvm restart
              

To let a guest be backed by hugepages is now also supported via recent
libvirt, just add the following snippet to your virsh xml (or the equivalent
libvirt interface you use). Those xmls can also be used as templates to easily
spawn guests with "uvt-kvm create".

    <numa>
    <cell id='0' cpus='0' memory='6291456' unit='KiB' memAccess='shared'/>
    </numa>
    [...]
    <memoryBacking>
    <hugepages/>
    </memoryBacking>
              

The new and recommended way to get to a KVM guest is using vhost\_user. This
will cause DPDK to create a socket that qemu will connect the guest to. Here
an example how to add such a port to the bridge you created (see above).

    ovs-vsctl add-port ovsdpdkbr0 vhost-user-1 -- set Interface vhost-user-1 type=dpdkvhostuser
              

This will create a vhost\_user socket at /var/run/openvswitch/vhost-user-1

To let libvirt/kvm consume this socket and create a guest virtio network
device for it add a snippet like this to your guest definition as the network
definition.

    <interface type='vhostuser'>
    <source type='unix'
    path='/var/run/openvswitch/vhost-user-1'
    mode='client'/>
    <model type='virtio'/>
    </interface>
              

## DPDK in KVM Guests {#dpdk-in-guest}

If you have no access to DPDK supported network cards you can still work with
DPDK by using its support for virtio. To do so you have to create guests
backed by hugepages (see above).

On top of that there it is required to have at least SSE3. The default CPU
model qemu/libvirt uses is only up to SSE2. So you will have to define a model
that passed the proper feature flag - and of course have a Host system that
supportes it. An example can be found in following snippet to your virsh xml
(or the equivalent virsh interface you use).

    <cpu mode='host-passthrough'>
              

This example is rather offensive and passes all host features. That in turn
makes the guest not very migratable as the target would need all the features
as well. A "softer" way is to just add sse3 to the default model like the
following example.

    <cpu mode='custom' match='exact'>
    <model fallback='allow'>qemu64</model>
    <feature policy='require' name='ssse3'/>
    </cpu>
              

Also virtio nowadays supports multiqueue which DPDK in turn can exploit for
better speed. To modify a normal virtio definition to have multiple queues add
the following to your interface definition. This is about enhancing a normal
virtio nic to have multiple queues, to later on be consumed e.g. by DPDK in
the guest.

    <driver name="vhost" queues="4"/>
              

## Tuning Openvswitch-DPDK {#ovs-dpdk-tuning}

DPDK has plenty of options - in combination with Openvswitch-DPDK the two most
commonly used are:

    ovs-vsctl set Open_vSwitch . other_config:n-dpdk-rxqs=2
    ovs-vsctl set Open_vSwitch . other_config:pmd-cpu-mask=0x6
              

The first select how many rx queues are to be used for each DPDK interface,
while the second controls how many and where to run PMD threads. The example
above will utilize two rx queues and run PMD threads on CPU 1 and 2. See the
referred links to "EAL Command-line Options" and "OpenVswitch DPDK
installation" at the end of this document for more.

As usual with tunings you have to know your system and workload really well -
so please verify any tunings with workloads matching your real use case.

## Support and Troubleshooting {#dpdk-support}

DPDK is a fast evolving project. In any case of a search for support and
further guides it is highly recommended to first check if they apply to the
current version.

-   [DPDK Mailing Lists]

-   For OpenVswitch-DPDK [OpenStack Mailing Lists]

-   Known issues in [DPDK Launchpad Area]

-   Join the IRC channels \#DPDK or \#openvswitch on freenode.

Issues are often due to missing small details in the general setup. Later on,
these missing details cause problems which can be hard to track down to their
root cause. A common case seems to be the "could not open network device dpdk0
(No such device)" issue. This occurs rather late when setting up a port in
Open vSwitch with DPDK. But the root cause most of the time is very early in
the setup and initialization. Here an example how a proper initialization of a
device looks - this can be found in the syslog/journal when starting Open
vSwitch with DPDK enabled.

    ovs-ctl[3560]: EAL: PCI device 0000:04:00.1 on NUMA socket 0
    ovs-ctl[3560]: EAL:   probe driver: 8086:1528 rte_ixgbe_pmd
    ovs-ctl[3560]: EAL:   PCI memory mapped at 0x7f2140000000
    ovs-ctl[3560]: EAL:   PCI memory mapped at 0x7f2140200000
              

If this is missing, either by ignored cards, failed initialization or other
reasons, later on there will be no DPDK device to refer to. Unfortunately the
logging is spread across syslog/journal and the openvswitch log. To allow some
cross checking here an example what can be found in these logs, relative to
the entered command.

    #Note: This log was taken with dpdk 2.2 and openvswitch 2.5
    Captions:
    CMD: that you enter
    SYSLOG: (Inlcuding EAL and OVS Messages)
    OVS-LOG: (Openvswitch messages)

    #PREPARATION
    Bind an interface to DPDK UIO drivers, make Hugepages available, enable DPDK on OVS

    CMD: sudo service openvswitch-switch restart

    SYSLOG:
    2016-01-22T08:58:31.372Z|00003|daemon_unix(monitor)|INFO|pid 3329 died, killed (Terminated), exiting
    2016-01-22T08:58:33.377Z|00002|vlog|INFO|opened log file /var/log/openvswitch/ovs-vswitchd.log
    2016-01-22T08:58:33.381Z|00003|ovs_numa|INFO|Discovered 12 CPU cores on NUMA node 0
    2016-01-22T08:58:33.381Z|00004|ovs_numa|INFO|Discovered 1 NUMA nodes and 12 CPU cores
    2016-01-22T08:58:33.381Z|00005|reconnect|INFO|unix:/var/run/openvswitch/db.sock: connecting...
    2016-01-22T08:58:33.383Z|00006|reconnect|INFO|unix:/var/run/openvswitch/db.sock: connected
    2016-01-22T08:58:33.386Z|00007|bridge|INFO|ovs-vswitchd (Open vSwitch) 2.5.0

    OVS-LOG:
    systemd[1]: Stopping Open vSwitch...
    systemd[1]: Stopped Open vSwitch.
    systemd[1]: Stopping Open vSwitch Internal Unit...
    ovs-ctl[3541]: * Killing ovs-vswitchd (3329)
    ovs-ctl[3541]: * Killing ovsdb-server (3318)
    systemd[1]: Stopped Open vSwitch Internal Unit.
    systemd[1]: Starting Open vSwitch Internal Unit...
    ovs-ctl[3560]: * Starting ovsdb-server
    ovs-vsctl: ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait -- init -- set Open_vSwitch . db-version=7.12.1
    ovs-vsctl: ovs|00001|vsctl|INFO|Called as ovs-vsctl --no-wait set Open_vSwitch . ovs-version=2.5.0 "external-ids:system-id=\"e7c5ba80-bb14-45c1-b8eb-628f3ad03903\"" "system-type=\"Ubuntu\"" "system-version=\"16.04-xenial\""
    ovs-ctl[3560]: * Configuring Open vSwitch system IDs
    ovs-ctl[3560]: 2016-01-22T08:58:31Z|00001|dpdk|INFO|No -vhost_sock_dir provided - defaulting to /var/run/openvswitch
    ovs-vswitchd: ovs|00001|dpdk|INFO|No -vhost_sock_dir provided - defaulting to /var/run/openvswitch
    ovs-ctl[3560]: EAL: Detected lcore 0 as core 0 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 1 as core 1 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 2 as core 2 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 3 as core 3 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 4 as core 4 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 5 as core 5 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 6 as core 0 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 7 as core 1 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 8 as core 2 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 9 as core 3 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 10 as core 4 on socket 0
    ovs-ctl[3560]: EAL: Detected lcore 11 as core 5 on socket 0
    ovs-ctl[3560]: EAL: Support maximum 128 logical core(s) by configuration.
    ovs-ctl[3560]: EAL: Detected 12 lcore(s)
    ovs-ctl[3560]: EAL: VFIO modules not all loaded, skip VFIO support...
    ovs-ctl[3560]: EAL: Setting up physically contiguous memory...
    ovs-ctl[3560]: EAL: Ask a virtual area of 0x100000000 bytes
    ovs-ctl[3560]: EAL: Virtual area found at 0x7f2040000000 (size = 0x100000000)
    ovs-ctl[3560]: EAL: Requesting 4 pages of size 1024MB from socket 0
    ovs-ctl[3560]: EAL: TSC frequency is ~2397202 KHz
    ovs-vswitchd[3592]: EAL: TSC frequency is ~2397202 KHz
    ovs-vswitchd[3592]: EAL: Master lcore 0 is ready (tid=fc6cbb00;cpuset=[0])
    ovs-vswitchd[3592]: EAL: PCI device 0000:04:00.0 on NUMA socket 0
    ovs-vswitchd[3592]: EAL:   probe driver: 8086:1528 rte_ixgbe_pmd
    ovs-vswitchd[3592]: EAL:   Not managed by a supported kernel driver, skipped
    ovs-vswitchd[3592]: EAL: PCI device 0000:04:00.1 on NUMA socket 0
    ovs-vswitchd[3592]: EAL:   probe driver: 8086:1528 rte_ixgbe_pmd
    ovs-vswitchd[3592]: EAL:   PCI memory mapped at 0x7f2140000000
    ovs-vswitchd[3592]: EAL:   PCI memory mapped at 0x7f2140200000
    ovs-ctl[3560]: EAL: Master lcore 0 is ready (tid=fc6cbb00;cpuset=[0])
    ovs-ctl[3560]: EAL: PCI device 0000:04:00.0 on NUMA socket 0
    ovs-ctl[3560]: EAL:   probe driver: 8086:1528 rte_ixgbe_pmd
    ovs-ctl[3560]: EAL:   Not managed by a supported kernel driver, skipped
    ovs-ctl[3560]: EAL: PCI device 0000:04:00.1 on NUMA socket 0
    ovs-ctl[3560]: EAL:   probe driver: 8086:1528 rte_ixgbe_pmd
    ovs-ctl[3560]: EAL:   PCI memory mapped at 0x7f2140000000
    ovs-ctl[3560]: EAL:   PCI memory mapped at 0x7f2140200000
    ovs-vswitchd[3592]: PMD: eth_ixgbe_dev_init(): MAC: 4, PHY: 3
    ovs-vswitchd[3592]: PMD: eth_ixgbe_dev_init(): port 0 vendorID=0x8086 deviceID=0x1528
    ovs-ctl[3560]: PMD: eth_ixgbe_dev_init(): MAC: 4, PHY: 3
    ovs-ctl[3560]: PMD: eth_ixgbe_dev_init(): port 0 vendorID=0x8086 deviceID=0x1528
    ovs-ctl[3560]: Zone 0: name:<RG_MP_log_history>, phys:0x83fffdec0, len:0x2080, virt:0x7f213fffdec0, socket_id:0, flags:0
    ovs-ctl[3560]: Zone 1: name:<MP_log_history>, phys:0x83fd73d40, len:0x28a0c0, virt:0x7f213fd73d40, socket_id:0, flags:0
    ovs-ctl[3560]: Zone 2: name:<rte_eth_dev_data>, phys:0x83fd43380, len:0x2f700, virt:0x7f213fd43380, socket_id:0, flags:0
    ovs-ctl[3560]: * Starting ovs-vswitchd
    ovs-ctl[3560]: * Enabling remote OVSDB managers
    systemd[1]: Started Open vSwitch Internal Unit.
    systemd[1]: Starting Open vSwitch...
    systemd[1]: Started Open vSwitch.


    CMD: sudo ovs-vsctl add-br ovsdpdkbr0 -- set bridge ovsdpdkbr0 datapath_type=netdev

    SYSLOG:
    2016-01-22T08:58:56.344Z|00008|memory|INFO|37256 kB peak resident set size after 24.5 seconds
    2016-01-22T08:58:56.346Z|00009|ofproto_dpif|INFO|netdev@ovs-netdev: Datapath supports recirculation
    2016-01-22T08:58:56.346Z|00010|ofproto_dpif|INFO|netdev@ovs-netdev: MPLS label stack length probed as 3
    2016-01-22T08:58:56.346Z|00011|ofproto_dpif|INFO|netdev@ovs-netdev: Datapath supports unique flow ids
    2016-01-22T08:58:56.346Z|00012|ofproto_dpif|INFO|netdev@ovs-netdev: Datapath does not support ct_state
    2016-01-22T08:58:56.346Z|00013|ofproto_dpif|INFO|netdev@ovs-netdev: Datapath does not support ct_zone
    2016-01-22T08:58:56.346Z|00014|ofproto_dpif|INFO|netdev@ovs-netdev: Datapath does not support ct_mark
    2016-01-22T08:58:56.346Z|00015|ofproto_dpif|INFO|netdev@ovs-netdev: Datapath does not support ct_label
    2016-01-22T08:58:56.360Z|00016|bridge|INFO|bridge ovsdpdkbr0: added interface ovsdpdkbr0 on port 65534
    2016-01-22T08:58:56.361Z|00017|bridge|INFO|bridge ovsdpdkbr0: using datapath ID 00005a4a1ed0a14d
    2016-01-22T08:58:56.361Z|00018|connmgr|INFO|ovsdpdkbr0: added service controller "punix:/var/run/openvswitch/ovsdpdkbr0.mgmt"

    OVS-LOG:
    ovs-vsctl: ovs|00001|vsctl|INFO|Called as ovs-vsctl add-br ovsdpdkbr0 -- set bridge ovsdpdkbr0 datapath_type=netdev
    systemd-udevd[3607]: Could not generate persistent MAC address for ovs-netdev: No such file or directory
    kernel: [50165.886554] device ovs-netdev entered promiscuous mode
    kernel: [50165.901261] device ovsdpdkbr0 entered promiscuous mode


    CMD: sudo ovs-vsctl add-port ovsdpdkbr0 dpdk0 -- set Interface dpdk0 type=dpdk

    SYSLOG:
    2016-01-22T08:59:06.369Z|00019|memory|INFO|peak resident set size grew 155% in last 10.0 seconds, from 37256 kB to 95008 kB
    2016-01-22T08:59:06.369Z|00020|memory|INFO|handlers:4 ports:1 revalidators:2 rules:5
    2016-01-22T08:59:30.989Z|00021|dpdk|INFO|Port 0: 8c:dc:d4:b3:6d:e9
    2016-01-22T08:59:31.520Z|00022|dpdk|INFO|Port 0: 8c:dc:d4:b3:6d:e9
    2016-01-22T08:59:31.521Z|00023|dpif_netdev|INFO|Created 1 pmd threads on numa node 0
    2016-01-22T08:59:31.522Z|00001|dpif_netdev(pmd16)|INFO|Core 0 processing port 'dpdk0'
    2016-01-22T08:59:31.522Z|00024|bridge|INFO|bridge ovsdpdkbr0: added interface dpdk0 on port 1
    2016-01-22T08:59:31.522Z|00025|bridge|INFO|bridge ovsdpdkbr0: using datapath ID 00008cdcd4b36de9
    2016-01-22T08:59:31.523Z|00002|dpif_netdev(pmd16)|INFO|Core 0 processing port 'dpdk0'

    OVS-LOG:
    ovs-vsctl: ovs|00001|vsctl|INFO|Called as ovs-vsctl add-port ovsdpdkbr0 dpdk0 -- set Interface dpdk0 type=dpdk
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a79ebc0 hw_ring=0x7f211a7a6c00 dma_addr=0x81a7a6c00
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_rx_queue_setup(): sw_ring=0x7f211a78a6c0 sw_sc_ring=0x7f211a786580 hw_ring=0x7f211a78e800 dma_addr=0x81a78e800
    ovs-vswitchd[3595]: PMD: ixgbe_set_rx_function(): Vector rx enabled, please make sure RX burst size no less than 4 (port=0).
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a79ebc0 hw_ring=0x7f211a7a6c00 dma_addr=0x81a7a6c00
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a76e4c0 hw_ring=0x7f211a776500 dma_addr=0x81a776500
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a756440 hw_ring=0x7f211a75e480 dma_addr=0x81a75e480
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a73e3c0 hw_ring=0x7f211a746400 dma_addr=0x81a746400
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a726340 hw_ring=0x7f211a72e380 dma_addr=0x81a72e380
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a70e2c0 hw_ring=0x7f211a716300 dma_addr=0x81a716300
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a6f6240 hw_ring=0x7f211a6fe280 dma_addr=0x81a6fe280
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a6de1c0 hw_ring=0x7f211a6e6200 dma_addr=0x81a6e6200
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a6c6140 hw_ring=0x7f211a6ce180 dma_addr=0x81a6ce180
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a6ae0c0 hw_ring=0x7f211a6b6100 dma_addr=0x81a6b6100
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a696040 hw_ring=0x7f211a69e080 dma_addr=0x81a69e080
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a67dfc0 hw_ring=0x7f211a686000 dma_addr=0x81a686000
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_tx_queue_setup(): sw_ring=0x7f211a665e40 hw_ring=0x7f211a66de80 dma_addr=0x81a66de80
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Using simple tx code path
    ovs-vswitchd[3595]: PMD: ixgbe_set_tx_function(): Vector tx enabled.
    ovs-vswitchd[3595]: PMD: ixgbe_dev_rx_queue_setup(): sw_ring=0x7f211a78a6c0 sw_sc_ring=0x7f211a786580 hw_ring=0x7f211a78e800 dma_addr=0x81a78e800
    ovs-vswitchd[3595]: PMD: ixgbe_set_rx_function(): Vector rx enabled, please make sure RX burst size no less than 4 (port=0).


    CMD: sudo ovs-vsctl add-port ovsdpdkbr0 vhost-user-1 -- set Interface vhost-user-1 type=dpdkvhostuser

    OVS-LOG:
    2016-01-22T09:00:35.145Z|00026|dpdk|INFO|Socket /var/run/openvswitch/vhost-user-1 created for vhost-user port vhost-user-1
    2016-01-22T09:00:35.145Z|00003|dpif_netdev(pmd16)|INFO|Core 0 processing port 'dpdk0'
    2016-01-22T09:00:35.145Z|00004|dpif_netdev(pmd16)|INFO|Core 0 processing port 'vhost-user-1'
    2016-01-22T09:00:35.145Z|00027|bridge|INFO|bridge ovsdpdkbr0: added interface vhost-user-1 on port 2

    SYSLOG:
    ovs-vsctl: ovs|00001|vsctl|INFO|Called as ovs-vsctl add-port ovsdpdkbr0 vhost-user-1 -- set Interface vhost-user-1 type=dpdkvhostuser
    ovs-vswitchd[3595]: VHOST_CONFIG: socket created, fd:46
    ovs-vswitchd[3595]: VHOST_CONFIG: bind to /var/run/openvswitch/vhost-user-1

    Eventually we can see the poll thread in top
      PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
     3595 root      10 -10 4975344 103936   9916 S 100.0  0.3  33:13.56 ovs-vswitchd
              

## Resources {#dpdk-references}

-   [DPDK Documentation]

-   [Release Notes matching the version packages in Ubuntu 16.04]

-   [Linux DPDK User Getting Started]

-   [EAL Command-line Options]

-   [DPDK Api Documentation]

-   [OpenVswitch DPDK installation]

-   [Wikipedias definition of DPDK]

  [Ubuntu Wiki Network page]: https://help.ubuntu.com/community/Network
  [resolvconf man page]: http://manpages.ubuntu.com/manpages/man8/resolvconf.8.html
  [interfaces man page]: http://manpages.ubuntu.com/manpages/man5/interfaces.5.html
  [dhclient man page]: http://manpages.ubuntu.com/manpages/man8/dhclient.8.html
  [resolver man page]: http://manpages.ubuntu.com/manpages/man5/resolver.5.html
  [Linux Network Administrator's Guide]: http://oreilly.com/catalog/linag2/book/ch06.html
  [brctl man page]: http://manpages.ubuntu.com/manpages/man8/brctl.8.html
  [Networking-Bridge]: http://www.linuxfoundation.org/collaborate/workgroups/networking/bridge
  [TCP]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man7/tcp.7.html
  [IP]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/man7/ip.7.html
  [TCP/IP Tutorial and Technical Overview]: http://www.redbooks.ibm.com/abstracts/gg243376.html
  [TCP/IP Network Administration]: http://oreilly.com/catalog/9780596002978/
  [dhcp3-server Ubuntu Wiki]: https://help.ubuntu.com/community/dhcp3-server
  [dhcpd.conf man page]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/dhcpd.conf.5.html
  [ISC dhcp-server]: http://www.isc.org/software/dhcp
  [Ubuntu Time]: https://help.ubuntu.com/community/UbuntuTime
  [ntp.org, home of the Network Time Protocol project]: http://www.ntp.org/
  [ntp.org faq on configuring PPS]: http://www.ntp.org/ntpfaq/NTP-s-config-adv.htm#S-CONFIG-ADV-PPS
  [supported NICs]: http://dpdk.org/doc/nics
  [e1000]: http://dpdk.org/doc/guides/nics/e1000em.html
  [e1000e]: http://dpdk.org/browse/dpdk/tree/drivers/net/e1000/
  [ixgbe]: http://dpdk.org/doc/guides/nics/ixgbe.html
  [i40e]: http://dpdk.org/browse/dpdk/tree/drivers/net/i40e/
  [fm10k]: http://dpdk.org/doc/guides/nics/fm10k.html
  [cxgbe]: http://dpdk.org/doc/guides/nics/cxgbe.html
  [enic]: http://dpdk.org/browse/dpdk/tree/drivers/net/enic
  [virtio-net]: http://dpdk.org/doc/guides/nics/virtio.html
  [vmxnet3]: http://dpdk.org/doc/guides/nics/vmxnet3.html
  [af\_packet]: http://dpdk.org/browse/dpdk/tree/drivers/net/af_packet
  [ring]: http://dpdk.org/doc/guides/nics/pcap_ring.html#rings-based-pmd
  [xenvirt]: http://dpdk.org/doc/guides/xen/pkt_switch.html#xen-pmd-frontend-prerequisites
  [pcap]: http://dpdk.org/doc/guides/nics/pcap_ring.html#libpcap-based-pmd
  [Ubuntu Cloud Archive]: https://wiki.ubuntu.com/OpenStack/CloudArchive
  [DPDK Mailing Lists]: http://dpdk.org/ml
  [OpenStack Mailing Lists]: http://openvswitch.org/mlists
  [DPDK Launchpad Area]: https://bugs.launchpad.net/ubuntu/+source/dpdk
  [DPDK Documentation]: http://dpdk.org/doc
  [Release Notes matching the version packages in Ubuntu 16.04]: http://dpdk.org/doc/guides/rel_notes/release_2_2.html
  [Linux DPDK User Getting Started]: http://dpdk.org/doc/guides/linux_gsg/index.html
  [EAL Command-line Options]: http://dpdk.org/doc/guides/testpmd_app_ug/run_app.html
  [DPDK Api Documentation]: http://dpdk.org/doc/api/
  [OpenVswitch DPDK installation]: https://github.com/openvswitch/ovs/blob/branch-2.5/INSTALL.DPDK.md
  [Wikipedias definition of DPDK]: https://en.wikipedia.org/wiki/Data_Plane_Development_Kit
