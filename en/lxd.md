Title: LXD

# LXD

LXD (pronounced lex-dee) is the lightervisor, or lightweight container hypervisor. While this claim has been controversial, it has been quite well justified based on the original academic paper. It also nicely distinguishes LXD from LXC.

LXC (lex-see) is a program which creates and administers "containers" on a local system. It also provides an API to allow higher level managers, such as LXD, to administer containers. In a sense, one could compare LXC to QEMU, while comparing LXD to libvirt.

The LXC API deals with a 'container'. The LXD API deals with 'remotes', which serve images and containers. This extends the LXC functionality over the network, and allows concise management of tasks like container migration and container image publishing.

LXD uses LXC under the covers for some container management tasks. However, it keeps its own container configuration information and has its own conventions, so that it is best not to use classic LXC commands by hand with LXD containers. This document will focus on how to configure and administer LXD on Ubuntu systems.

## Online Resources

There is excellent documentation for getting started with LXD in the online LXD README. There is also an online server allowing you to try out LXD remotely. Stephane Graber also has an excellent blog series on LXD 2.0. Finally, there is great documentation on how to drive lxd using juju.

This document will offer an Ubuntu Server-specific view of LXD, focusing on administration.

## Installation

LXD is pre-installed on Ubuntu Server cloud images. On other systems, the lxd package can be installed using:

```bash
sudo apt install lxd
```

This will install LXD as well as the recommended dependencies, including the LXC library and lxcfs.

## Kernel preparation

In general, Ubuntu 16.04 should have all the desired features enabled by default. One exception to this is that in order to enable swap accounting the boot argument swapaccount=1 must be set. This can be done by appending it to the GRUB_CMDLINE_LINUX_DEFAULT=variable in /etc/default/grub, then running 'update-grub' as root and rebooting.

## Configuration
By default, LXD is installed listening on a local UNIX socket, which members of group LXD can talk to. It has no trust password setup. And it uses the filesystem at /var/lib/lxd to store containers. To configure LXD with different settings, use lxd init. This will allow you to choose:

Directory or ZFS container backend. If you choose ZFS, you can choose which block devices to use, or the size of a file to use as backing store.

## Availability over the network

A 'trust password' used by remote clients to vouch for their client certificate

You must run 'lxd init' as root. 'lxc' commands can be run as any user who is member of group lxd. If user joe is not a member of group 'lxd', you may run:

```bash
adduser joe lxd
```

as root to change it. The new membership will take effect on the next login, or after running 'newgrp lxd' from an existing login.

For more information on server, container, profile, and device configuration, please refer to the definitive configuration provided with the source code, which can be found online

## Creating your first container

This section will describe the simplest container tasks.

### Creating a container

Every new container is created based on either an image, an existing container, or a container snapshot. At install time, LXD is configured with the following image servers:

 1. ubuntu: this serves official Ubuntu server cloud image releases.

 1. ubuntu-daily: this serves official Ubuntu server cloud images of the daily development releases.

 1. images: this is a default-installed alias for images.linuxcontainers.org. This is serves classical lxc images built using the same images which the LXC 'download' template uses. This includes various distributions and minimal custom-made Ubuntu images. This is not the recommended server for Ubuntu images.

The command to create and start a container is

```bash
lxc launch remote:image containername
```

Images are identified by their hash, but are also aliased. The 'ubuntu' server knows many aliases such as '16.04' and 'xenial'. A list of all images available from the Ubuntu Server can be seen using:

```bash
lxc image list ubuntu:
```

To see more information about a particular image, including all the aliases it is known by, you can use:

```bash
lxc image info ubuntu:xenial
```

You can generally refer to an Ubuntu image using the release name ('xenial') or the release number (16.04). In addition, 'lts' is an alias for the latest supported LTS release. To choose a different architecture, you can specify the desired architecture:

```bash
lxc image info ubuntu:lts/arm64
```

Now, let's start our first container:

```bash
lxc launch ubuntu:xenial x1
```
This will download the official current Xenial cloud image for your current architecture, then create a container using that image, and finally start it. Once the command returns, you can see it using:

```bash
lxc list
lxc info x1
```

and open a shell in it using:

```
lxc exec x1 bash
```

The try-it page gives a full synopsis of the commands you can use to administer containers.

Now that the 'xenial' image has been downloaded, it will be kept in sync until no new containers have been created based on it for (by default) 10 days. After that, it will be deleted.

## LXD Server Configuration
By default, LXD is socket activated and configured to listen only on a local UNIX socket. While LXD may not be running when you first look at the process listing, any LXC command will start it up. For instance:

```bash
lxc list
```

This will create your client certificate and contact the LXD server for a list of containers. To make the server accessible over the network you can set the http port using:

```bash
lxc config set core.https_address :8443
```

This will tell LXD to listen to port 8843 on all addresses.

### Authentication

By default, LXD will allow all members of group 'lxd' (which by default includes all members of group admin) to talk to it over the UNIX socket. Communication over the network is authorized using server and client certificates.

Before client c1 wishes to use remote r1, r1 must be registered using:

```bash
lxc remote add r1 r1.example.com:8443
```

The fingerprint of r1's certificate will be shown, to allow the user at c1 to reject a false certificate. The server in turn will verify that c1 may be trusted in one of two ways. The first is to register it in advance from any already-registered client, using:

```bash
lxc config trust add r1 certfile.crt
```

Now when the client adds r1 as a known remote, it will not need to provide a password as it is already trusted by the server.

The other is to configure a 'trust password' with r1, either at initial configuration using 'lxd init', or after the fact using

```bash
lxc config set core.trust_password PASSWORD
```

The password can then be provided when the client registers r1 as a known remote.

### Backing store

LXD supports several backing stores. The recommended backing store is ZFS, however this is not available on all platforms. Supported backing stores include:

 1. ext4: this is the default, and easiest to use. With an ext4 backing store, containers and images are simply stored as directories on the host filesystem. Launching new containers requires copying a whole filesystem, and 10 containers will take up 10 times as much space as one container.

 1. ZFS: if ZFS is supported on your architecture (amd64, arm64, or ppc64le), you can set LXD up to use it using 'lxd init'. If you already have a ZFS pool configured, you can tell LXD to use it by setting the zfs_pool_name configuration key:
      lxc config set storage.zfs_pool_name lxd
With ZFS, launching a new container is fast because the filesystem starts as a copy on write clone of the images' filesystem. Note that unless the container is privileged (see below) LXD will need to change ownership of all files before the container can start, however this is fast and change very little of the actual filesystem data.

 1. Btrfs: btrfs can be used with many of the same advantages as ZFS. To use BTRFS as a LXD backing store, simply mount a Btrfs filesystem under /var/lib/lxd. LXD will detect this and exploit the Btrfs subvolume feature whenever launching a new container or snapshotting a container.

 1. LVM: To use a LVM volume group called 'lxd', you may tell LXD to use that for containers and images using the command:
	lxc config set storage.lvm_vg_name lxd

When launching a new container, its rootfs will start as a lv clone. It is immediately mounted so that the file uids can be shifted, then unmounted. Container snapshots also are created as lv snapshots.

## Container configuration
Containers are configured according to a set of profiles, described in the next section, and a set of container-specific configuration. Profiles are applied first, so that container specific configuration can override profile configuration.

Container configuration includes properties like the architecture, limits on resources such as CPU and RAM, security details including apparmor restriction overrides, and devices to apply to the container.

Devices can be of several types, including UNIX character, UNIX block, network interface, or 'disk'. In order to insert a host mount into a container, a 'disk' device type would be used. For instance, to mount /opt in container c1 at /opt, you could use:

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

to edit the whole of c1's configuration in your specified `$EDITOR`. Comments at the top of the configuration will show examples of correct syntax to help administrators hit the ground running. If the edited configuration is not valid when the `$EDITOR` is exited, then `$EDITOR` will be restarted.

## Profiles

Profiles are named collections of configurations which may be applied to more than one container. For instance, all containers created with `lxc launch`, by default, include the 'default' profile, which provides a network interface `eth0`.

To mask a device which would be inherited from a profile but which should not be in the final container, define a device by the same name but of type 'none':

```bash
lxc config device add c1 eth1 none
```

## Nesting

Containers all share the same host kernel. This means that there is always an inherent trade-off between features exposed to the container and host security from malicious containers. Containers by default are therefore restricted from features needed to nest child containers. In order to run lxc or lxd containers under a lxd container, the 'security.nesting' feature must be set to true:

```bash
lxc config set container1 security.nesting true
```

Once this is done, container1 will be able to start sub-containers.

In order to run unprivileged (the default in LXD) containers nested under an unprivileged container, you will need to ensure a wide enough UID mapping. Please see the 'UID mapping' section below.

### Docker

In order to facilitate running docker containers inside a LXD container, a 'docker' profile is provided. To launch a new container with the docker profile, you can run:

```bash
lxc launch xenial container1 -p default -p docker
```

Note that currently the docker package in Ubuntu 16.04 is patched to facilitate running in a container. This support is expected to land upstream soon.

Note that 'cgroup namespace' support is also required. This is available in the 16.04 kernel as well as in the 4.6 upstream source.

## Limits
LXD supports flexible constraints on the resources which containers can consume. The limits come in the following categories:

 - CPU: limit cpu available to the container in several ways.

 - Disk: configure the priority of I/O requests under load

 - RAM: configure memory and swap availability

 - Network: configure the network priority under load

 - Processes: limit the number of concurrent processes in the container.

For a full list of limits known to LXD, see the configuration documentation.

## UID mappings and Privileged containers

By default, LXD creates unprivileged containers. This means that root in the container is a non-root UID on the host. It is privileged against the resources owned by the container, but unprivileged with respect to the host, making root in a container roughly equivalent to an unprivileged user on the host. (The main exception is the increased attack surface exposed through the system call interface)

Briefly, in an unprivileged container, 65536 UIDs are 'shifted' into the container. For instance, UID 0 in the container may be 100000 on the host, UID 1 in the container is 100001, etc, up to 165535. The starting value for UIDs and GIDs, respectively, is determined by the 'root' entry the /etc/subuid and /etc/subgid files. (See the subuid(5) manual page.

It is possible to request a container to run without a UID mapping by setting the security.privileged flag to true:

```bash
lxc config set c1 security.privileged true
```

Note however that in this case the root user in the container is the root user on the host.

## Apparmor

LXD confines containers by default with an apparmor profile which protects containers from each other and the host from containers. For instance this will prevent root in one container from signaling root in another container, even though they have the same uid mapping. It also prevents writing to dangerous, un-namespaced files such as many sysctls and /proc/sysrq-trigger.

If the apparmor policy for a container needs to be modified for a container c1, specific apparmor policy lines can be added in the 'raw.apparmor' configuration key.

## Seccomp

All containers are confined by a default seccomp policy. This policy prevents some dangerous actions such as forced umounts, kernel module loading and unloading, kexec, and the open_by_handle_at system call. The seccomp configuration cannot be modified, however a completely different seccomp policy - or none - can be requested using raw.lxc (see below).

## Raw LXC configuration

LXD configures containers for the best balance of host safety and container usability. Whenever possible it is highly recommended to use the defaults, and use the LXD configuration keys to request LXD to modify as needed. Sometimes, however, it may be necessary to talk to the underlying lxc driver itself. This can be done by specifying LXC configuration items in the 'raw.lxc' LXD configuration key. These must be valid items as documented in the lxc.container.conf(5) manual page.

## Images and containers

LXD is image based. When you create your first container, you will generally do so using an existing image. LXD comes pre-configured with three default image remotes:

ubuntu: This is a simplestreams-based remote serving released ubuntu cloud images.

ubuntu-daily: This is another simplestreams based remote which serves 'daily' ubuntu cloud images. These provide quicker but potentially less stable images.

images: This is a remote publishing best-effort container images for many distributions, created using community-provided build scripts.

To view the images available on one of these servers, you can use:


```bash
lxcimage list ubuntu:
```

Most of the images are known by several aliases for easier reference. To see the full list of aliases, you can use


```bash
lxcimage alias list images:
```

Any alias or image fingerprint can be used to specify how to create the new container. For instance, to create an amd64 Ubuntu 14.04 container, some options are:


```bash
lxc launch ubuntu:14.04 trusty1
lxc launch ubuntu:trusty trusty1
lxc launch ubuntu:trusty/amd64 trusty1
lxc launch ubuntu:lts trusty1
```

The 'lts' alias always refers to the latest released LTS image.


### Snapshots

Containers can be renamed and live-migrated using the 'lxc move' command:

```
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

### Publishing images

When a container or container snapshot is ready for consumption by others, it can be published as a new image using;

```
lxc publish u1/YYYY-MM-DD --alias foo-2.0
```

The published image will be private by default, meaning that LXD will not allow clients without a trusted certificate to see them. If the image is safe for public viewing (i.e. contains no private information), then the 'public' flag can be set, either at publish time using:

```
lxc publish u1/YYYY-MM-DD --alias foo-2.0 public=true
```

or after the fact using

```bash
lxc image edit foo-2.0
```
and changing the value of the public field.

### Image export and import

Image can be exported as, and imported from, tarballs:

```bash
lxc image export foo-2.0 foo-2.0.tar.gz
lxc image import foo-2.0.tar.gz --alias foo-2.0 --public
```

##Troubleshooting

To view debug information about LXD itself, on a systemd based host use

```bash
journalctl -u LXD
```

On an Upstart-based system, you can find the log in /var/log/upstart/lxd.log. To make LXD provide much more information about requests it is serving, add '--debug' to LXD's arguments. In systemd, append '--debug' to the 'ExecStart=' line in /lib/systemd/system/lxd.service. In Upstart, append it to the exec /usr/bin/lxd line in /etc/init/lxd.conf.

Container logfiles for container c1 may be seen using:

```
lxc info c1 --show-log
```

The configuration file which was used may be found under /var/log/lxd/c1/lxc.conf while apparmor profiles can be found in /var/lib/lxd/security/apparmor/profiles/c1 and seccomp profiles in /var/lib/lxd/security/seccomp/c1.
