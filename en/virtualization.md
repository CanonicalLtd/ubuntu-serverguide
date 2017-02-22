# Virtualization

Virtualization is being adopted in many different environments and situations.
If you are a developer, virtualization can provide you with a contained
environment where you can safely do almost any sort of development safe from
messing up your main working environment. If you are a systems administrator,
you can use virtualization to more easily separate your services and move them
around based on demand.

The default virtualization technology supported in Ubuntu is KVM. KVM requires
virtualization extensions built into Intel and AMD hardware. Xen is also
supported on Ubuntu. Xen can take advantage of virtualization extensions, when
available, but can also be used on hardware without virtualization extensions.
Qemu is another popular solution for hardware without virtualization
extensions.

## libvirt
The libvirt library is used to interface with different virtualization
technologies. Before getting started with libvirt it is best to make sure your
hardware supports the necessary virtualization extensions for KVM. Enter the
following from a terminal prompt:

```bash
kvm-ok
```

A message will be printed informing you if your CPU *does* or *does not*
support hardware virtualization.

!!! Note: On many computers with processors supporting hardware assisted
virtualization, it is necessary to activate an option in the BIOS to enable
it.

### Virtual Networking
There are a few different ways to allow a virtual machine access to the
external network. The default virtual network configuration includes
*bridging* and *iptables* rules implementing *usermode* networking, which uses
the SLIRP protocol. Traffic is NATed through the host interface to the outside
network.

To enable external hosts to directly access services on virtual machines a
different type of *bridge* than the default needs to be configured. This
allows the virtual interfaces to connect to the outside network through the
physical interface, making them appear as normal hosts to the rest of the
network.

### Installation 
To install the necessary packages, from a terminal prompt enter:

```bash
sudo apt install qemu-kvm libvirt-bin
```

After installing libvirt-bin, the user used to manage virtual machines will
need to be added to the *libvirtd* group. Doing so will grant the user access
to the advanced networking options.

In a terminal enter:

```bash
sudo adduser $USER libvirtd
```

!!! Note: If the user chosen is the current user, you will need to log out and back in
for the new group membership to take effect.

!!! Note: In more recent releases (&gt;= Yakkety) the group was renamed to *libvirt*.
Upgraded systems get a new *libvirt* group with the same gid as the
*libvirtd* group to match that.

You are now ready to install a *Guest* operating system. Installing a virtual
machine follows the same process as installing the operating system directly
on the hardware. You either need a way to automate the installation, or a
keyboard and monitor will need to be attached to the physical machine.

In the case of virtual machines a Graphical User Interface (GUI) is analogous
to using a physical keyboard and mouse. Instead of installing a GUI the
virt-viewer application can be used to connect to a virtual machine's console
using VNC. See [Virtual Machine Viewer] for more information.

There are several ways to automate the Ubuntu installation process, for
example using preseeds, kickstart, etc. Refer to the [Ubuntu Installation
Guide] for details.

Yet another way to install an Ubuntu virtual machine is to use uvtool. This
application, available as of 14.04, allows you to set up specific VM options,
execute custom post-install scripts, etc. For details see
[Cloud images and uvtool].

Libvirt can also be configured work with Xen. For details, see the Xen Ubuntu
community page referenced below.

### virt-install 
virt-install is part of the virtinst package. To install it, from a terminal
prompt enter:

```bash
sudo apt install virtinst
```

There are several options available when using virt-install. For example:

```bash
sudo virt-install -n web_devel -r 256 \
--disk path=/var/lib/libvirt/images/web_devel.img,bus=virtio,size=4 -c \
ubuntu-DISTRO-REV-SHORT-server-i386.iso --network network=default,model=virtio \
--graphics vnc,listen=0.0.0.0 --noautoconsole -v
```

-   *-n web\_devel:* the name of the new virtual machine will be *web\_devel*
    in this example.

-   *-r 256:* specifies the amount of memory the virtual machine will use
    in megabytes.

-   *--disk path=/var/lib/libvirt/images/web\_devel.img,size=4:* indicates the
    path to the virtual disk which can be a file, partition, or
    logical volume. In this example a file named `web_devel.img` in the
    /var/lib/libvirt/images/ directory, with a size of 4 gigabytes, and using
    *virtio* for the disk bus.

-   *-c ubuntu-DISTRO-REV-SHORT-server-i386.iso:* file to be used as a
    virtual CDROM. The file can be either an ISO file or the path to the
    host's CDROM device.

-   *--network* provides details related to the VM's network interface. Here
    the *default* network is used, and the interface model is configured for
    *virtio*.

-   *--graphics vnc,listen=0.0.0.0:* exports the guest's virtual console using
    VNC and on all host interfaces. Typically servers have no GUI, so another
    GUI based computer on the Local Area Network (LAN) can connect via VNC to
    complete the installation.

-   *--noautoconsole:* will not automatically connect to the virtual
    machine's console.

-   *-v:* creates a fully virtualized guest.

After launching virt-install you can connect to the virtual machine's console
either locally using a GUI (if your server has a GUI), or via a remote VNC
client from a GUI-based computer.

### virt-clone 
The virt-clone application can be used to copy one virtual machine to another.
For example:

```bash
sudo virt-clone -o web_devel -n database_devel -f /path/to/database_devel.img 
```

-   *-o:* original virtual machine.

-   *-n:* name of the new virtual machine.

-   *-f:* path to the file, logical volume, or partition to be used by the new
    virtual machine.

Also, use *-d* or *--debug* option to help troubleshoot problems with
virt-clone.

!!! Note: Replace *web\_devel* and *database\_devel* with appropriate virtual machine
names.

### Virtual Machine Management 
#### virsh
There are several utilities available to manage virtual machines and libvirt.
The virsh utility can be used from the command line. Some examples:

-   To list running virtual machines:

```bash
    virsh list
```

-   To start a virtual machine:

```bash
    virsh start web_devel
```

-   Similarly, to start a virtual machine at boot:

```bash
    virsh autostart web_devel
```

-   Reboot a virtual machine with:

```bash
    virsh reboot web_devel
```

-   The *state* of virtual machines can be saved to a file in order to be
    restored later. The following will save the virtual machine state into a
    file named according to the date:

```bash
    virsh save web_devel web_devel-022708.state
```

```bash
Once saved the virtual machine will no longer be running.
```

-   A saved virtual machine can be restored using:

```bash
    virsh restore web_devel-022708.state
```

-   To shutdown a virtual machine do:

```bash
    virsh shutdown web_devel
```

-   A CDROM device can be mounted in a virtual machine by entering:

```bash
    virsh attach-disk web_devel /dev/cdrom /media/cdrom
```

!!! Note: In the above examples replace *web\_devel* with the appropriate virtual
machine name, and `web_devel-022708.state` with a descriptive file name.
If virsh (or other vir\* tools) shall connect to something else than the
default qemu-kvm/system hipervisor one can find alternatives for the
*connect* option in *man virsh* or [libvirt doc]

#### migration
There are different types of migration available depending on the versions of
libvirt and the hipervisor being used. In general those types are:

-   [offline migration]

-   [live migration]

-   [postcopy migration]

There are various options to those methods, but the entry point for all of
them is *virsh migrate*. Read the integrated help for more detail.

```bash
 virsh migrate --help 
```

Some useful documentation on constraints and considerations about live
migration can be found at the [Ubuntu Wiki]

#### Virtual Machine Manager 
The virt-manager package contains a graphical utility to manage local and
remote virtual machines. To install virt-manager enter:

```bash
sudo apt install virt-manager
```

Since virt-manager requires a Graphical User Interface (GUI) environment it is
recommended to be installed on a workstation or test machine instead of a
production server. To connect to the local libvirt service enter:

```bash
virt-manager -c qemu:///system
```

You can connect to the libvirt service running on another host by entering the
following in a terminal prompt:

```bash
virt-manager -c qemu+ssh://virtnode1.mydomain.com/system
```

!!! Note: The above example assumes that SSH connectivity between the management
system and virtnode1.mydomain.com has already been configured, and uses SSH
keys for authentication. SSH *keys* are needed because libvirt sends the
password prompt to another process. For details on configuring SSH see [???]

### Virtual Machine Viewer 
The virt-viewer application allows you to connect to a virtual machine's
console. virt-viewer does require a Graphical User Interface (GUI) to
interface with the virtual machine.

To install virt-viewer from a terminal enter:

```bash
sudo apt install virt-viewer
```

Once a virtual machine is installed and running you can connect to the virtual
machine's console by using:

```bash
virt-viewer web_devel
```

Similar to virt-manager, virt-viewer can connect to a remote host using *SSH*
with key authentication, as well:

```bash
virt-viewer -c qemu+ssh://virtnode1.mydomain.com/system web_devel
```

Be sure to replace *web\_devel* with the appropriate virtual machine name.

If configured to use a *bridged* network interface you can also setup SSH
access to the virtual machine.

### Resources 
-   See the [KVM] home page for more details.

-   For more information on libvirt see the [libvirt home page]

-   The [Virtual Machine Manager] site has more information on
    virt-manager development.

-   Also, stop by the *\#ubuntu-virt* IRC channel on [freenode] to discuss
    virtualization technology in Ubuntu.

-   Another good resource is the [Ubuntu Wiki KVM] page.

-   For information on Xen, including using Xen with libvirt, please see the
    [Ubuntu Wiki Xen] page.

## Qemu
[Qemu] is a machine emulator that can run operating systems and programs for
one machine on a different machine. Mostly it is not used as emulator but as
virtualizer in collaboration with KVM or XEN kernel components. In that case
it utilizes the virtualization technology of the hardware to virtualize
guests.

While qemu has a [command line interface] and a [monitor] to interact with
running guests those is rarely used that way for other means than development
purposes. [Libvirt] provides an abstraction from specific versions and
hipervisors and encapsulates some workarounds and best practices.

### Upgrading the machine type 
!!! Note: This also is documented along some more constraints and considerations at
the [Ubuntu Wiki][1]

You might want to update your machine type of an existing defined guest to:

-   to pick up latest security fixes and features

-   continue using a guest created on a now unsupported release

In general it is recommended to update machine types when upgrading qemu/kvm
to a new major version. But this can likely never be an automated task as this
change is guest visible. The guest devices might change in appearance, new
features will be announced to the guest and so on. Linux is usually very good
at tolerating such changes, but it depends so much on the setup and workload
of the guest that this has to be evaluated by the owner/admin of the system.
Other operating systems where known to often have severe impacts by changing
the hardware. Consider a machine type change similar to replacing all devices
and firmware of a physical machine to the latest revision - all considerations
that apply there apply to evaluating a machine type upgrade as well.

As usual with major configuration changes it is wise to back up your guest
definition and disk state to be able to do a rollback just in case. There is
no integrated single command to update the machine type via virsh or similar
tools. It is a normal part of your machine definition. And therefore updated
the same way as most others.

First shutdown your machine and wait until it has reached that state.

```bash
virsh shutdown <yourmachine>
# wait
virsh list --inactive
# should now list your machine as "shut off"
```
```bash
        
```

Then edit the machine definition and find the type in the type tag at the
machine attribute.

```bash
virsh edit <yourmachine>
<type arch='x86_64' machine='pc-i440fx-xenial'>hvm</type>
```
```bash
        
```

Change this to the value you want. If you need to check what types are
available via "-M ?" Note that while providing upstream types as convenience
only Ubuntu types are supported. There you can also see what the current
default would be. In general it is strongly recommended that you change to
newer types if possible to exploit newer features, but also to benefit of
bugfixes that only apply to the newer device virtualization.

```bash
kvm -M ?
# lists machine types, e.g.
pc-i440fx-xenial       Ubuntu 16.04 PC (i440FX + PIIX, 1996) (default)
...
```
```bash
        
```

After this you can start your guest again. You can check the current machine
type from guest and host depending on your needs.

```bash
virsh start <yourmachine>
# check from host, via dumping the active xml definition
virsh dumpxml <yourmachine> | xmllint --xpath "string(//domain/os/type/@machine)" -
# or from the guest via dmidecode
sudo dmidecode | grep Product -A 1
        Product Name: Standard PC (i440FX + PIIX, 1996)
        Version: pc-i440fx-xenial
```
```bash
        
```

If you keep non-live definitions around like xml files remember to update
those as well.

## Cloud images and uvtool
### Introduction 
With Ubuntu being one of the most used operating systems on many cloud
platforms, the availability of stable and secure cloud images has become very
important. As of 12.04 the utilization of cloud images outside of a cloud
infrastructure has been improved. It is now possible to use those images to
create a virtual machine without the need of a complete installation.

### Creating virtual machines using uvtool
Starting with 14.04 LTS, a tool called uvtool greatly facilitates the task of
generating virtual machines (VM) using the cloud images. uvtool provides a
simple mechanism to synchronize cloud-images locally and use them to create
new VMs in minutes.

#### Uvtool packages
The following packages and their dependencies will be required in order to use
uvtool:

-   uvtool

-   uvtool-libvirt

To install uvtool, run:

```bash
$ apt -y install uvtool
```

This will install uvtool's main commands:

-   uvt-simplestreams-libvirt

-   uvt-kvm

#### Get the Ubuntu Cloud Image with uvt-simplestreams-libvirt
This is one of the major simplifications that uvtool brings. It is aware of
where to find the cloud images so only one command is required to get a new
cloud image. For instance, if you want to synchronize all cloud images for the
amd64 architecture, the uvtool command would be:

```bash
$ uvt-simplestreams-libvirt sync arch=amd64
```

After an amount of time required to download all the images from the Internet,
you will have a complete set of cloud images stored locally. To see what has
been downloaded use the following command:

```bash
$ uvt-simplestreams-libvirt query
release=oneiric arch=amd64 label=release (20130509)
release=precise arch=amd64 label=release (20160315)
release=quantal arch=amd64 label=release (20140409)
release=raring arch=amd64 label=release (20140111)
release=saucy arch=amd64 label=release (20140709)
release=trusty arch=amd64 label=release (20160314)
release=utopic arch=amd64 label=release (20150723)
release=vivid arch=amd64 label=release (20160203)
release=wily arch=amd64 label=release (20160315)
release=xenial arch=amd64 label=beta1 (20160223.1)
```

In the case where you want to synchronize only one specific cloud-image, you
need to use the release= and arch= filters to identify which image needs to be
synchronized.

```bash
$ uvt-simplestreams-libvirt sync release=DISTRO-SHORT-CODENAME arch=amd64
```

#### Create the VM using uvt-kvm
In order to connect to the virtual machine once it has been created, you must
have a valid SSH key available for the Ubuntu user. If your environment does
not have an SSH key, you can easily create one using the following command:

```bash
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/ubuntu/.ssh/id_rsa): 
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/ubuntu/.ssh/id_rsa.
Your public key has been saved in /home/ubuntu/.ssh/id_rsa.pub.
The key fingerprint is:
4d:ba:5d:57:c9:49:ef:b5:ab:71:14:56:6e:2b:ad:9b ubuntu@DISTRO-SHORT-CODENAMES
The key's randomart image is:
+--[ RSA 2048]----+
|               ..|
|              o.=|
|          .    **|
|         +    o+=|
|        S . ...=.|
|         o . .+ .|
|        . .  o o |
|              *  |
|             E   |
+-----------------+
```

To create of a new virtual machine using uvtool, run the following in a
terminal:

```bash
$ uvt-kvm create firsttest
```

This will create a VM named **firsttest** using the current LTS cloud image
available locally. If you want to specify a release to be used to create the
VM, you need to use the **release=** filter:

```bash
$ uvt-kvm create secondtest release=DISTRO-SHORT-CODENAME
```

uvt-kvm wait can be used to wait until the creation of the VM has completed:

```bash
$ uvt-kvm wait secondttest --insecure
Warning: secure wait for boot-finished not yet implemented; use --insecure.
```

#### Connect to the running VM
Once the virtual machine creation is completed, you can connect to it using
SSH:

```bash
$ uvt-kvm ssh secondtest --insecure
```

For the time being, the **--insecure** is required, so use this mechanism to
connect to your VM only if you completely trust your network infrastructure.

You can also connect to your VM using a regular SSH session using the IP
address of the VM. The address can be queried using the following command:

```bash
$ uvt-kvm ip secondtest
192.168.122.199
$ ssh -i ~/.ssh/id_rsa ubuntu@192.168.122.199
The authenticity of host '192.168.122.199 (192.168.122.199)' can't be established.
ECDSA key fingerprint is SHA256:8oxaztRWzTMtv8SC9LYyjuqBu79Z9JP8bUGh6G8R8cw.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.122.199' (ECDSA) to the list of known hosts.
Welcome to Ubuntu DISTRO-VERSION (development branch) (GNU/Linux LINUX-KERNEL-VERSION-X-generic ARCH)
```

```bash
 * Documentation:  https://help.ubuntu.com/
```

```bash
  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud
```

```bash
0 packages can be updated.
0 updates are security updates.
```



```bash
The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.
```

```bash
Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.
```

```bash
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.
```

```bash
ubuntu@secondtest:~$ 
```

#### Get the list of running VMs
You can get the list of VMs running on your system with this command:

```bash
$ uvt-kvm list
secondtest
```

#### Destroy your VM
Once you are done with your VM, you can destroy it with:

```bash
$ uvt-kvm destroy secondtest
```

#### More uvt-kvm options
The following options can be used to change some of the characteristics of the
VM that you are creating:

-   --memory : Amount of RAM in megabytes. Default: 512.

-   --disk : Size of the OS disk in gigabytes. Default: 8.

-   --cpu : Number of CPU cores. Default: 1.

Some other parameters will have an impact on the cloud-init configuration:

-   --password password : Allow login to the VM using the Ubuntu account and
    this provided password.

-   --run-script-once script\_file : Run script\_file as root on the VM the
    first time it is booted, but never again.

-   --packages package\_list : Install the comma-separated packages specified
    in package\_list on first boot.

A complete description of all available modifiers is available in the manpage
of uvt-kvm.

### Resources 
If you are interested in learning more, have questions or suggestions, please
contact the Ubuntu Server Team at:

-   IRC: \#ubuntu-server on freenode

-   Mailing list: [ubuntu-server at lists.ubuntu.com]

## Ubuntu Cloud 
Cloud computing is a computing model that allows vast pools of resources to be
allocated on-demand. These resources such as storage, computing power, network
and software are abstracted and delivered as a service over the Internet
anywhere, anytime. These services are billed per time consumed similar to the
ones used by public services such as electricity, water and telephony. Ubuntu
Cloud Infrastructure uses OpenStack open source software to help build highly
scalable, cloud computing for both public and private clouds.

### Installation and Configuration 
Due to the current high rate of development of this complex technology we
refer the reader to [upstream documentation] for all matters concerning
installation and configuration.

### Support and Troubleshooting 
Community Support

-   [OpenStack Mailing list]

-   [The OpenStack Wiki search]

-   [Launchpad bugs area]

-   Join the IRC channel \#openstack on freenode.

### Resources 
-   [Cloud Computing - Service models]

-   [OpenStack Compute]

-   [OpenStack Image Service]

-   [OpenStack Object Storage Administration Guide]

-   [Installing OpenStack Object Storage on Ubuntu]

-   <http://cloudglossary.com/>

## LXD
LXD (pronounced lex-dee) is the lightervisor, or lightweight container
hypervisor. While this claim has been controversial, it has been [quite well
justified] based on the original academic paper. It also nicely distinguishes
LXD from [LXC].

LXC (lex-see) is a program which creates and administers "containers" on a
local system. It also provides an API to allow higher level managers, such as
LXD, to administer containers. In a sense, one could compare LXC to QEMU,
while comparing LXD to libvirt.

The LXC API deals with a 'container'. The LXD API deals with 'remotes', which
serve images and containers. This extends the LXC functionality over the
network, and allows concise management of tasks like container migration and
container image publishing.

LXD uses LXC under the covers for some container management tasks. However, it
keeps its own container configuration information and has its own conventions,
so that it is best not to use classic LXC commands by hand with LXD
containers. This document will focus on how to configure and administer LXD on
Ubuntu systems.

### Online Resources 
There is excellent documentation for [getting started with LXD] in the online
LXD README. There is also an online server allowing you to [try out LXD
remotely]. Stephane Graber also has an [excellent blog series] on LXD 2.0.
Finally, there is great documentation on how to [drive lxd using juju].

This document will offer an Ubuntu Server-specific view of LXD, focusing on
administration.

### Installation 
LXD is pre-installed on Ubuntu Server cloud images. On other systems, the lxd
package can be installed using:


```bash
sudo apt install lxd
```

This will install LXD as well as the recommended dependencies, including the
LXC library and lxcfs.

### Kernel preparation 
In general, Ubuntu 16.04 should have all the desired features enabled by
default. One exception to this is that in order to enable swap accounting the
boot argument `swapaccount=1` must be set. This can be done by appending it to
the `GRUB_CMDLINE_LINUX_DEFAULT=`variable in /etc/default/grub, then running
'update-grub' as root and rebooting.

### Configuration 
By default, LXD is installed listening on a local UNIX socket, which members
of group LXD can talk to. It has no trust password setup. And it uses the
filesystem at `/var/lib/lxd` to store containers. To configure LXD with
different settings, use `lxd
      init`. This will allow you to choose:

-   Directory or [ZFS] container backend. If you choose ZFS, you can choose
    which block devices to use, or the size of a file to use as backing store.

-   Availability over the network

-   A 'trust password' used by remote clients to vouch for their client
    certificate

You must run 'lxd init' as root. 'lxc' commands can be run as any user who is
member of group lxd. If user joe is not a member of group 'lxd', you may run:


```bash
adduser joe lxd
```

as root to change it. The new membership will take effect on the next login,
or after running 'newgrp lxd' from an existing login.

For more information on server, container, profile, and device configuration,
please refer to the definitive configuration provided with the source code,
which can be found [online]

### Creating your first container 
This section will describe the simplest container tasks.

#### Creating a container
Every new container is created based on either an image, an existing
container, or a container snapshot. At install time, LXD is configured with
the following image servers:

-   `ubuntu`: this serves official Ubuntu server cloud image releases.

-   `ubuntu-daily`: this serves official Ubuntu server cloud images of the
    daily development releases.

-   `images`: this is a default-installed alias for
    images.linuxcontainers.org. This is serves classical lxc images built
    using the same images which the LXC 'download' template uses. This
    includes various distributions and minimal custom-made Ubuntu images. This
    is not the recommended server for Ubuntu images.

The command to create and start a container is


```bash
lxc launch remote:image containername
```

Images are identified by their hash, but are also aliased. The 'ubuntu' server
knows many aliases such as '16.04' and 'xenial'. A list of all images
available from the Ubuntu Server can be seen using:


```bash
lxc image list ubuntu:
```

To see more information about a particular image, including all the aliases it
is known by, you can use:


```bash
lxc image info ubuntu:xenial
```

You can generally refer to an Ubuntu image using the release name ('xenial')
or the release number (16.04). In addition, 'lts' is an alias for the latest
supported LTS release. To choose a different architecture, you can specify the
desired architecture:


```bash
lxc image info ubuntu:lts/arm64
```

Now, let's start our first container:


```bash
lxc launch ubuntu:xenial x1
```

This will download the official current Xenial cloud image for your current
architecture, then create a container using that image, and finally start it.
Once the command returns, you can see it using:


```bash
lxc list
lxc info x1
```

and open a shell in it using:


```bash
lxc exec x1 bash
```

The try-it page gives a full synopsis of the commands you can use to
administer containers.

Now that the 'xenial' image has been downloaded, it will be kept in sync until
no new containers have been created based on it for (by default) 10 days.
After that, it will be deleted.

### LXD Server Configuration 
By default, LXD is socket activated and configured to listen only on a local
UNIX socket. While LXD may not be running when you first look at the process
listing, any LXC command will start it up. For instance:


```bash
lxc list
```

This will create your client certificate and contact the LXD server for a list
of containers. To make the server accessible over the network you can set the
http port using:


```bash
lxc config set core.https_address :8443
```

This will tell LXD to listen to port 8843 on all addresses.

#### Authentication
By default, LXD will allow all members of group 'lxd' (which by default
includes all members of group admin) to talk to it over the UNIX socket.
Communication over the network is authorized using server and client
certificates.

Before client c1 wishes to use remote r1, r1 must be registered using:


```bash
lxc remote add r1 r1.example.com:8443
```

The fingerprint of r1's certificate will be shown, to allow the user at c1 to
reject a false certificate. The server in turn will verify that c1 may be
trusted in one of two ways. The first is to register it in advance from any
already-registered client, using:


```bash
lxc config trust add r1 certfile.crt
```

Now when the client adds r1 as a known remote, it will not need to provide a
password as it is already trusted by the server.

The other is to configure a 'trust password' with r1, either at initial
configuration using 'lxd init', or after the fact using


```bash
lxc config set core.trust_password PASSWORD
```

The password can then be provided when the client registers r1 as a known
remote.

#### Backing store
LXD supports several backing stores. The recommended backing store is ZFS,
however this is not available on all platforms. Supported backing stores
include:

-   ext4: this is the default, and easiest to use. With an ext4 backing store,
    containers and images are simply stored as directories on the host
    filesystem. Launching new containers requires copying a whole filesystem,
    and 10 containers will take up 10 times as much space as one container.

-   ZFS: if ZFS is supported on your architecture (amd64, arm64, or ppc64le),
    you can set LXD up to use it using 'lxd init'. If you already have a ZFS
    pool configured, you can tell LXD to use it by setting the zfs\_pool\_name
    configuration key:


```bash
    lxc config set storage.zfs_pool_name lxd
```

```bash
With ZFS, launching a new container is fast because the filesystem starts
as a copy on write clone of the images' filesystem. Note that unless the
container is privileged (see below) LXD will need to change ownership of
all files before the container can start, however this is fast and change
very little of the actual filesystem data.
```

-   Btrfs: btrfs can be used with many of the same advantages as ZFS. To use
    BTRFS as a LXD backing store, simply mount a Btrfs filesystem under
    `/var/lib/lxd`. LXD will detect this and exploit the Btrfs subvolume
    feature whenever launching a new container or snapshotting a container.

-   LVM: To use a LVM volume group called 'lxd', you may tell LXD to use that
    for containers and images using the command


```bash
        lxc config set storage.lvm_vg_name lxd
```

```bash
When launching a new container, its rootfs will start as a lv clone. It is
immediately mounted so that the file uids can be shifted, then unmounted.
Container snapshots also are created as lv snapshots.
```

### Container configuration 
Containers are configured according to a set of profiles, described in the
next section, and a set of container-specific configuration. Profiles are
applied first, so that container specific configuration can override profile
configuration.

Container configuration includes properties like the architecture, limits on
resources such as CPU and RAM, security details including apparmor restriction
overrides, and devices to apply to the container.

Devices can be of several types, including UNIX character, UNIX block, network
interface, or 'disk'. In order to insert a host mount into a container, a
'disk' device type would be used. For instance, to mount /opt in container c1
at /opt, you could use:


```bash
lxc config device add c1 opt disk source=/opt path=opt
```

See:


```bash
lxc help config
```

for more information about editing container configurations. You may also use:


```bash
lxc config edit c1
```

to edit the whole of c1's configuration in your specified \$EDITOR. Comments
at the top of the configuration will show examples of correct syntax to help
administrators hit the ground running. If the edited configuration is not
valid when the \$EDITOR is exited, then \$EDITOR will be restarted.

### Profiles 
Profiles are named collections of configurations which may be applied to more
than one container. For instance, all containers created with 'lxc launch', by
default, include the 'default' profile, which provides a network interface
'eth0'.

To mask a device which would be inherited from a profile but which should not
be in the final container, define a device by the same name but of type
'none':


```bash
lxc config device add c1 eth1 none
```

### Nesting 
Containers all share the same host kernel. This means that there is always an
inherent trade-off between features exposed to the container and host security
from malicious containers. Containers by default are therefore restricted from
features needed to nest child containers. In order to run lxc or lxd
containers under a lxd container, the 'security.nesting' feature must be set
to true:


```bash
lxc config set container1 security.nesting true
```

Once this is done, container1 will be able to start sub-containers.

In order to run unprivileged (the default in LXD) containers nested under an
unprivileged container, you will need to ensure a wide enough UID mapping.
Please see the 'UID mapping' section below.

#### Docker
In order to facilitate running docker containers inside a LXD container, a
'docker' profile is provided. To launch a new container with the docker
profile, you can run:


```bash
lxc launch xenial container1 -p default -p docker
```

Note that currently the docker package in Ubuntu 16.04 is patched to
facilitate running in a container. This support is expected to land upstream
soon.

Note that 'cgroup namespace' support is also required. This is available in
the 16.04 kernel as well as in the 4.6 upstream source.

### Limits 
LXD supports flexible constraints on the resources which containers can
consume. The limits come in the following categories:

-   CPU: limit cpu available to the container in several ways.

-   Disk: configure the priority of I/O requests under load

-   RAM: configure memory and swap availability

-   Network: configure the network priority under load

-   Processes: limit the number of concurrent processes in the container.

For a full list of limits known to LXD, see [the configuration
documentation][online].

### UID mappings and Privileged containers 
By default, LXD creates unprivileged containers. This means that root in the
container is a non-root UID on the host. It is privileged against the
resources owned by the container, but unprivileged with respect to the host,
making root in a container roughly equivalent to an unprivileged user on the
host. (The main exception is the increased attack surface exposed through the
system call interface)

Briefly, in an unprivileged container, 65536 UIDs are 'shifted' into the
container. For instance, UID 0 in the container may be 100000 on the host, UID
1 in the container is 100001, etc, up to 165535. The starting value for UIDs
and GIDs, respectively, is determined by the 'root' entry the `/etc/subuid`
and `/etc/subgid` files. (See the [subuid(5) manual page].

It is possible to request a container to run without a UID mapping by setting
the security.privileged flag to true:


```bash
lxc config set c1 security.privileged true
```

Note however that in this case the root user in the container is the root user
on the host.

### Apparmor 
LXD confines containers by default with an apparmor profile which protects
containers from each other and the host from containers. For instance this
will prevent root in one container from signaling root in another container,
even though they have the same uid mapping. It also prevents writing to
dangerous, un-namespaced files such as many sysctls and
` /proc/sysrq-trigger`.

If the apparmor policy for a container needs to be modified for a container
c1, specific apparmor policy lines can be added in the 'raw.apparmor'
configuration key.

### Seccomp 
All containers are confined by a default seccomp policy. This policy prevents
some dangerous actions such as forced umounts, kernel module loading and
unloading, kexec, and the open\_by\_handle\_at system call. The seccomp
configuration cannot be modified, however a completely different seccomp
policy - or none - can be requested using raw.lxc (see below).

### Raw LXC configuration
LXD configures containers for the best balance of host safety and container
usability. Whenever possible it is highly recommended to use the defaults, and
use the LXD configuration keys to request LXD to modify as needed. Sometimes,
however, it may be necessary to talk to the underlying lxc driver itself. This
can be done by specifying LXC configuration items in the 'raw.lxc' LXD
configuration key. These must be valid items as documented in [the
lxc.container.conf(5) manual page].

### Images and containers
LXD is image based. When you create your first container, you will generally
do so using an existing image. LXD comes pre-configured with three default
image remotes:

-   ubuntu: This is a [simplestreams-based] remote serving released ubuntu
    cloud images.

-   ubuntu-daily: This is another simplestreams based remote which serves
    'daily' ubuntu cloud images. These provide quicker but potentially less
    stable images.

-   images: This is a remote publishing best-effort container images for many
    distributions, created using community-provided build scripts.

To view the images available on one of these servers, you can use:


```bash
lxc image list ubuntu:
```

Most of the images are known by several aliases for easier reference. To see
the full list of aliases, you can use


```bash
lxc image alias list images:
```

Any alias or image fingerprint can be used to specify how to create the new
container. For instance, to create an amd64 Ubuntu 14.04 container, some
options are:


```bash
lxc launch ubuntu:14.04 trusty1
lxc launch ubuntu:trusty trusty1
lxc launch ubuntu:trusty/amd64 trusty1
lxc launch ubuntu:lts trusty1
```

The 'lts' alias always refers to the latest released LTS image.

#### Snapshots
Containers can be renamed and live-migrated using the 'lxc move' command:


```bash
lxc move c1 final-beta
```

They can also be snapshotted:


```bash
lxc snapshot c1 YYYY-MM-DD
```

Later changes to c1 can then be reverted by restoring the snapshot:


```bash
lxc restore u1 YYYY-MM-DD
```

New containers can also be created by copying a container or snapshot:


```bash
lxc copy u1/YYYY-MM-DD testcontainer
```

#### Publishing images
When a container or container snapshot is ready for consumption by others, it
can be published as a new image using;


```bash
lxc publish u1/YYYY-MM-DD --alias foo-2.0
```

The published image will be private by default, meaning that LXD will not
allow clients without a trusted certificate to see them. If the image is safe
for public viewing (i.e. contains no private information), then the 'public'
flag can be set, either at publish time using


```bash
lxc publish u1/YYYY-MM-DD --alias foo-2.0 public=true
```

or after the fact using


```bash
lxc image edit foo-2.0
```

and changing the value of the public field.

#### Image export and import
Image can be exported as, and imported from, tarballs:


```bash
lxc image export foo-2.0 foo-2.0.tar.gz
lxc image import foo-2.0.tar.gz --alias foo-2.0 --public
```

### Troubleshooting 
To view debug information about LXD itself, on a systemd based host use


```bash
journalctl -u LXD
```

On an Upstart-based system, you can find the log in
`/var/log/upstart/lxd.log`. To make LXD provide much more information about
requests it is serving, add '--debug' to LXD's arguments. In systemd, append
'--debug' to the 'ExecStart=' line in `/lib/systemd/system/lxd.service`. In
Upstart, append it to the `exec /usr/bin/lxd` line in `/etc/init/lxd.conf`.

Container logfiles for container c1 may be seen using:


```bash
lxc info c1 --show-log
```

The configuration file which was used may be found under
` /var/log/lxd/c1/lxc.conf` while apparmor profiles can be found in
` /var/lib/lxd/security/apparmor/profiles/c1` and seccomp profiles in
` /var/lib/lxd/security/seccomp/c1`.

## LXC
Containers are a lightweight virtualization technology. They are more akin to
an enhanced chroot than to full virtualization like Qemu or VMware, both
because they do not emulate hardware and because containers share the same
operating system as the host. Containers are similar to Solaris zones or BSD
jails. Linux-vserver and OpenVZ are two pre-existing, independently developed
implementations of containers-like functionality for Linux. In fact,
containers came about as a result of the work to upstream the vserver and
OpenVZ functionality.

There are two user-space implementations of containers, each exploiting the
same kernel features. Libvirt allows the use of containers through the LXC
driver by connecting to 'lxc:///'. This can be very convenient as it supports
the same usage as its other drivers. The other implementation, called simply
'LXC', is not compatible with libvirt, but is more flexible with more
userspace tools. It is possible to switch between the two, though there are
peculiarities which can cause confusion.

In this document we will mainly describe the lxc package. Use of libvirt-lxc
is not generally recommended due to a lack of Apparmor protection for
libvirt-lxc containers.

In this document, a container name will be shown as CN, C1, or C2.

### Installation 
The lxc package can be installed using


```bash
sudo apt install lxc
```

This will pull in the required and recommended dependencies, as well as set up
a network bridge for containers to use. If you wish to use unprivileged
containers, you will need to ensure that users have sufficient allocated
subuids and subgids, and will likely want to allow users to connect containers
to a bridge (see [Basic unprivileged usage]).

### Basic usage 
LXC can be used in two distinct ways - privileged, by running the lxc commands
as the root user; or unprivileged, by running the lxc commands as a non-root
user. (The starting of unprivileged containers by the root user is possible,
but not described here.) Unprivileged containers are more limited, for
instance being unable to create device nodes or mount block-backed
filesystems. However they are less dangerous to the host, as the root userid
in the container is mapped to a non-root userid on the host.

#### Basic privileged usage
To create a privileged container, you can simply do:


```bash
sudo lxc-create --template download --name u1
```

```bash
or, abbreviated
```

```bash
sudo lxc-create -t download -n u1
```

This will interactively ask for a container root filesystem type to download -
in particular the distribution, release, and architecture. To create the
container non-interactively, you can specify these values on the command line:


```bash
sudo lxc-create -t download -n u1 -- --dist ubuntu --release DISTRO-SHORT-CODENAME --arch amd64
```

```bash
or
```

```bash
sudo lxc-create -t download -n u1 -- -d ubuntu -r DISTRO-SHORT-CODENAME -a amd64
```

You can now use `lxc-ls` to list containers, `lxc-info` to obtain detailed
container information, `lxc-start` to start and `lxc-stop` to stop the
container. `lxc-attach` and `lxc-console` allow you to enter a container, if
ssh is not an option. `lxc-destroy` removes the container, including its
rootfs. See the manual pages for more information on each command. An example
session might look like:


```bash
sudo lxc-ls --fancy
sudo lxc-start --name u1 --daemon
sudo lxc-info --name u1
sudo lxc-stop --name u1
sudo lxc-destroy --name u1
```

#### User namespaces
Unprivileged containers allow users to create and administer containers
without having any root privilege. The feature underpinning this is called
user namespaces. User namespaces are hierarchical, with privileged tasks in a
parent namespace being able to map its ids into child namespaces. By default
every task on the host runs in the initial user namespace, where the full
range of ids is mapped onto the full range. This can be seen by looking at
/proc/self/uid\_map and /proc/self/gid\_map, which both will show "0 0
4294967295" when read from the initial user namespace. As of Ubuntu 14.04,
when new users are created they are by default offered a range of userids. The
list of assigned ids can be seen in the files `/etc/subuid` and `/etc/subgid`
See their respective manpages for more information. Subuids and subgids are by
convention started at id 100000 to avoid conflicting with system users.

If a user was created on an earlier release, it can be granted a range of ids
using `usermod`, as follows:


```bash
sudo usermod -v 100000-200000 -w 100000-200000 user1
```

The programs `newuidmap` and `
    newgidmap` are setuid-root programs in the `uidmap` package, which are
used internally by lxc to map subuids and subgids from the host into the
unprivileged container. They ensure that the user only maps ids which are
authorized by the host configuration.

#### Basic unprivileged usage 
To create unprivileged containers, a few first steps are needed. You will need
to create a default container configuration file, specifying your desired id
mappings and network setup, as well as configure the host to allow the
unprivileged user to hook into the host network. The example below assumes
that your mapped user and group id ranges are 100000-165536. Check your actual
user and group id ranges and modify the example accordingly:


```bash
grep $USER /etc/subuid
grep $USER /etc/subgid
```


```bash
mkdir -p ~/.config/lxc
echo "lxc.id_map = u 0 100000 65536" > ~/.config/lxc/default.conf
echo "lxc.id_map = g 0 100000 65536" >> ~/.config/lxc/default.conf
echo "lxc.network.type = veth" >> ~/.config/lxc/default.conf
echo "lxc.network.link = lxcbr0" >> ~/.config/lxc/default.conf
echo "$USER veth lxcbr0 2" | sudo tee -a /etc/lxc/lxc-usernet
```

After this, you can create unprivileged containers the same way as privileged
ones, simply without using sudo.


```bash
lxc-create -t download -n u1 -- -d ubuntu -r DISTRO-SHORT-CODENAME -a amd64
lxc-start -n u1 -d
lxc-attach -n u1
lxc-stop -n u1
lxc-destroy -n u1
```

#### Nesting 
In order to run containers inside containers - referred to as nested
containers - two lines must be present in the parent container configuration
file:

```bash
lxc.mount.auto = cgroup
lxc.aa_profile = lxc-container-default-with-nesting
```

The first will cause the cgroup manager socket to be bound into the container,
so that lxc inside the container is able to administer cgroups for its nested
containers. The second causes the container to run in a looser Apparmor policy
which allows the container to do the mounting required for starting
containers. Note that this policy, when used with a privileged container, is
much less safe than the regular policy or an unprivileged container. See
[Apparmor] for more information.

### Global configuration 
The following configuration files are consulted by LXC. For privileged use,
they are found under `/etc/lxc`, while for unprivileged use they are under
`~/.config/lxc`.

-   `lxc.conf` may optionally specify alternate values for several lxc
    settings, including the lxcpath, the default configuration, cgroups to
    use, a cgroup creation pattern, and storage backend settings for lvm
    and zfs.

-   `default.conf` specifies configuration which every newly created container
    should contain. This usually contains at least a network section, and, for
    unprivileged users, an id mapping section

-   `lxc-usernet.conf` specifies how unprivileged users may connect their
    containers to the host-owned network.

`lxc.conf` and `default.conf` are both under `/etc/lxc` and
`$HOME/.config/lxc`, while `lxc-usernet.conf` is only host-wide.

By default, containers are located under /var/lib/lxc for the root user, and
\$HOME/.local/share/lxc otherwise. The location can be specified for all lxc
commands using the "-P|--lxcpath" argument.

### Networking 
By default LXC creates a private network namespace for each container, which
includes a layer 2 networking stack. Containers usually connect to the outside
world by either having a physical NIC or a veth tunnel endpoint passed into
the container. LXC creates a NATed bridge, lxcbr0, at host startup. Containers
created using the default configuration will have one veth NIC with the remote
end plugged into the lxcbr0 bridge. A NIC can only exist in one namespace at a
time, so a physical NIC passed into the container is not usable on the host.

It is possible to create a container without a private network namespace. In
this case, the container will have access to the host networking like any
other application. Note that this is particularly dangerous if the container
is running a distribution with upstart, like Ubuntu, since programs which talk
to init, like `shutdown`, will talk over the abstract Unix domain socket to
the host's upstart, and shut down the host.

To give containers on lxcbr0 a persistent ip address based on domain name, you
can write entries to `/etc/lxc/dnsmasq.conf` like:

```bash
dhcp-host=lxcmail,10.0.3.100
dhcp-host=ttrss,10.0.3.101
```

If it is desirable for the container to be publicly accessible, there are a
few ways to go about it. One is to use `iptables` to forward host ports to the
container, for instance

```bash
 iptables -t nat -A PREROUTING -p tcp -i eth0 --dport 587 -j DNAT \
    --to-destination 10.0.3.100:587
```
```bash
 
```

Another is to bridge the host's network interfaces (see the Ubuntu Server
Guide's Network Configuration chapter, [???][2]). Then, specify the host's
bridge in the container configuration file in place of lxcbr0, for instance

```bash
lxc.network.type = veth
lxc.network.link = br0
```

Finally, you can ask LXC to use macvlan for the container's NIC. Note that
this has limitations and depending on configuration may not allow the
container to talk to the host itself. Therefore the other two options are
preferred and more commonly used.

There are several ways to determine the ip address for a container. First, you
can use `lxc-ls --fancy` which will print the ip addresses for all running
containers, or `lxc-info -i -H -n C1` which will print C1's ip address. If
dnsmasq is installed on the host, you can also add an entry to
`/etc/dnsmasq.conf` as follows

```bash
server=/lxc/10.0.3.1
```

after which dnsmasq will resolve C1.lxc locally, so that you can do:

```bash
ping C1
ssh C1
```

For more information, see the lxc.conf manpage as well as the example network
configurations under `/usr/share/doc/lxc/examples/`.

### LXC startup
LXC does not have a long-running daemon. However it does have three upstart
jobs.

-   `/etc/init/lxc-net.conf:` is an optional job which only runs if `
           /etc/default/lxc-net` specifies USE\_LXC\_BRIDGE (true by default).
    It sets up a NATed bridge for containers to use.

-   `/etc/init/lxc.conf` loads the lxc apparmor profiles and optionally starts
    any autostart containers. The autostart containers will be ignored if
    LXC\_AUTO (true by default) is set to true in `/etc/default/lxc`. See the
    lxc-autostart manual page for more information on autostarted containers.

-   `/etc/init/lxc-instance.conf` is used by `/etc/init/lxc.conf` to autostart
    a container.

### Backing Stores 
LXC supports several backing stores for container root filesystems. The
default is a simple directory backing store, because it requires no prior host
customization, so long as the underlying filesystem is large enough. It also
requires no root privilege to create the backing store, so that it is seamless
for unprivileged use. The rootfs for a privileged directory backed container
is located (by default) under `/var/lib/lxc/C1/rootfs`, while the rootfs for
an unprivileged container is under `~/.local/share/lxc/C1/rootfs`. If a custom
lxcpath is specified in lxc.system.com, then the container rootfs will be
under `$lxcpath/C1/rootfs`.

A snapshot clone C2 of a directory backed container C1 becomes an overlayfs
backed container, with a rootfs called
`overlayfs:/var/lib/lxc/C1/rootfs:/var/lib/lxc/C2/delta0`. Other backing store
types include loop, btrfs, LVM and zfs.

A btrfs backed container mostly looks like a directory backed container, with
its root filesystem in the same location. However, the root filesystem
comprises a subvolume, so that a snapshot clone is created using a subvolume
snapshot.

The root filesystem for an LVM backed container can be any separate LV. The
default VG name can be specified in lxc.conf. The filesystem type and size are
configurable per-container using lxc-create.

The rootfs for a zfs backed container is a separate zfs filesystem, mounted
under the traditional `/var/lib/lxc/C1/rootfs` location. The zfsroot can be
specified at lxc-create, and a default can be specified in lxc.system.conf.

More information on creating containers with the various backing stores can be
found in the lxc-create manual page.

### Templates 
Creating a container generally involves creating a root filesystem for the
container. `lxc-create` delegates this work to *templates*, which are
generally per-distribution. The lxc templates shipped with lxc can be found
under `/usr/share/lxc/templates`, and include templates to create Ubuntu,
Debian, Fedora, Oracle, centos, and gentoo containers among others.

Creating distribution images in most cases requires the ability to create
device nodes, often requires tools which are not available in other
distributions, and usually is quite time-consuming. Therefore lxc comes with a
special *download* template, which downloads pre-built container images from a
central lxc server. The most important use case is to allow simple creation of
unprivileged containers by non-root users, who could not for instance easily
run the `debootstrap` command.

When running `lxc-create`, all options which come after *--* are passed to the
template. In the following command, *--name*, *--template* and *--bdev* are
passed to `lxc-create`, while *--release* is passed to the template:


```bash
lxc-create --template ubuntu --name c1 --bdev loop -- --release DISTRO-SHORT-CODENAME
```

You can obtain help for the options supported by any particular container by
passing *--help* and the template name to `lxc-create`. For instance, for help
with the download template,


```bash
lxc-create --template download --help
```

### Autostart 
LXC supports marking containers to be started at system boot. Prior to Ubuntu
14.04, this was done using symbolic links under the directory `/etc/lxc/auto`.
Starting with Ubuntu 14.04, it is done through the container configuration
files. An entry


```bash
lxc.start.auto = 1
lxc.start.delay = 5
```

would mean that the container should be started at boot, and the system should
wait 5 seconds before starting the next container. LXC also supports ordering
and grouping of containers, as well as reboot and shutdown by autostart
groups. See the manual pages for lxc-autostart and lxc.container.conf for more
information.

### Apparmor 
LXC ships with a default Apparmor profile intended to protect the host from
accidental misuses of privilege inside the container. For instance, the
container will not be able to write to `/proc/sysrq-trigger` or to most `/sys`
files.

The `usr.bin.lxc-start` profile is entered by running `lxc-start`. This
profile mainly prevents `lxc-start` from mounting new filesystems outside of
the container's root filesystem. Before executing the container's `init`,
`LXC` requests a switch to the container's profile. By default, this profile
is the `lxc-container-default` policy which is defined in
`/etc/apparmor.d/lxc/lxc-default`. This profile prevents the container from
accessing many dangerous paths, and from mounting most filesystems.

Programs in a container cannot be further confined - for instance, MySQL runs
under the container profile (protecting the host) but will not be able to
enter the MySQL profile (to protect the container).

`lxc-execute` does not enter an Apparmor profile, but the container it spawns
will be confined.

#### Customizing container policies
If you find that `lxc-start` is failing due to a legitimate access which is
being denied by its Apparmor policy, you can disable the lxc-start profile by
doing:

```bash
sudo apparmor_parser -R /etc/apparmor.d/usr.bin.lxc-start
sudo ln -s /etc/apparmor.d/usr.bin.lxc-start /etc/apparmor.d/disabled/
```

This will make `lxc-start` run unconfined, but continue to confine the
container itself. If you also wish to disable confinement of the container,
then in addition to disabling the `usr.bin.lxc-start` profile, you must add:

```bash
lxc.aa_profile = unconfined
```

to the container's configuration file.

LXC ships with a few alternate policies for containers. If you wish to run
containers inside containers (nesting), then you can use the
lxc-container-default-with-nesting profile by adding the following line to the
container configuration file

```bash
lxc.aa_profile = lxc-container-default-with-nesting
```
```bash
    
```

If you wish to use libvirt inside containers, then you will need to edit that
policy (which is defined in `/etc/apparmor.d/lxc/lxc-default-with-nesting`) by
uncommenting the following line:

```bash
mount fstype=cgroup -> /sys/fs/cgroup/**,
```
```bash
    
```

and re-load the policy.

Note that the nesting policy with privileged containers is far less safe than
the default policy, as it allows containers to re-mount `/sys` and `/proc` in
nonstandard locations, bypassing apparmor protections. Unprivileged containers
do not have this drawback since the container root cannot write to root-owned
`proc` and `sys` files.

Another profile shipped with lxc allows containers to mount block filesystem
types like ext4. This can be useful in some cases like maas provisioning, but
is deemed generally unsafe since the superblock handlers in the kernel have
not been audited for safe handling of untrusted input.

If you need to run a container in a custom profile, you can create a new
profile under `/etc/apparmor.d/lxc/`. Its name must start with `lxc-` in order
for `lxc-start` to be allowed to transition to that profile. The `lxc-default`
profile includes the re-usable abstractions file
`/etc/apparmor.d/abstractions/lxc/container-base`. An easy way to start a new
profile therefore is to do the same, then add extra permissions at the bottom
of your policy.

After creating the policy, load it using:

```bash
sudo apparmor_parser -r /etc/apparmor.d/lxc-containers
```

The profile will automatically be loaded after a reboot, because it is sourced
by the file `/etc/apparmor.d/lxc-containers`. Finally, to make container `CN`
use this new `lxc-CN-profile`, add the following line to its configuration
file:

```bash
lxc.aa_profile = lxc-CN-profile
```

### Control Groups 
Control groups (cgroups) are a kernel feature providing hierarchical task
grouping and per-cgroup resource accounting and limits. They are used in
containers to limit block and character device access and to freeze (suspend)
containers. They can be further used to limit memory use and block i/o,
guarantee minimum cpu shares, and to lock containers to specific cpus.

By default, a privileged container CN will be assigned to a cgroup called
`/lxc/CN`. In the case of name conflicts (which can occur when using custom
lxcpaths) a suffix "-n", where n is an integer starting at 0, will be appended
to the cgroup name.

By default, a privileged container CN will be assigned to a cgroup called `CN`
under the cgroup of the task which started the container, for instance
`/usr/1000.user/1.session/CN`. The container root will be given group
ownership of the directory (but not all files) so that it is allowed to create
new child cgroups.

As of Ubuntu 14.04, LXC uses the cgroup manager (cgmanager) to administer
cgroups. The cgroup manager receives D-Bus requests over the Unix socket
`/sys/fs/cgroup/cgmanager/sock`. To facilitate safe nested containers, the
line


```bash
lxc.mount.auto = cgroup
```

can be added to the container configuration causing the
`/sys/fs/cgroup/cgmanager` directory to be bind-mounted into the container.
The container in turn should start the cgroup management proxy (done by
default if the cgmanager package is installed in the container) which will
move the `/sys/fs/cgroup/cgmanager` directory to
`/sys/fs/cgroup/cgmanager.lower`, then start listening for requests to proxy
on its own socket `/sys/fs/cgroup/cgmanager/sock`. The host cgmanager will
ensure that nested containers cannot escape their assigned cgroups or make
requests for which they are not authorized.

### Cloning 
For rapid provisioning, you may wish to customize a canonical container
according to your needs and then make multiple copies of it. This can be done
with the `lxc-clone` program.

Clones are either snapshots or copies of another container. A copy is a new
container copied from the original, and takes as much space on the host as the
original. A snapshot exploits the underlying backing store's snapshotting
ability to make a copy-on-write container referencing the first. Snapshots can
be created from btrfs, LVM, zfs, and directory backed containers. Each backing
store has its own peculiarities - for instance, LVM containers which are not
thinpool-provisioned cannot support snapshots of snapshots; zfs containers
with snapshots cannot be removed until all snapshots are released; LVM
containers must be more carefully planned as the underlying filesystem may not
support growing; btrfs does not suffer any of these shortcomings, but suffers
from reduced fsync performance causing dpkg and apt to be slower.

Snapshots of directory-packed containers are created using the overlay
filesystem. For instance, a privileged directory-backed container C1 will have
its root filesystem under `/var/lib/lxc/C1/rootfs`. A snapshot clone of C1
called C2 will be started with C1's rootfs mounted readonly under
`/var/lib/lxc/C2/delta0`. Importantly, in this case C1 should not be allowed
to run or be removed while C2 is running. It is advised instead to consider C1
a *canonical* base container, and to only use its snapshots.

Given an existing container called C1, a copy can be created using:


```bash
sudo lxc-clone -o C1 -n C2
```

A snapshot can be created using:


```bash
sudo lxc-clone -s -o C1 -n C2
```

See the lxc-clone manpage for more information.

#### Snapshots
To more easily support the use of snapshot clones for iterative container
development, LXC supports *snapshots*. When working on a container C1, before
making a potentially dangerous or hard-to-revert change, you can create a
snapshot


```bash
sudo lxc-snapshot -n C1
```

which is a snapshot-clone called 'snap0' under /var/lib/lxcsnaps or
\$HOME/.local/share/lxcsnaps. The next snapshot will be called 'snap1', etc.
Existing snapshots can be listed using `lxc-snapshot -L -n C1`, and a snapshot
can be restored - erasing the current C1 container - using
`lxc-snapshot -r snap1 -n C1`. After the restore command, the snap1 snapshot
continues to exist, and the previous C1 is erased and replaced with the snap1
snapshot.

Snapshots are supported for btrfs, lvm, zfs, and overlayfs containers. If
lxc-snapshot is called on a directory-backed container, an error will be
logged and the snapshot will be created as a copy-clone. The reason for this
is that if the user creates an overlayfs snapshot of a directory-backed
container and then makes changes to the directory-backed container, then the
original container changes will be partially reflected in the snapshot. If
snapshots of a directory backed container C1 are desired, then an overlayfs
clone of C1 should be created, C1 should not be touched again, and the
overlayfs clone can be edited and snapshotted at will, as such


```bash
lxc-clone -s -o C1 -n C2
lxc-start -n C2 -d # make some changes
lxc-stop -n C2
lxc-snapshot -n C2
lxc-start -n C2 # etc
```

#### Ephemeral Containers
While snapshots are useful for longer-term incremental development of images,
ephemeral containers utilize snapshots for quick, single-use throwaway
containers. Given a base container C1, you can start an ephemeral container
using


```bash
lxc-start-ephemeral -o C1
```

The container begins as a snapshot of C1. Instructions for logging into the
container will be printed to the console. After shutdown, the ephemeral
container will be destroyed. See the lxc-start-ephemeral manual page for more
options.

### Lifecycle management hooks 
Beginning with Ubuntu 12.10, it is possible to define hooks to be executed at
specific points in a container's lifetime:

-   Pre-start hooks are run in the host's namespace before the container ttys,
    consoles, or mounts are up. If any mounts are done in this hook, they
    should be cleaned up in the post-stop hook.

-   Pre-mount hooks are run in the container's namespaces, but before the root
    filesystem has been mounted. Mounts done in this hook will be
    automatically cleaned up when the container shuts down.

-   Mount hooks are run after the container filesystems have been mounted, but
    before the container has called `pivot_root` to change its
    root filesystem.

-   Start hooks are run immediately before executing the container's init.
    Since these are executed after pivoting into the container's filesystem,
    the command to be executed must be copied into the container's filesystem.

-   Post-stop hooks are executed after the container has been shut down.

If any hook returns an error, the container's run will be aborted. Any
*post-stop* hook will still be executed. Any output generated by the script
will be logged at the debug priority.

Please see the lxc.container.conf manual page for the configuration file
format with which to specify hooks. Some sample hooks are shipped with the lxc
package to serve as an example of how to write and use such hooks.

### Consoles 
Containers have a configurable number of consoles. One always exists on the
container's `/dev/console`. This is shown on the terminal from which you ran
`lxc-start`, unless the *-d* option is specified. The output on `/dev/console`
can be redirected to a file using the *-c console-file* option to `lxc-start`.
The number of extra consoles is specified by the `lxc.tty` variable, and is
usually set to 4. Those consoles are shown on `/dev/ttyN` (for 1 &lt;= N &lt;=
4). To log into console 3 from the host, use:


```bash
sudo lxc-console -n container -t 3
```

or if the *-t N* option is not specified, an unused console will be
automatically chosen. To exit the console, use the escape sequence Ctrl-a q.
Note that the escape sequence does not work in the console resulting from
`lxc-start` without the *-d* option.

Each container console is actually a Unix98 pty in the host's (not the
guest's) pty mount, bind-mounted over the guest's `/dev/ttyN` and
`/dev/console`. Therefore, if the guest unmounts those or otherwise tries to
access the actual character device `4:N`, it will not be serving getty to the
LXC consoles. (With the default settings, the container will not be able to
access that character device and getty will therefore fail.) This can easily
happen when a boot script blindly mounts a new `/dev`.

### Troubleshooting 
#### Logging
If something goes wrong when starting a container, the first step should be to
get full logging from LXC:


```bash
sudo lxc-start -n C1 -l trace -o debug.out
```

This will cause lxc to log at the most verbose level, `trace`, and to output
log information to a file called 'debug.out'. If the file `debug.out` already
exists, the new log information will be appended.

#### Monitoring container status 
Two commands are available to monitor container state changes. `lxc-monitor`
monitors one or more containers for any state changes. It takes a container
name as usual with the *-n* option, but in this case the container name can be
a posix regular expression to allow monitoring desirable sets of containers.
`lxc-monitor` continues running as it prints container changes. `lxc-wait`
waits for a specific state change and then exits. For instance,


```bash
sudo lxc-monitor -n cont[0-5]*
```

would print all state changes to any containers matching the listed regular
expression, whereas


```bash
sudo lxc-wait -n cont1 -s 'STOPPED|FROZEN'
```

will wait until container cont1 enters state STOPPED or state FROZEN and then
exit.

#### Attach
As of Ubuntu 14.04, it is possible to attach to a container's namespaces. The
simplest case is to simply do


```bash
sudo lxc-attach -n C1
```

which will start a shell attached to C1's namespaces, or, effectively inside
the container. The attach functionality is very flexible, allowing attaching
to a subset of the container's namespaces and security context. See the manual
page for more information.

#### Container init verbosity
If LXC completes the container startup, but the container init fails to
complete (for instance, no login prompt is shown), it can be useful to request
additional verbosity from the init process. For an upstart container, this
might be:


```bash
sudo lxc-start -n C1 /sbin/init loglevel=debug
```

You can also start an entirely different program in place of init, for
instance


```bash
sudo lxc-start -n C1 /bin/bash
sudo lxc-start -n C1 /bin/sleep 100
sudo lxc-start -n C1 /bin/cat /proc/1/status
```

### LXC API 
Most of the LXC functionality can now be accessed through an API exported by
`liblxc` for which bindings are available in several languages, including
Python, lua, ruby, and go.

Below is an example using the python bindings (which are available in the
python3-lxc package) which creates and starts a container, then waits until it
has been shut down:

```bash
# sudo python3
Python 3.2.3 (default, Aug 28 2012, 08:26:03)
[GCC 4.7.1 20120814 (prerelease)] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import lxc
__main__:1: Warning: The python-lxc API isn't yet stable and may change at any p
oint in the future.
>>> c=lxc.Container("C1")
>>> c.create("ubuntu")
True
>>> c.start()
True
>>> c.wait("STOPPED")
True
```

### Security 
A namespace maps ids to resources. By not providing a container any id with
which to reference a resource, the resource can be protected. This is the
basis of some of the security afforded to container users. For instance, IPC
namespaces are completely isolated. Other namespaces, however, have various
*leaks* which allow privilege to be inappropriately exerted from a container
into another container or to the host.

By default, LXC containers are started under a Apparmor policy to restrict
some actions. The details of AppArmor integration with lxc are in section
[Apparmor]. Unprivileged containers go further by mapping root in the
container to an unprivileged host userid. This prevents access to `/proc` and
`/sys` files representing host resources, as well as any other files owned by
root on the host.

#### Exploitable system calls 
It is a core container feature that containers share a kernel with the host.
Therefore if the kernel contains any exploitable system calls the container
can exploit these as well. Once the container controls the kernel it can fully
control any resource known to the host.

Since Ubuntu 12.10 (Quantal) a container can also be constrained by a seccomp
filter. Seccomp is a new kernel feature which filters the system calls which
may be used by a task and its children. While improved and simplified policy
management is expected in the near future, the current policy consists of a
simple whitelist of system call numbers. The policy file begins with a version
number (which must be 1) on the first line and a policy type (which must be
'whitelist') on the second line. It is followed by a list of numbers, one per
line.

In general to run a full distribution container a large number of system calls
will be needed. However for application containers it may be possible to
reduce the number of available system calls to only a few. Even for system
containers running a full distribution security gains may be had, for instance
by removing the 32-bit compatibility system calls in a 64-bit container. See
the lxc.container.conf manual page for details of how to configure a container
to use seccomp. By default, no seccomp policy is loaded.

### Resources 
-   The DeveloperWorks article [LXC: Linux container tools] was an early
    introduction to the use of containers.

-   The [Secure Containers Cookbook] demonstrated the use of security modules
    to make containers more secure.

-   Manual pages referenced above can be found at:

```bash
    capabilities
    lxc.conf
```

-   The upstream LXC project is hosted at [linuxcontainers.org].

-   LXC security issues are listed and discussed at [the LXC Security wiki
    page]

-   For more on namespaces in Linux, see: S. Bhattiprolu, E. W. Biederman, S.
    E. Hallyn, and D. Lezcano. Virtual Servers and Check- point/Restart in
    Mainstream Linux. SIGOPS Operating Systems Review, 42(5), 2008.

  [Virtual Machine Viewer]: #libvirt-virt-viewer
  [Ubuntu Installation Guide]: https://help.ubuntu.com/&distro-rev-short;/installation-guide/
  [Cloud images and uvtool]: #cloud-images-and-uvtool
  [libvirt doc]: http://libvirt.org/uri.html
  [offline migration]: https://libvirt.org/migration.html#offline
  [live migration]: https://libvirt.org/migration.html
  [postcopy migration]: http://wiki.qemu.org/Features/PostCopyLiveMigration
  [Ubuntu Wiki]: https://wiki.ubuntu.com/QemuKVMMigration
  [???]: #openssh-server
  [KVM]: http://www.linux-kvm.org/
  [libvirt home page]: http://libvirt.org/
  [Virtual Machine Manager]: http://virt-manager.org/
  [freenode]: http://freenode.net/
  [Ubuntu Wiki KVM]: https://help.ubuntu.com/community/KVM
  [Ubuntu Wiki Xen]: https://help.ubuntu.com/community/Xen
  [Qemu]: http://wiki.qemu.org/Main_Page
  [command line interface]: http://wiki.qemu.org/download/qemu-doc.html#sec_005finvocation
  [monitor]: http://wiki.qemu.org/download/qemu-doc.html#pcsys_005fmonitor
  [Libvirt]: #libvirt
  [1]: https://wiki.ubuntu.com/QemuKVMMigration#Upgrade_machine_type
  [ubuntu-server at lists.ubuntu.com]: https://lists.ubuntu.com/mailman/listinfo/ubuntu-server
  [upstream documentation]: http://docs.openstack.org/havana/install-guide/install/apt/content/
  [OpenStack Mailing list]: https://launchpad.net/~openstack
  [The OpenStack Wiki search]: http://wiki.openstack.org
  [Launchpad bugs area]: https://bugs.launchpad.net/nova
  [Cloud Computing - Service models]: http://en.wikipedia.org/wiki/Cloud_computing#Service_Models
  [OpenStack Compute]: http://www.openstack.org/software/openstack-compute/
  [OpenStack Image Service]: http://docs.openstack.org/diablo/openstack-compute/starter/content/GlanceMS-d2s21.html
  [OpenStack Object Storage Administration Guide]: http://docs.openstack.org/trunk/openstack-object-storage/admin/content/index.html
  [Installing OpenStack Object Storage on Ubuntu]: http://docs.openstack.org/trunk/openstack-object-storage/admin/content/installing-openstack-object-storage-on-ubuntu.html
  [quite well justified]: http://blog.dustinkirkland.com/2015/09/container-summit-presentation-and-live.html
  [LXC]: https://help.ubuntu.com/lts/serverguide/lxc.html
  [getting started with LXD]: http://github.com/lxc/lxd
  [try out LXD remotely]: http://linuxcontainers.org/lxd/try-it
  [excellent blog series]: https://www.stgraber.org/2016/03/11/lxd-2-0-blog-post-series-012/
  [drive lxd using juju]: https://jujucharms.com/docs/devel/config-LXD
  [ZFS]: http://open-zfs.org
  [online]: https://github.com/lxc/lxd/blob/master/doc/configuration.md
  [subuid(5) manual page]: http://manpages.ubuntu.com/manpages/xenial/en/man5/subuid.5.html
  [the lxc.container.conf(5) manual page]: http://manpages.ubuntu.com/manpages/xenial/en/man5/lxc.container.conf.5.html
  [simplestreams-based]: https://launchpad.net/simplestreams
  [Basic unprivileged usage]: #lxc-unpriv
  [Apparmor]: #lxc-apparmor
  [2]: #bridging
  [LXC: Linux container tools]: https://www.ibm.com/developerworks/linux/library/l-lxc-containers/
  [Secure Containers Cookbook]: http://www.ibm.com/developerworks/linux/library/l-lxc-security/index.html
  [linuxcontainers.org]: http://linuxcontainers.org
  [the LXC Security wiki page]: http://wiki.ubuntu.com/LxcSecurity
