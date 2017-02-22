# Installation

This chapter provides a quick overview of installing Ubuntu DISTRO-REV Server
Edition. For more detailed instructions, please refer to the [Ubuntu
Installation Guide].

# Preparing to Install

This section explains various aspects to consider before starting the
installation.

## System Requirements

Ubuntu DISTRO-REV Server Edition supports three (3) major architectures: Intel
x86, AMD64 and ARM. The table below lists recommended hardware specifications.
Depending on your needs, you might manage with less than this. However, most
users risk being frustrated if they ignore these suggestions.

+----------------+----------------+----------------+----------------+--------------------+
| Install Type   | CPU            | RAM            | Hard Drive     |
|                |                |                | Space          |
+================+================+================+================+====================+
| Server         | 1 gigahertz    | 512 megabytes  | 1 gigabyte     | 1.75 gigabytes     |
| (Standard)     |                |                |                |                    |
+----------------+----------------+----------------+----------------+--------------------+
| Server         | 300 megahertz  | 192 megabytes  | 700 megabytes  | 1.4 gigabytes      |
| (Minimal)      |                |                |                |                    |
+----------------+----------------+----------------+----------------+--------------------+

: Recommended Minimum Requirements

The Server Edition provides a common base for all sorts of server
applications. It is a minimalist design providing a platform for the desired
services, such as file/print services, web hosting, email hosting, etc.

## Server and Desktop Differences {#intro-server-differences}

There are a few differences between the *Ubuntu Server Edition* and the
*Ubuntu Desktop Edition*. It should be noted that both editions use the same
apt repositories, making it just as easy to install a *server* application on
the Desktop Edition as it is on the Server Edition.

The differences between the two editions are the lack of an X window
environment in the Server Edition and the installation process.

### Kernel Differences: {#intro-kernel-diffs}

Ubuntu version 10.10 and prior, actually had different kernels for the server
and desktop editions. Ubuntu no longer has separate -server and -generic
kernel flavors. These have been merged into a single -generic kernel flavor to
help reduce the maintenance burden over the life of the release.

> **Note**
>
> When running a 64-bit version of Ubuntu on 64-bit processors you are not
> limited by memory addressing space.

To see all kernel configuration options you can look through
`/boot/config-LINUX-KERNEL-VERSION-server`. Also, [Linux Kernel in a Nutshell]
is a great resource on the options available.

## Backing Up

-   Before installing Ubuntu Server Edition you should make sure all data on
    the system is backed up. See [???] for backup options.

    If this is not the first time an operating system has been installed on
    your computer, it is likely you will need to re-partition your disk to
    make room for Ubuntu.

    Any time you partition your disk, you should be prepared to lose
    everything on the disk should you make a mistake or something goes wrong
    during partitioning. The programs used in installation are quite reliable,
    most have seen years of use, but they also perform destructive actions.

# Installing from CD

The basic steps to install Ubuntu Server Edition from CD are the same as those
for installing any operating system from CD. Unlike the *Desktop Edition*, the
*Server Edition* does not include a graphical installation program. The Server
Edition uses a console menu based process instead.

-   Download and burn the appropriate ISO file from the [Ubuntu web site].

-   Boot the system from the CD-ROM drive.

-   At the boot prompt you will be asked to select a language.

-   From the main boot menu there are some additional options to install
    Ubuntu Server Edition. You can install a basic Ubuntu Server, check the
    CD-ROM for defects, check the system's RAM, boot from first hard disk, or
    rescue a broken system. The rest of this section will cover the basic
    Ubuntu Server install.

-   The installer asks which language it should use. Afterwards, you are asked
    to select your location.

-   Next, the installation process begins by asking for your keyboard layout.
    You can ask the installer to attempt auto-detecting it, or you can select
    it manually from a list.

-   The installer then discovers your hardware configuration, and configures
    the network settings using DHCP. If you do not wish to use DHCP at the
    next screen choose "Go Back", and you have the option to "Configure the
    network manually".

-   Next, the installer asks for the system's hostname.

-   A new user is set up; this user will have *root* access through the
    sudo utility.

-   After the user settings have been completed, you will be asked if you want
    to encrypt your `home` directory.

-   Next, the installer asks for the system's Time Zone.

-   You can then choose from several options to configure the hard
    drive layout. Afterwards you are asked which disk to install to. You may
    get confirmation prompts before rewriting the partition table or setting
    up LVM depending on disk layout. If you choose LVM, you will be asked for
    the size of the root logical volume. For advanced disk options see
    [Advanced Installation].

-   The Ubuntu base system is then installed.

-   The next step in the installation process is to decide how you want to
    update the system. There are three options:

    -   *No automatic updates*: this requires an administrator to log into the
        machine and manually install updates.

    -   *Install security updates automatically*: this will install the
        unattended-upgrades package, which will install security updates
        without the intervention of an administrator. For more details see
        [???][1].

    -   *Manage the system with Landscape*: Landscape is a paid service
        provided by Canonical to help manage your Ubuntu machines. See the
        [Landscape] site for details.

-   You now have the option to install, or not install, several package tasks.
    See [Package Tasks] for details. Also, there is an option to launch
    aptitude to choose specific packages to install. For more information see
    [???][2].

-   Finally, the last step before rebooting is to set the clock to UTC.

> **Note**
>
> If at any point during installation you are not satisfied by the default
> setting, use the "Go Back" function at any prompt to be brought to a
> detailed installation menu that will allow you to modify the default
> settings.

At some point during the installation process you may want to read the help
screen provided by the installation system. To do this, press F1.

Once again, for detailed instructions see the [Ubuntu Installation Guide].

## Package Tasks {#install-tasks}

During the Server Edition installation you have the option of installing
additional packages from the CD. The packages are grouped by the type of
service they provide.

-   DNS server: Selects the BIND DNS server and its documentation.

-   LAMP server: Selects a ready-made Linux/Apache/MySQL/PHP server.

-   Mail server: This task selects a variety of packages useful for a general
    purpose mail server system.

-   OpenSSH server: Selects packages needed for an OpenSSH server.

-   PostgreSQL database: This task selects client and server packages for the
    PostgreSQL database.

-   Print server: This task sets up your system to be a print server.

-   Samba File server: This task sets up your system to be a Samba file
    server, which is especially suitable in networks with both Windows and
    Linux systems.

-   Tomcat Java server: Installs Apache Tomcat and needed dependencies.

-   Virtual Machine host: Includes packages needed to run KVM
    virtual machines.

-   Manually select packages: Executes aptitude allowing you to individually
    select packages.

Installing the package groups is accomplished using the tasksel utility. One
of the important differences between Ubuntu (or Debian) and other GNU/Linux
distribution is that, when installed, a package is also configured to
reasonable defaults, eventually prompting you for additional required
information. Likewise, when installing a task, the packages are not only
installed, but also configured to provided a fully integrated service.

Once the installation process has finished you can view a list of available
tasks by entering the following from a terminal prompt:

    tasksel --list-tasks

> **Note**
>
> The output will list tasks from other Ubuntu based distributions such as
> Kubuntu and Edubuntu. Note that you can also invoke the `tasksel` command by
> itself, which will bring up a menu of the different tasks available.

You can view a list of which packages are installed with each task using the
*--task-packages* option. For example, to list the packages installed with the
*DNS Server* task enter the following:

    tasksel --task-packages dns-server

The output of the command should list:

    bind9-doc 
    bind9utils 
    bind9

If you did not install one of the tasks during the installation process, but
for example you decide to make your new LAMP server a DNS server as well,
simply insert the installation CD and from a terminal:

    sudo tasksel install dns-server

# Upgrading {#installing-upgrading}

There are several ways to upgrade from one Ubuntu release to another. This
section gives an overview of the recommended upgrade method.

## do-release-upgrade

The recommended way to upgrade a Server Edition installation is to use the
do-release-upgrade utility. Part of the *update-manager-core* package, it does
not have any graphical dependencies and is installed by default.

Debian based systems can also be upgraded by using `apt dist-upgrade`.
However, using do-release-upgrade is recommended because it has the ability to
handle system configuration changes sometimes needed between releases.

To upgrade to a newer release, from a terminal prompt enter:

    do-release-upgrade

It is also possible to use do-release-upgrade to upgrade to a development
version of Ubuntu. To accomplish this use the *-d* switch:

    do-release-upgrade -d

> **Warning**
>
> Upgrading to a development release is *not* recommended for production
> environments.

For further stability of a LTS release there is a slight change in behaviour
if you are currently running a LTS version. LTS systems are only automatically
considered for an upgrade to the next LTS via do-release-upgrade with the
first point release. So for example 14.04 will only upgrade once 16.04.1 is
released. If you want to update before, e.g. on a subset of machines to
evaluate the LTS upgrade for your setup the same argument as an upgrade to a
dev release has to be used via the *-d* switch.

# Advanced Installation

## Software RAID

Redundant Array of Independent Disks "RAID" is a method of using multiple
disks to provide different balances of increasing data reliability and/or
increasing input/output performance, depending on the RAID level being used.
RAID is implemented in either software (where the operating system knows about
both drives and actively maintains both of them) or hardware (where a special
controller makes the OS think there's only one drive and maintains the drives
'invisibly').

The RAID software included with current versions of Linux (and Ubuntu) is
based on the 'mdadm' driver and works very well, better even than many
so-called 'hardware' RAID controllers. This section will guide you through
installing Ubuntu Server Edition using two RAID1 partitions on two physical
hard drives, one for */* and another for *swap*.

### Partitioning {#raid-partitioning}

Follow the installation steps until you get to the *Partition disks* step,
then:

Select *Manual* as the partition method.

Select the first hard drive, and agree to *"Create a new empty partition table
on this device?"*.

Repeat this step for each drive you wish to be part of the RAID array.

Select the *"FREE SPACE"* on the first drive then select *"Create a new
partition"*.

Next, select the *Size* of the partition. This partition will be the *swap*
partition, and a general rule for swap size is twice that of RAM. Enter the
partition size, then choose *Primary*, then *Beginning*.

> **Note**
>
> A swap partition size of twice the available RAM capacity may not always be
> desirable, especially on systems with large amounts of RAM. Calculating the
> swap partition size for servers is highly dependent on how the system is
> going to be used.

Select the *"Use as:"* line at the top. By default this is *"Ext4 journaling
file system"*, change that to *"physical volume for RAID"* then *"Done setting
up partition"*.

For the */* partition once again select *"Free Space"* on the first drive then
*"Create a new partition"*.

Use the rest of the free space on the drive and choose *Continue*, then
*Primary*.

As with the swap partition, select the *"Use as:"* line at the top, changing
it to *"physical volume for RAID"*. Also select the *"Bootable flag:"* line to
change the value to *"on"*. Then choose *"Done setting up partition"*.

Repeat steps three through eight for the other disk and partitions.

### RAID Configuration

With the partitions setup the arrays are ready to be configured:

Back in the main "Partition Disks" page, select *"Configure Software RAID"* at
the top.

Select *"yes"* to write the changes to disk.

Choose *"Create MD device"*.

For this example, select *"RAID1"*, but if you are using a different setup
choose the appropriate type (RAID0 RAID1 RAID5).

> **Note**
>
> In order to use *RAID5* you need at least *three* drives. Using RAID0 or
> RAID1 only *two* drives are required.

Enter the number of active devices *"2"*, or the amount of hard drives you
have, for the array. Then select *"Continue"*.

Next, enter the number of spare devices *"0"* by default, then choose
*"Continue"*.

Choose which partitions to use. Generally they will be sda1, sdb1, sdc1, etc.
The numbers will usually match and the different letters correspond to
different hard drives.

For the *swap* partition choose *sda1* and *sdb1*. Select *"Continue"* to go
to the next step.

Repeat steps *three* through *seven* for the */* partition choosing *sda2* and
*sdb2*.

Once done select *"Finish"*.

### Formatting {#raid-formatting}

There should now be a list of hard drives and RAID devices. The next step is
to format and set the mount point for the RAID devices. Treat the RAID device
as a local hard drive, format and mount accordingly.

Select *"\#1"* under the *"RAID1 device \#0"* partition.

Choose *"Use as:"*. Then select *"swap area"*, then *"Done setting up
partition"*.

Next, select *"\#1"* under the *"RAID1 device \#1"* partition.

Choose *"Use as:"*. Then select *"Ext4 journaling file system"*.

Then select the *"Mount point"* and choose *"/ - the root file system"*.
Change any of the other options as appropriate, then select *"Done setting up
partition"*.

Finally, select *"Finish partitioning and write changes to disk"*.

If you choose to place the root partition on a RAID array, the installer will
then ask if you would like to boot in a *degraded* state. See [Degraded RAID]
for further details.

The installation process will then continue normally.

### Degraded RAID {#raid-degraded}

At some point in the life of the computer a disk failure event may occur. When
this happens, using Software RAID, the operating system will place the array
into what is known as a *degraded* state.

If the array has become degraded, due to the chance of data corruption, by
default Ubuntu Server Edition will boot to *initramfs* after thirty seconds.
Once the initramfs has booted there is a fifteen second prompt giving you the
option to go ahead and boot the system, or attempt manual recover. Booting to
the initramfs prompt may or may not be the desired behavior, especially if the
machine is in a remote location. Booting to a degraded array can be configured
several ways:

-   The dpkg-reconfigure utility can be used to configure the default
    behavior, and during the process you will be queried about additional
    settings related to the array. Such as monitoring, email alerts, etc. To
    reconfigure mdadm enter the following:

        sudo dpkg-reconfigure mdadm

-   The `dpkg-reconfigure mdadm` process will change the
    `/etc/initramfs-tools/conf.d/mdadm` configuration file. The file has the
    advantage of being able to pre-configure the system's behavior, and can
    also be manually edited:

        BOOT_DEGRADED=true

    > **Note**
    >
    > The configuration file can be overridden by using a Kernel argument.

-   Using a Kernel argument will allow the system to boot to a degraded array
    as well:

    -   When the server is booting press Shift to open the Grub menu.

    -   Press e to edit your kernel command options.

    -   Press the down arrow to highlight the kernel line.

    -   Add *"bootdegraded=true"* (without the quotes) to the end of the line.

    -   Press <span class="keycombo">Ctrl+x</span> to boot the system.

Once the system has booted you can either repair the array see
[RAID Maintenance] for details, or copy important data to another machine due
to major hardware failure.

### RAID Maintenance

The mdadm utility can be used to view the status of an array, add disks to an
array, remove disks, etc:

-   To view the status of an array, from a terminal prompt enter:

        sudo mdadm -D /dev/md0

    The *-D* tells mdadm to display *detailed* information about the
    `/dev/md0` device. Replace `/dev/md0` with the appropriate RAID device.

-   To view the status of a disk in an array:

        sudo mdadm -E /dev/sda1

    The output if very similar to the `mdadm -D` command, adjust `/dev/sda1`
    for each disk.

-   If a disk fails and needs to be removed from an array enter:

        sudo mdadm --remove /dev/md0 /dev/sda1

    Change `/dev/md0` and `/dev/sda1` to the appropriate RAID device and disk.

-   Similarly, to add a new disk:

        sudo mdadm --add /dev/md0 /dev/sda1

Sometimes a disk can change to a *faulty* state even though there is nothing
physically wrong with the drive. It is usually worthwhile to remove the drive
from the array then re-add it. This will cause the drive to re-sync with the
array. If the drive will not sync with the array, it is a good indication of
hardware failure.

The `/proc/mdstat` file also contains useful information about the system's
RAID devices:

    cat /proc/mdstat
    Personalities : [linear] [multipath] [raid0] [raid1] [raid6] [raid5] [raid4] [raid10] 
    md0 : active raid1 sda1[0] sdb1[1]
          10016384 blocks [2/2] [UU]
          
    unused devices: <none>

The following command is great for watching the status of a syncing drive:

    watch -n1 cat /proc/mdstat

Press *Ctrl+c* to stop the watch command.

If you do need to replace a faulty drive, after the drive has been replaced
and synced, grub will need to be installed. To install grub on the new drive,
enter the following:

    sudo grub-install /dev/md0

Replace `/dev/md0` with the appropriate array device name.

### Resources {#raid-resources}

The topic of RAID arrays is a complex one due to the plethora of ways RAID can
be configured. Please see the following links for more information:

-   [Ubuntu Wiki Articles on RAID].

-   [Software RAID HOWTO]

-   [Managing RAID on Linux]

## Logical Volume Manager (LVM) {#lvm}

Logical Volume Manger, or *LVM*, allows administrators to create *logical*
volumes out of one or multiple physical hard disks. LVM volumes can be created
on both software RAID partitions and standard partitions residing on a single
disk. Volumes can also be extended, giving greater flexibility to systems as
requirements change.

### Overview {#lvm-overview}

A side effect of LVM's power and flexibility is a greater degree of
complication. Before diving into the LVM installation process, it is best to
get familiar with some terms.

-   *Physical Volume (PV):* physical hard disk, disk partition or software
    RAID partition formatted as LVM PV.

-   *Volume Group (VG):* is made from one or more physical volumes. A VG can
    can be extended by adding more PVs. A VG is like a virtual disk drive,
    from which one or more logical volumes are carved.

-   *Logical Volume (LV):* is similar to a partition in a non-LVM system. A LV
    is formatted with the desired file system (EXT3, XFS, JFS, etc), it is
    then available for mounting and data storage.

### Installation {#lvm-installation}

As an example this section covers installing Ubuntu Server Edition with `/srv`
mounted on a LVM volume. During the initial install only one Physical Volume
(PV) will be part of the Volume Group (VG). Another PV will be added after
install to demonstrate how a VG can be extended.

There are several installation options for LVM, *"Guided - use the entire disk
and setup LVM"* which will also allow you to assign a portion of the available
space to LVM, *"Guided - use entire and setup encrypted LVM"*, or *Manually*
setup the partitions and configure LVM. At this time the only way to configure
a system with both LVM and standard partitions, during installation, is to use
the Manual approach.

Follow the installation steps until you get to the *Partition disks* step,
then:

At the *"Partition Disks* screen choose *"Manual"*.

Select the hard disk and on the next screen choose "yes" to *"Create a new
empty partition table on this device"*.

Next, create standard */boot*, *swap*, and */* partitions with whichever
filesystem you prefer.

For the LVM */srv*, create a new *Logical* partition. Then change *"Use as"*
to *"physical volume for LVM"* then *"Done setting up the partition"*.

Now select *"Configure the Logical Volume Manager"* at the top, and choose
*"Yes"* to write the changes to disk.

For the *"LVM configuration action"* on the next screen, choose *"Create
volume group"*. Enter a name for the VG such as *vg01*, or something more
descriptive. After entering a name, select the partition configured for LVM,
and choose *"Continue"*.

Back at the *"LVM configuration action"* screen, select *"Create logical
volume"*. Select the newly created volume group, and enter a name for the new
LV, for example *srv* since that is the intended mount point. Then choose a
size, which may be the full partition because it can always be extended later.
Choose *"Finish"* and you should be back at the main *"Partition Disks"*
screen.

Now add a filesystem to the new LVM. Select the partition under *"LVM VG vg01,
LV srv"*, or whatever name you have chosen, the choose *Use as*. Setup a file
system as normal selecting */srv* as the mount point. Once done, select *"Done
setting up the partition"*.

Finally, select *"Finish partitioning and write changes to disk"*. Then
confirm the changes and continue with the rest of the installation.

There are some useful utilities to view information about LVM:

-   *pvdisplay:* shows information about Physical Volumes.

-   *vgdisplay:* shows information about Volume Groups.

-   *lvdisplay:* shows information about Logical Volumes.

### Extending Volume Groups {#lvm-extending}

Continuing with *srv* as an LVM volume example, this section covers adding a
second hard disk, creating a Physical Volume (PV), adding it to the volume
group (VG), extending the logical volume `srv` and finally extending the
filesystem. This example assumes a second hard disk has been added to the
system. In this example, this hard disk will be named `/dev/sdb` and we will
use the entire disk as a physical volume (you could choose to create
partitions and use them as different physical volumes)

> **Warning**
>
> Make sure you don't already have an existing `/dev/sdb` before issuing the
> commands below. You could lose some data if you issue those commands on a
> non-empty disk.

First, create the physical volume, in a terminal execute:

    sudo pvcreate /dev/sdb
                    

Now extend the Volume Group (VG):

    sudo vgextend vg01 /dev/sdb

Use vgdisplay to find out the free physical extents - Free PE / size (the size
you can allocate). We will assume a free size of 511 PE (equivalent to 2GB
with a PE size of 4MB) and we will use the whole free space available. Use
your own PE and/or free space.

The Logical Volume (LV) can now be extended by different methods, we will only
see how to use the PE to extend the LV:

    sudo lvextend /dev/vg01/srv -l +511

The *-l* option allows the LV to be extended using PE. The *-L* option allows
the LV to be extended using Meg, Gig, Tera, etc bytes.

Even though you are supposed to be able to *expand* an ext3 or ext4 filesystem
without unmounting it first, it may be a good practice to unmount it anyway
and check the filesystem, so that you don't mess up the day you want to reduce
a logical volume (in that case unmounting first is compulsory).

The following commands are for an *EXT3* or *EXT4* filesystem. If you are
using another filesystem there may be other utilities available.

    sudo umount /srv
    sudo e2fsck -f /dev/vg01/srv

The *-f* option of e2fsck forces checking even if the system seems clean.

Finally, resize the filesystem:

    sudo resize2fs /dev/vg01/srv

Now mount the partition and check its size.

    mount /dev/vg01/srv /srv && df -h /srv

### Resources {#lvm-resources}

-   See the [Ubuntu Wiki LVM Articles].

-   See the [LVM HOWTO] for more information.

-   Another good article is [Managing Disk Space with LVM] on O'Reilly's
    linuxdevcenter.com site.

-   For more information on fdisk see the [fdisk man page].

## iSCSI

The iSCSI protocol can be used to install Ubuntu on systems with or without
hard disks attached.

### Installation on a diskless system

The first steps of a diskless iSCSI installation are identical to the
[Installing from CD] section up to "Hard drive layout".

The installer will display a warning with the following message:

    No disk drive was detected. If you know the name of the driver needed by your disk drive, you can select it from the list.

Select the item in the list titled *login to iSCSI targets.*

You will be prompted to Enter an IP address to scan for iSCSI targets with a
description of the format for the address. Enter the IP address for the
location of your iSCSI target and navigate to *&lt;continue&gt;* then hit
ENTER

If authentication is required in order to access the iSCSI device, provide the
*username* in the next field. Otherwise leave it blank.

If your system is able to connect to the iSCSI provider, you should see a list
of available iSCSI targets where the operating system can be installed. The
list should be similar to the following :

    Select the iSCSI targets you wish to use.

    iSCSI targets on 192.168.1.29:3260:

    [ ] iqn.2016-03.TrustyS-iscsitarget:storage.sys0

    <Go Back>                          <Continue>

Select the iSCSI target that you want to use with the space bar. Use the arrow
keys to navigate to the target that you want to select.

Navigate to *&lt;Continue&gt;* and hit ENTER.

If the connection to the iSCSI target is successful, you will be prompted with
the *\[!!\] Partition disks* installation menu. The rest of the procedure is
identical to any normal installation on attached disks. Once the installation
is completed, you will be asked to reboot.

### Installation on a system with disk attached

Again, the iSCSI installation on a normal server with one or many disks
attached is identical to the [Installing from CD] section until we reach the
disk partitioning menu. Instead of using any of the Guided selection, we need
to perform the following steps :

Navigate to the Manual menu entry

Select the Configure iSCSI Volumes menu entry

Choose the Log into iSCSI targets

You will be prompted to Enter an IP address to scan for iSCSI targets. with a
description of the format for the address. Enter the IP address and navigate
to *&lt;continue&gt;* then hit ENTER

If authentication is required in order to access the iSCSI device, provide the
*username* in the next field or leave it blank.

If your system is able to connect to the iSCSI provider, you should see a list
of available iSCSI targets where the operating system can be installed. The
list should be similar to the following :

    Select the iSCSI targets you wish to use.

    iSCSI targets on 192.168.1.29:3260:

    [ ] iqn.2016-03.TrustyS-iscsitarget:storage.sys0

    <Go Back>                          <Continue>

Select the iSCSI target that you want to use with the space bar. Use the arrow
keys to navigate to the target that you want to select

Navigate to &lt;Continue&gt; and hit ENTER.

If successful, you will come back to the menu asking you to Log into iSCSI
targets. Navigate to Finish and hit ENTER

The newly connected iSCSI disk will appear in the overview section as a device
prefixed with SCSI. This is the disk that you should select as your
installation disk. Once identified, you can choose any of the partitioning
methods.

> **Warning**
>
> Depending on your system configuration, there may be other SCSI disks
> attached to the system. Be very careful to identify the proper device before
> proceeding with the installation. Otherwise, irreversible data loss may
> result from performing an installation on the wrong disk.

### Rebooting to an iSCSI target

The procedure is specific to your hardware platform. As an example, here is
how to reboot to your iSCSI target using iPXE

    iPXE> dhcp

    Configuring (net0 52:54:00:a4:f2:a9)....... ok

    iPXE> sanboot iscsi:192.168.1.29::::iqn.2016-03.TrustyS-iscsitarget:storage.sys0

If the procedure is successful, you should see the Grub menu appear on the
screen.

# Kernel Crash Dump

## Introduction {#kernel-dump-introduction}

A Kernel Crash Dump refers to a portion of the contents of volatile memory
(RAM) that is copied to disk whenever the execution of the kernel is
disrupted. The following events can cause a kernel disruption :

-   Kernel Panic

-   Non Maskable Interrupts (NMI)

-   Machine Check Exceptions (MCE)

-   Hardware failure

-   Manual intervention

For some of those events (panic, NMI) the kernel will react automatically and
trigger the crash dump mechanism through *kexec*. In other situations a manual
intervention is required in order to capture the memory. Whenever one of the
above events occurs, it is important to find out the root cause in order to
prevent it from happening again. The cause can be determined by inspecting the
copied memory contents.

## Kernel Crash Dump Mechanism {#kernel-crash-dump-mechanisms}

When a kernel panic occurs, the kernel relies on the *kexec* mechanism to
quickly reboot a new instance of the kernel in a pre-reserved section of
memory that had been allocated when the system booted (see below). This
permits the existing memory area to remain untouched in order to safely copy
its contents to storage.

## Installation {#Installation}

The kernel crash dump utility is installed with the following command:

    sudo apt install linux-crashdump

> **Note**
>
> Starting with 16.04, the kernel crash dump mechanism is enabled by default.
> During the installation, you will be prompted with the following dialog.
> Unless chosen otherwise, the kdump mechanism will be enabled.

     |------------------------| Configuring kdump-tools |------------------------|
     |                                                                           |
     |                                                                           |
     | If you choose this option, the kdump-tools mechanism will be enabled. A   |
     | reboot is still required in order to enable the crashkernel kernel        |
     | parameter.                                                                |
     |                                                                           |
     | Should kdump-tools be enabled by default?                                 |
     |                                                                           |
     |                    <Yes>                       <No>                       |
     |                                                                           |
     |---------------------------------------------------------------------------|
            

If you ever need to manually enable the functionality, you can use the
`dpkg-reconfigure kdump-tools` command and answer Yes to the question. You can
also edit `/etc/default/kdump-tools` by including the following line:

    USE_KDUMP=1

If a reboot has not been done since installation of the linux-crashdump
package, a reboot will be required in order to activate the crashkernel= boot
parameter. Upon reboot, kdump-tools will be enabled and active.

If you enable kdump-tools after a reboot, you will only need to issue the
`kdump-config load` command to activate the kdump mechanism.

## Configuration {#kernel-dump-configuration}

In addition to local dump, it is now possible to use the remote dump
functionality to send the kernel crash dump to a remote server, using either
the *SSH* or *NFS* protocols.

### Local Kernel Crash Dumps {#local-dump}

Local dumps are configured automatically and will remain in use unless a
remote protocol is chosen. Many configuration options exist and are thoroughly
documented in the `/etc/default/kdump-tools` file.

### Remote Kernel Crash Dumps using the SSH protocol {#ssh-dump}

To enable remote dumps using the *SSH* protocol, the
`/etc/default/kdump-tools` must be modified in the following manner :

    # ---------------------------------------------------------------------------
    # Remote dump facilities:
    # SSH - username and hostname of the remote server that will receive the dump
    #       and dmesg files.
    # SSH_KEY - Full path of the ssh private key to be used to login to the remote
    #           server. use kdump-config propagate to send the public key to the
    #           remote server
    # HOSTTAG - Select if hostname of IP address will be used as a prefix to the
    #           timestamped directory when sending files to the remote server.
    #           'ip' is the default.
    SSH="ubuntu@kdump-netcrash"
            

The only mandatory variable to define is SSH. It must contain the username and
hostname of the remote server using the format {username}@{remote server}.

SSH\_KEY may be used to provide an existing private key to be used. Otherwise,
the `kdump-config propagate` command will create a new keypair. The HOSTTAG
variable may be used to use the hostname of the system as a prefix to the
remote directory to be created instead of the IP address.

The following example shows how `kdump-config propagate` is used to create and
propagate a new keypair to the remote server :

    sudo kdump-config propagate
    Need to generate a new ssh key...
    The authenticity of host 'kdump-netcrash (192.168.1.74)' can't be established.
    ECDSA key fingerprint is SHA256:iMp+5Y28qhbd+tevFCWrEXykDd4dI3yN4OVlu3CBBQ4.
    Are you sure you want to continue connecting (yes/no)? yes
    ubuntu@kdump-netcrash's password: 
    propagated ssh key /root/.ssh/kdump_id_rsa to server ubuntu@kdump-netcrash
            

The password of the account used on the remote server will be required in
order to successfully send the public key to the server

The `kdump-config show` command can be used to confirm that kdump is correctly
configured to use the SSH protocol :

    kdump-config show
    DUMP_MODE:        kdump
    USE_KDUMP:        1
    KDUMP_SYSCTL:     kernel.panic_on_oops=1
    KDUMP_COREDIR:    /var/crash
    crashkernel addr: 0x2c000000
       /var/lib/kdump/vmlinuz: symbolic link to /boot/vmlinuz-4.4.0-10-generic
    kdump initrd: 
       /var/lib/kdump/initrd.img: symbolic link to /var/lib/kdump/initrd.img-4.4.0-10-generic
    SSH:              ubuntu@kdump-netcrash
    SSH_KEY:          /root/.ssh/kdump_id_rsa
    HOSTTAG:          ip
    current state:    ready to kdump
            

### Remote Kernel Crash Dumps using the NFS protocol {#nfs-dump}

To enable remote dumps using the *NFS* protocol, the
`/etc/default/kdump-tools` must be modified in the following manner :

    # NFS -     Hostname and mount point of the NFS server configured to receive
    #           the crash dump. The syntax must be {HOSTNAME}:{MOUNTPOINT} 
    #           (e.g. remote:/var/crash)
    #
    NFS="kdump-netcrash:/var/crash"
              

As with the SSH protocol, the HOSTTAG variable can be used to replace the IP
address by the hostname as the prefix of the remote directory.

The `kdump-config show` command can be used to confirm that kdump is correctly
configured to use the NFS protocol :

    kdump-config show
    DUMP_MODE:        kdump
    USE_KDUMP:        1
    KDUMP_SYSCTL:     kernel.panic_on_oops=1
    KDUMP_COREDIR:    /var/crash
    crashkernel addr: 0x2c000000
       /var/lib/kdump/vmlinuz: symbolic link to /boot/vmlinuz-4.4.0-10-generic
    kdump initrd: 
       /var/lib/kdump/initrd.img: symbolic link to /var/lib/kdump/initrd.img-4.4.0-10-generic
    NFS:              kdump-netcrash:/var/crash
    HOSTTAG:          hostname
    current state:    ready to kdump
          

## Verification

To confirm that the kernel dump mechanism is enabled, there are a few things
to verify. First, confirm that the *crashkernel* boot parameter is present
(note: The following line has been split into two to fit the format of this
document:

    cat /proc/cmdline

    BOOT_IMAGE=/vmlinuz-3.2.0-17-server root=/dev/mapper/PreciseS-root ro
     crashkernel=384M-2G:64M,2G-:128M

The *crashkernel* parameter has the following syntax:

    crashkernel=<range1>:<size1>[,<range2>:<size2>,...][@offset]
        range=start-[end] 'start' is inclusive and 'end' is exclusive.
            

So for the crashkernel parameter found in `/proc/cmdline` we would have :

    crashkernel=384M-2G:64M,2G-:128M

The above value means:

-   if the RAM is smaller than 384M, then don't reserve anything (this is the
    "rescue" case)

-   if the RAM size is between 386M and 2G (exclusive), then reserve 64M

-   if the RAM size is larger than 2G, then reserve 128M

Second, verify that the kernel has reserved the requested memory area for the
kdump kernel by doing:

    dmesg | grep -i crash

    ...
    [    0.000000] Reserving 64MB of memory at 800MB for crashkernel (System RAM: 1023MB)

Finally, as seen previously, the `kdump-config show` command displays the
current status of the kdump-tools configuration :

            kdump-config show
    DUMP_MODE:        kdump
    USE_KDUMP:        1
    KDUMP_SYSCTL:     kernel.panic_on_oops=1
    KDUMP_COREDIR:    /var/crash
    crashkernel addr: 0x2c000000
       /var/lib/kdump/vmlinuz: symbolic link to /boot/vmlinuz-4.4.0-10-generic
    kdump initrd: 
          /var/lib/kdump/initrd.img: symbolic link to /var/lib/kdump/initrd.img-4.4.0-10-generic
    current state:    ready to kdump

    kexec command:
          /sbin/kexec -p --command-line="BOOT_IMAGE=/vmlinuz-4.4.0-10-generic root=/dev/mapper/VividS--vg-root ro debug break=init console=ttyS0,115200 irqpoll maxcpus=1 nousb systemd.unit=kdump-tools.service" --initrd=/var/lib/kdump/initrd.img /var/lib/kdump/vmlinuz
          

## Testing the Crash Dump Mechanism {#kdump-testing}

> **Warning**
>
> Testing the Crash Dump Mechanism will cause *a system reboot.* In certain
> situations, this can cause data loss if the system is under heavy load. If
> you want to test the mechanism, make sure that the system is idle or under
> very light load.

Verify that the *SysRQ* mechanism is enabled by looking at the value of the
`/proc/sys/kernel/sysrq` kernel parameter :

    cat /proc/sys/kernel/sysrq

If a value of *0* is returned the feature is disabled. Enable it with the
following command :

    sudo sysctl -w kernel.sysrq=1

Once this is done, you must become root, as just using `sudo` will not be
sufficient. As the *root* user, you will have to issue the command
`echo c > /proc/sysrq-trigger`. If you are using a network connection, you
will lose contact with the system. This is why it is better to do the test
while being connected to the system console. This has the advantage of making
the kernel dump process visible.

A typical test output should look like the following :

    sudo -s
    [sudo] password for ubuntu: 
    # echo c > /proc/sysrq-trigger
    [   31.659002] SysRq : Trigger a crash
    [   31.659749] BUG: unable to handle kernel NULL pointer dereference at           (null)
    [   31.662668] IP: [<ffffffff8139f166>] sysrq_handle_crash+0x16/0x20
    [   31.662668] PGD 3bfb9067 PUD 368a7067 PMD 0 
    [   31.662668] Oops: 0002 [#1] SMP 
    [   31.662668] CPU 1 
    ....

The rest of the output is truncated, but you should see the system rebooting
and somewhere in the log, you will see the following line :

    Begin: Saving vmcore from kernel crash ...

Once completed, the system will reboot to its normal operational mode. You
will then find Kernel Crash Dump file in the `/var/crash` directory :

    ls /var/crash
    linux-image-3.0.0-12-server.0.crash

## Resources {#kdump-resources}

Kernel Crash Dump is a vast topic that requires good knowledge of the linux
kernel. You can find more information on the topic here :

-   [Kdump kernel documentation].

-   [The crash tool]

-   [Analyzing Linux Kernel Crash] (Based on Fedora, it still gives a good
    walkthrough of kernel dump analysis)

  [Ubuntu Installation Guide]: https://help.ubuntu.com/&distro-rev-short;/installation-guide/
  [Linux Kernel in a Nutshell]: http://www.kroah.com/lkn/
  [???]: #backups
  [Ubuntu web site]: http://www.ubuntu.com/download/server/download
  [Advanced Installation]: #advanced-installation
  [1]: #automatic-updates
  [Landscape]: http://landscape.canonical.com/
  [Package Tasks]: #install-tasks
  [2]: #aptitude
  [Degraded RAID]: #raid-degraded
  [RAID Maintenance]: #raid-maintenance
  [Ubuntu Wiki Articles on RAID]: https://help.ubuntu.com/community/Installation#raid
  [Software RAID HOWTO]: http://www.faqs.org/docs/Linux-HOWTO/Software-RAID-HOWTO.html
  [Managing RAID on Linux]: http://oreilly.com/catalog/9781565927308/
  [Ubuntu Wiki LVM Articles]: https://help.ubuntu.com/community/Installation#lvm
  [LVM HOWTO]: http://tldp.org/HOWTO/LVM-HOWTO/index.html
  [Managing Disk Space with LVM]: http://www.linuxdevcenter.com/pub/a/linux/2006/04/27/managing-disk-space-with-lvm.html
  [fdisk man page]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man8/fdisk.8.html
  [Installing from CD]: #installing-from-cd
  [Kdump kernel documentation]: http://www.kernel.org/doc/Documentation/kdump/kdump.txt
  [The crash tool]: http://people.redhat.com/~anderson/
  [Analyzing Linux Kernel Crash]: http://www.dedoimedo.com/computers/crash-analyze.html
