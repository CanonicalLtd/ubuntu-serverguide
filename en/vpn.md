# VPN

OpenVPN is a Virtual Private Networking (VPN) solution provided in the Ubuntu
Repositories. It is flexible, reliable and secure. It belongs to the family of
SSL/TLS VPN stacks (different from IPSec VPNs). This chapter will cover
installing and configuring OpenVPN to create a VPN.

## OpenVPN
If you want more than just pre-shared keys OpenVPN makes it easy to setup and
use a Public Key Infrastructure (PKI) to use SSL/TLS certificates for
authentication and key exchange between the VPN server and clients. OpenVPN
can be used in a routed or bridged VPN mode and can be configured to use
either UDP or TCP. The port number can be configured as well, but port 1194 is
the official one. And it is only using that single port for all communication.
VPN client implementations are available for almost anything including all
Linux distributions, OS X, Windows and OpenWRT based WLAN routers.

### Server Installation 
To install openvpn in a terminal enter:

```bash
sudo apt install openvpn easy-rsa
```

### Public Key Infrastructure Setup 
The first step in building an OpenVPN configuration is to establish a PKI
(public key infrastructure). The PKI consists of:

-   a separate certificate (also known as a public key) and private key for
    the server and each client, and

-   a master Certificate Authority (CA) certificate and key which is used to
    sign each of the server and client certificates.

OpenVPN supports bidirectional authentication based on certificates, meaning
that the client must authenticate the server certificate and the server must
authenticate the client certificate before mutual trust is established.

Both server and client will authenticate the other by first verifying that the
presented certificate was signed by the master certificate authority (CA), and
then by testing information in the now-authenticated certificate header, such
as the certificate common name or certificate type (client or server).

#### Certificate Authority Setup 
To setup your own Certificate Authority (CA) and generating certificates and
keys for an OpenVPN server and multiple clients first copy the `easy-rsa`
directory to `/etc/openvpn`. This will ensure that any changes to the scripts
will not be lost when the package is updated. From a terminal change to user
root and:

```bash
mkdir /etc/openvpn/easy-rsa/
cp -r /usr/share/easy-rsa/* /etc/openvpn/easy-rsa/
```

Next, edit `/etc/openvpn/easy-rsa/vars` adjusting the following to your
environment:

```bash
export KEY_COUNTRY="US"
export KEY_PROVINCE="NC"
export KEY_CITY="Winston-Salem"
export KEY_ORG="Example Company"
export KEY_EMAIL="steve@example.com"
export KEY_CN=MyVPN
export KEY_NAME=MyVPN
export KEY_OU=MyVPN
```

Enter the following to generate the master Certificate Authority (CA)
certificate and key:

```bash
cd /etc/openvpn/easy-rsa/
source vars
./clean-all
./build-ca
```

#### Server Certificates 
Next, we will generate a certificate and private key for the server:

```bash
./build-key-server myservername
```

As in the previous step, most parameters can be defaulted. Two other queries
require positive responses, "Sign the certificate? \[y/n\]" and "1 out of 1
certificate requests certified, commit? \[y/n\]".

Diffie Hellman parameters must be generated for the OpenVPN server:

```bash
./build-dh
```

All certificates and keys have been generated in the subdirectory keys/.
Common practice is to copy them to /etc/openvpn/:

```bash
cd keys/
cp myservername.crt myservername.key ca.crt dh2048.pem /etc/openvpn/
```

#### Client Certificates 
The VPN client will also need a certificate to authenticate itself to the
server. Usually you create a different certificate for each client. To create
the certificate, enter the following in a terminal while being user root:

```bash
cd /etc/openvpn/easy-rsa/
source vars
./build-key client1
```

Copy the following files to the client using a secure method:

-   /etc/openvpn/ca.crt

-   /etc/openvpn/easy-rsa/keys/client1.crt

-   /etc/openvpn/easy-rsa/keys/client1.key

As the client certificates and keys are only required on the client machine,
you should remove them from the server.

### Simple Server Configuration 
Along with your OpenVPN installation you got these sample config files (and
many more if if you check):

```bash
root@server:/# ls -l /usr/share/doc/openvpn/examples/sample-config-files/
total 68
-rw-r--r-- 1 root root 3427 2011-07-04 15:09 client.conf
-rw-r--r-- 1 root root 4141 2011-07-04 15:09 server.conf.gz
```

Start with copying and unpacking server.conf.gz to /etc/openvpn/server.conf.

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf.gz /etc/openvpn/
sudo gzip -d /etc/openvpn/server.conf.gz
```

Edit `/etc/openvpn/server.conf` to make sure the following lines are pointing
to the certificates and keys you created in the section above.

```bash
ca ca.crt
cert myservername.crt
key myservername.key
dh dh2048.pem
```

Edit `/etc/sysctl.conf` and uncomment the following line to enable IP
forwarding.

```bash
#net.ipv4.ip_forward=1
```

Then reload sysctl.

```bash
sudo sysctl -p /etc/sysctl.conf
```

That is the minimum you have to configure to get a working OpenVPN server. You
can use all the default settings in the sample server.conf file. Now start the
server. You will find logging and error messages in your via journal. Dependin
on what you look for:

```bash
sudo journalctl -xe
```

If you started a templatized service openvpn@server you can filter for this
particular message source with:

```bash
sudo journalctl --identifier ovpn-server
```

Be aware that the "service openvpn start" is not starting your openvpn you
just defined. Openvpn uses templatized systemd jobs, openvpn@CONFIGFILENAME.
So if for example your configuration file is "server.conf" your service is
called openvpn@server. You can run all kind of service and systemctl commands
like start/stop/enable/disable/preset against a templatized service like
openvpn@server.

```bash
ubuntu@testopenvpn-server:~$ sudo service openvpn@server start
```

```bash
ubuntu@testopenvpn-server:~$ sudo service openvpn@server status
. openvpn@server.service - OpenVPN connection to server
Loaded: loaded (/lib/systemd/system/openvpn@.service; enabled; vendor preset: enabled)
      Active: active (running) since Tue 2016-04-12 08:51:14 UTC; 1s ago
        Docs: man:openvpn(8)
              https://community.openvpn.net/openvpn/wiki/Openvpn23ManPage
              https://community.openvpn.net/openvpn/wiki/HOWTO
     Process: 1573 ExecStart=/usr/sbin/openvpn --daemon ovpn-%i --status /run/openvpn/%i.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/%i.conf --writep
    Main PID: 1575 (openvpn)
       Tasks: 1 (limit: 512)
      CGroup: /system.slice/system-openvpn.slice/openvpn@server.service
              |-1575 /usr/sbin/openvpn --daemon ovpn-server --status /run/openvpn/server.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/server.conf --wr
```

```bash
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: /sbin/ip route add 10.8.0.0/24 via 10.8.0.2
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: UDPv4 link local (bound): [undef]
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: UDPv4 link remote: [undef]
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: MULTI: multi_init called, r=256 v=256
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: IFCONFIG POOL: base=10.8.0.4 size=62, ipv6=0
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: ifconfig_pool_read(), in='client1,10.8.0.4', TODO: IPv6
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: succeeded -> ifconfig_pool_set()
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: IFCONFIG POOL LIST
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: client1,10.8.0.4
Apr 12 08:51:14 testopenvpn-server ovpn-server[1575]: Initialization Sequence Completed
```

You can enable/disable various openvpn services on one system, but you could
also let Ubuntu do the heavy lifting. There is config for AUTOSTART in
/etc/default/openvpn. Allowed values are "all", "none" or space separated list
of names of the VPNs. If empty, "all" is assumed. The VPN name refers to the
VPN configutation file name. i.e. "home" would be /etc/openvpn/home.conf If
you're running systemd, changing this variable will require running "systemctl
daemon-reload" followed by a restart of the openvpn service (if you removed
entries you may have to stop those manually) After "systemctl daemon-reload" a
restart of the "generic" openvon will restart all dependent services that the
generator in /lib/systemd/system-generators/openvpn-generator created for your
conf files when you called daemon-reload.

That is the minimum you have to configure to get a working OpenVPN server. You
can use all the default settings in the sample server.conf file. Now start the
server. You will find logging and error messages in your journal.

Now check if OpenVPN created a tun0 interface:

```bash
root@server:/etc/openvpn# ifconfig tun0
tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00
          inet addr:10.8.0.1  P-t-P:10.8.0.2  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
[...]
```

### Simple Client Configuration 
There are various different OpenVPN client implementations with and without
GUIs. You can read more about clients in a later section. For now we use the
OpenVPN client for Ubuntu which is the same executable as the server. So you
have to install the openvpn package again on the client machine:

```bash
sudo apt install openvpn
```

This time copy the client.conf sample config file to /etc/openvpn/.

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/
```

Copy the client keys and the certificate of the CA you created in the section
above to e.g. /etc/openvpn/ and edit `/etc/openvpn/client.conf` to make sure
the following lines are pointing to those files. If you have the files in
/etc/openvpn/ you can omit the path.

```bash
ca ca.crt
cert client1.crt
key client1.key
```

And you have to at least specify the OpenVPN server name or address. Make sure
the keyword client is in the config. That's what enables client mode.

```bash
client
remote vpnserver.example.com 1194
```

Also, make sure you specify the keyfile names you copied from the server

```bash
ca ca.crt
cert client1.crt
key client1.key
```

Now start the OpenVPN client:

```bash
ubuntu@testopenvpn-client:~$ sudo service  openvpn@client start
ubuntu@testopenvpn-client:~$ sudo service  openvpn@client status
. openvpn@client.service - OpenVPN connection to client
   Loaded: loaded (/lib/systemd/system/openvpn@.service; disabled; vendor preset: enabled)
   Active: active (running) since Tue 2016-04-12 08:50:50 UTC; 3s ago
     Docs: man:openvpn(8)
           https://community.openvpn.net/openvpn/wiki/Openvpn23ManPage
           https://community.openvpn.net/openvpn/wiki/HOWTO
 Process: 1677 ExecStart=/usr/sbin/openvpn --daemon ovpn-%i --status /run/openvpn/%i.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/%i.conf --writep
Main PID: 1679 (openvpn)
   Tasks: 1 (limit: 512)
  CGroup: /system.slice/system-openvpn.slice/openvpn@client.service
          |-1679 /usr/sbin/openvpn --daemon ovpn-client --status /run/openvpn/client.status 10 --cd /etc/openvpn --script-security 2 --config /etc/openvpn/client.conf --wr
```

```bash
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: OPTIONS IMPORT: --ifconfig/up options modified
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: OPTIONS IMPORT: route options modified
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: ROUTE_GATEWAY 192.168.122.1/255.255.255.0 IFACE=eth0 HWADDR=52:54:00:89:ca:89
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: TUN/TAP device tun0 opened
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: TUN/TAP TX queue length set to 100
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: do_ifconfig, tt->ipv6=0, tt->did_ifconfig_ipv6_setup=0
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: /sbin/ip link set dev tun0 up mtu 1500
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: /sbin/ip addr add dev tun0 local 10.8.0.6 peer 10.8.0.5
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: /sbin/ip route add 10.8.0.1/32 via 10.8.0.5
Apr 12 08:50:52 testopenvpn-client ovpn-client[1679]: Initialization Sequence Completed
```

Check if it created a tun0 interface:

```bash
root@client:/etc/openvpn# ifconfig tun0
tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
          inet addr:10.8.0.6  P-t-P:10.8.0.5  Mask:255.255.255.255
          UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
```

Check if you can ping the OpenVPN server:

```bash
root@client:/etc/openvpn# ping 10.8.0.1
PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
64 bytes from 10.8.0.1: icmp_req=1 ttl=64 time=0.920 ms
```

!!! Note: The OpenVPN server always uses the first usable IP address in the client
network and only that IP is pingable. E.g. if you configured a /24 for the
client network mask, the .1 address will be used. The P-t-P address you see
in the ifconfig output above is usually not answering ping requests.

Check out your routes:

```bash
root@client:/etc/openvpn# netstat -rn
Kernel IP routing table
Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
10.8.0.5        0.0.0.0         255.255.255.255 UH        0 0          0 tun0
10.8.0.1        10.8.0.5        255.255.255.255 UGH       0 0          0 tun0
192.168.42.0    0.0.0.0         255.255.255.0   U         0 0          0 eth0
0.0.0.0         192.168.42.1    0.0.0.0         UG        0 0          0 eth0
```

### First trouble shooting 
If the above didn't work for you, check this:

-   Check your journal, e.g. journalctl --identifier ovpn-server
    (for server.conf)

-   Check that you have specified the keyfile names correctly in client.conf
    and server.conf.

-   Can the client connect to the server machine? Maybe a firewall is blocking
    access? Check journal on server.

-   Client and server must use same protocol and port, e.g. UDP port 1194, see
    port and proto config option

-   Client and server must use same config regarding compression, see comp-lzo
    config option

-   Client and server must use same config regarding bridged vs routed mode,
    see server vs server-bridge config option

### Advanced configuration 
#### Advanced routed VPN configuration on server 
The above is a very simple working VPN. The client can access services on the
VPN server machine through an encrypted tunnel. If you want to reach more
servers or anything in other networks, push some routes to the clients. E.g.
if your company's network can be summarized to the network 192.168.0.0/16, you
could push this route to the clients. But you will also have to change the
routing for the way back - your servers need to know a route to the VPN
client-network.

Or you might push a default gateway to all the clients to send all their
internet traffic to the VPN gateway first and from there via the company
firewall into the internet. This section shows you some possible options.

Push routes to the client to allow it to reach other private subnets behind
the server. Remember that these private subnets will also need to know to
route the OpenVPN client address pool (10.8.0.0/24) back to the OpenVPN
server.

```bash
push "route 10.0.0.0 255.0.0.0"
```

If enabled, this directive will configure all clients to redirect their
default network gateway through the VPN, causing all IP traffic such as web
browsing and DNS lookups to go through the VPN (the OpenVPN server machine or
your central firewall may need to NAT the TUN/TAP interface to the internet in
order for this to work properly).

```bash
push "redirect-gateway def1 bypass-dhcp"
```

Configure server mode and supply a VPN subnet for OpenVPN to draw client
addresses from. The server will take 10.8.0.1 for itself, the rest will be
made available to clients. Each client will be able to reach the server on
10.8.0.1. Comment this line out if you are ethernet bridging.

```bash
server 10.8.0.0 255.255.255.0
```

Maintain a record of client to virtual IP address associations in this file.
If OpenVPN goes down or is restarted, reconnecting clients can be assigned the
same virtual IP address from the pool that was previously assigned.

```bash
ifconfig-pool-persist ipp.txt
```

Push DNS servers to the client.

```bash
push "dhcp-option DNS 10.0.0.2"
push "dhcp-option DNS 10.1.0.2"
```

Allow client to client communication.

```bash
client-to-client
```

Enable compression on the VPN link.

```bash
comp-lzo
```

The *keepalive* directive causes ping-like messages to be sent back and forth
over the link so that each side knows when the other side has gone down. Ping
every 1 second, assume that remote peer is down if no ping received during a 3
second time period.

```bash
keepalive 1 3
```

It's a good idea to reduce the OpenVPN daemon's privileges after
initialization.

```bash
user nobody
group nogroup
```

OpenVPN 2.0 includes a feature that allows the OpenVPN server to securely
obtain a username and password from a connecting client, and to use that
information as a basis for authenticating the client. To use this
authentication method, first add the auth-user-pass directive to the client
configuration. It will direct the OpenVPN client to query the user for a
username/password, passing it on to the server over the secure TLS channel.

```bash
# client config!
auth-user-pass
```

This will tell the OpenVPN server to validate the username/password entered by
clients using the login PAM module. Useful if you have centralized
authentication with e.g. Kerberos.

```bash
plugin /usr/lib/openvpn/openvpn-plugin-auth-pam.so login
```

!!! Note: Please read the OpenVPN [hardening security guide] for further security
advice.

#### Advanced bridged VPN configuration on server 
OpenVPN can be setup for either a routed or a bridged VPN mode. Sometimes this
is also referred to as OSI layer-2 versus layer-3 VPN. In a bridged VPN all
layer-2 frames - e.g. all ethernet frames - are sent to the VPN partners and
in a routed VPN only layer-3 packets are sent to VPN partners. In bridged mode
all traffic including traffic which was traditionally LAN-local like local
network broadcasts, DHCP requests, ARP requests etc. are sent to VPN partners
whereas in routed mode this would be filtered.

##### Prepare interface config for bridging on server 
Make sure you have the bridge-utils package installed:

```bash
sudo apt install bridge-utils
```

Before you setup OpenVPN in bridged mode you need to change your interface
configuration. Let's assume your server has an interface eth0 connected to the
internet and an interface eth1 connected to the LAN you want to bridge. Your
/etc/network/interfaces would like this:

```bash
auto eth0
iface eth0 inet static
  address 1.2.3.4
  netmask 255.255.255.248
  default 1.2.3.1
```

```bash
auto eth1
iface eth1 inet static
  address 10.0.0.4
  netmask 255.255.255.0
```

This straight forward interface config needs to be changed into a bridged mode
like where the config of interface eth1 moves to the new br0 interface. Plus
we configure that br0 should bridge interface eth1. We also need to make sure
that interface eth1 is always in promiscuous mode - this tells the interface
to forward all ethernet frames to the IP stack.

```bash
auto eth0
iface eth0 inet static
  address 1.2.3.4
  netmask 255.255.255.248
  default 1.2.3.1
```

```bash
auto eth1
iface eth1 inet manual
  up ip link set $IFACE up promisc on
```

```bash
auto br0
iface br0 inet static
  address 10.0.0.4
  netmask 255.255.255.0
  bridge_ports eth1
```

At this point you need to bring up the bridge. Be prepared that this might not
work as expected and that you will lose remote connectivity. Make sure you can
solve problems having local access.

```bash
sudo ifdown eth1 && sudo ifup -a
```

##### Prepare server config for bridging 
Edit `/etc/openvpn/server.conf` changing the following options to:

```bash
;dev tun
dev tap
up "/etc/openvpn/up.sh br0 eth1"
;server 10.8.0.0 255.255.255.0
server-bridge 10.0.0.4 255.255.255.0 10.0.0.128 10.0.0.254
```

Next, create a helper script to add the *tap* interface to the bridge and to
ensure that eth1 is promiscuous mode. Create `/etc/openvpn/up.sh`:

```bash
#!/bin/sh
```

```bash
BR=$1
ETHDEV=$2
TAPDEV=$3
```

```bash
/sbin/ip link set "$TAPDEV" up
/sbin/ip link set "$ETHDEV" promisc on
/sbin/brctl addif $BR $TAPDEV
```

Then make it executable:

```bash
sudo chmod 755 /etc/openvpn/up.sh
```

After configuring the server, restart openvpn by entering:

```bash
sudo service  openvpn@server restart
```

##### Client Configuration 
First, install openvpn on the client:

```bash
sudo apt install openvpn
```

Then with the server configured and the client certificates copied to the
`/etc/openvpn/` directory, create a client configuration file by copying the
example. In a terminal on the client machine enter:

```bash
sudo cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn
```

Now edit `/etc/openvpn/client.conf` changing the following options:

```bash
dev tap
;dev tun
ca ca.crt
cert client1.crt
key client1.key
```

Finally, restart openvpn:

```bash
sudo service  openvpn@client restart
```

You should now be able to connect to the remote LAN through the VPN.

### Client software implementations 
#### Linux Network-Manager GUI for OpenVPN 
Many Linux distributions including Ubuntu desktop variants come with Network
Manager, a nice GUI to configure your network settings. It also can manage
your VPN connections. Make sure you have package network-manager-openvpn
installed. Here you see that the installation installs all other required
packages as well:

```bash
root@client:~# apt install network-manager-openvpn
Reading package lists... Done
Building dependency tree
Reading state information... Done
The following extra packages will be installed:
  liblzo2-2 libpkcs11-helper1 network-manager-openvpn-gnome openvpn
Suggested packages:
  resolvconf
The following NEW packages will be installed:
  liblzo2-2 libpkcs11-helper1 network-manager-openvpn
  network-manager-openvpn-gnome openvpn
0 upgraded, 5 newly installed, 0 to remove and 631 not upgraded.
Need to get 700 kB of archives.
After this operation, 3,031 kB of additional disk space will be used.
Do you want to continue [Y/n]?
```

To inform network-manager about the new installed packages you will have to
restart it:

```bash
root@client:~# restart network-manager
network-manager start/running, process 3078
```

Open the Network Manager GUI, select the VPN tab and then the 'Add' button.
Select OpenVPN as the VPN type in the opening requester and press 'Create'. In
the next window add the OpenVPN's server name as the 'Gateway', set 'Type' to
'Certificates (TLS)', point 'User Certificate' to your user certificate, 'CA
Certificate' to your CA certificate and 'Private Key' to your private key
file. Use the advanced button to enable compression (e.g. comp-lzo), dev tap,
or other special settings you set on the server. Now try to establish your
VPN.

#### OpenVPN with GUI for Mac OS X: Tunnelblick 
Tunnelblick is an excellent free, open source implementation of a GUI for
OpenVPN for OS X. The project's homepage is at
<http://code.google.com/p/tunnelblick/>. Download the latest OS X installer
from there and install it. Then put your client.ovpn config file together with
the certificates and keys in /Users/username/Library/Application
Support/Tunnelblick/Configurations/ and lauch Tunnelblick from your
Application folder.

```bash
# sample client.ovpn for Tunnelblick
client
remote blue.example.com
port 1194
proto udp
dev tun
dev-type tun
ns-cert-type server
reneg-sec 86400
auth-user-pass
auth-nocache
auth-retry interact
comp-lzo yes
verb 3
ca ca.crt
cert client.crt
key client.key
```

#### OpenVPN with GUI for Win 7 
First download and install the latest [OpenVPN Windows Installer]. OpenVPN
2.3.2 was the latest when this was written. As of this writing, the management
GUI is included with the Windows binary installer.

You need to start the OpenVPN service. Goto Start &gt; Computer &gt; Manage
&gt; Services and Applications &gt; Services. Find the OpenVPN service and
start it. Set it's startup type to automatic. When you start the OpenVPN MI
GUI the first time you need to run it as an administrator. You have to right
click on it and you will see that option.

You will have to write your OpenVPN config in a textfile and place it in
C:\\Program Files\\OpenVPN\\config\\client.ovpn along with the CA certificate.
You could put the user certificate in the user's home directory like in the
follwing example.

```bash
# C:\Program Files\OpenVPN\config\client.ovpn
client
remote server.example.com
port 1194
proto udp
dev tun
dev-type tun
ns-cert-type server
reneg-sec 86400
auth-user-pass
auth-retry interact
comp-lzo yes
verb 3
ca ca.crt
cert "C:\\Users\\username\\My Documents\\openvpn\\client.crt"
key "C:\\Users\\username\\My Documents\\openvpn\\client.key"
management 127.0.0.1 1194
management-hold
management-query-passwords
auth-retry interact
; Set the name of the Windows TAP network interface device here
dev-node MyTAP
```

Note: If you are not using user authentication and/or you want to run the
service without user interaction, comment out the following options:

```bash
auth-user-pass
auth-retry interact
management 127.0.0.1 1194
management-hold
management-query-passwords
```

You may want to set the Windows service to "automatic".

#### OpenVPN for OpenWRT 
OpenWRT is described as a Linux distribution for embedded devices like WLAN
router. There are certain types of WLAN routers who can be flashed to run
OpenWRT. Depending on the available memory on your OpenWRT router you can run
software like OpenVPN and you could for example build a small inexpensive
branch office router with VPN connectivity to the central office. More info on
OpenVPN on OpenWRT is [here]. And here is the OpenWRT project's homepage:
<http://openwrt.org>

Log into your OpenWRT router and install OpenVPN:

```bash
opkg update
opkg install openvpn
```

Check out /etc/config/openvpn and put your client config in there. Copy
certificates and keys to /etc/openvpn/

```bash
config openvpn client1
        option enable 1
        option client 1
#       option dev tap
        option dev tun
        option proto udp
        option ca /etc/openvpn/ca.crt
        option cert /etc/openvpn/client.crt
        option key /etc/openvpn/client.key
        option comp_lzo 1
```

Restart OpenVPN on OpenWRT router to pick up the config

You will have to see if you need to adjust your router's routing and firewall
rules.

### References 
-   See the [OpenVPN] website for additional information.

-   [OpenVPN hardening security guide][hardening security guide]

-   Also, Pakt's [OpenVPN: Building and Integrating Virtual Private Networks]
    is a good resource.

  [hardening security guide]: http://openvpn.net/index.php/open-source/documentation/howto.html#security
  [OpenVPN Windows Installer]: http://www.openvpn.net/index.php/open-source/downloads.html
  [here]: http://wiki.openwrt.org/doc/howto/vpn.overview
  [OpenVPN]: http://openvpn.net/
  [OpenVPN: Building and Integrating Virtual Private Networks]: http://www.packtpub.com/openvpn/book
