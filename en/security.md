# Security

Security should always be considered when installing, deploying, and using any
type of computer system. Although a fresh installation of Ubuntu is relatively
safe for immediate use on the Internet, it is important to have a balanced
understanding of your system's security posture based on how it will be used
after deployment.

This chapter provides an overview of security-related topics as they pertain
to Ubuntu DISTRO-REV Server Edition, and outlines simple measures you may use
to protect your server and network from any number of potential security
threats.

# User Management

User management is a critical part of maintaining a secure system. Ineffective
user and privilege management often lead many systems into being compromised.
Therefore, it is important that you understand how you can protect your server
through simple and effective user account management techniques.

## Where is root?

Ubuntu developers made a conscientious decision to disable the administrative
root account by default in all Ubuntu installations. This does not mean that
the root account has been deleted or that it may not be accessed. It merely
has been given a password which matches no possible encrypted value, therefore
may not log in directly by itself.

Instead, users are encouraged to make use of a tool by the name of sudo to
carry out system administrative duties. Sudo allows an authorized user to
temporarily elevate their privileges using their own password instead of
having to know the password belonging to the root account. This simple yet
effective methodology provides accountability for all user actions, and gives
the administrator granular control over which actions a user can perform with
said privileges.

-   If for some reason you wish to enable the root account, simply give it a
    password:

    > **Note**
    >
    > Configurations with root passwords are not supported.

        sudo passwd

    Sudo will prompt you for your password, and then ask you to supply a new
    password for root as shown below:

        [sudo] password for username: (enter your own password)
        Enter new UNIX password: (enter a new password for root)
        Retype new UNIX password: (repeat new password for root)
        passwd: password updated successfully

-   To disable the root account password, use the following passwd syntax:

        sudo passwd -l root

    However, to disable the root account itself, use the following command:

        usermod --expiredate 1

-   You should read more on Sudo by reading the man page:

        man sudo

By default, the initial user created by the Ubuntu installer is a member of
the group "*sudo*" which is added to the file `/etc/sudoers` as an authorized
sudo user. If you wish to give any other account full root access through
sudo, simply add them to the *sudo* group.

## Adding and Deleting Users {#adding-deleting-users}

The process for managing local users and groups is straightforward and differs
very little from most other GNU/Linux operating systems. Ubuntu and other
Debian based distributions encourage the use of the "adduser" package for
account management.

-   To add a user account, use the following syntax, and follow the prompts to
    give the account a password and identifiable characteristics, such as a
    full name, phone number, etc.

        sudo adduser username

-   To delete a user account and its primary group, use the following syntax:

        sudo deluser username

    Deleting an account does not remove their respective home folder. It is up
    to you whether or not you wish to delete the folder manually or keep it
    according to your desired retention policies.

    Remember, any user added later on with the same UID/GID as the previous
    owner will now have access to this folder if you have not taken the
    necessary precautions.

    You may want to change these UID/GID values to something more appropriate,
    such as the root account, and perhaps even relocate the folder to avoid
    future conflicts:

        sudo chown -R root:root /home/username/
        sudo mkdir /home/archived_users/
        sudo mv /home/username /home/archived_users/

-   To temporarily lock or unlock a user account, use the following syntax,
    respectively:

        sudo passwd -l username
        sudo passwd -u username

-   To add or delete a personalized group, use the following syntax,
    respectively:

        sudo addgroup groupname
        sudo delgroup groupname

-   To add a user to a group, use the following syntax:

        sudo adduser username groupname

## User Profile Security

When a new user is created, the adduser utility creates a brand new home
directory named `/home/username`. The default profile is modeled after the
contents found in the directory of `/etc/skel`, which includes all profile
basics.

If your server will be home to multiple users, you should pay close attention
to the user home directory permissions to ensure confidentiality. By default,
user home directories in Ubuntu are created with world read/execute
permissions. This means that all users can browse and access the contents of
other users home directories. This may not be suitable for your environment.

-   To verify your current user home directory permissions, use the following
    syntax:

        ls -ld /home/username

    The following output shows that the directory `/home/username` has
    world-readable permissions:

        drwxr-xr-x  2 username username    4096 2007-10-02 20:03 username

-   You can remove the world readable-permissions using the following syntax:

        sudo chmod 0750 /home/username

    > **Note**
    >
    > Some people tend to use the recursive option (-R) indiscriminately which
    > modifies all child folders and files, but this is not necessary, and may
    > yield other undesirable results. The parent directory alone is
    > sufficient for preventing unauthorized access to anything below
    > the parent.

    A much more efficient approach to the matter would be to modify the
    adduser global default permissions when creating user home folders. Simply
    edit the file `/etc/adduser.conf` and modify the `DIR_MODE` variable to
    something appropriate, so that all new home directories will receive the
    correct permissions.

        DIR_MODE=0750

-   After correcting the directory permissions using any of the previously
    mentioned techniques, verify the results using the following syntax:

        ls -ld /home/username

    The results below show that world-readable permissions have been removed:

        drwxr-x---   2 username username    4096 2007-10-02 20:03 username

## Password Policy

A strong password policy is one of the most important aspects of your security
posture. Many successful security breaches involve simple brute force and
dictionary attacks against weak passwords. If you intend to offer any form of
remote access involving your local password system, make sure you adequately
address minimum password complexity requirements, maximum password lifetimes,
and frequent audits of your authentication systems.

### Minimum Password Length

By default, Ubuntu requires a minimum password length of 6 characters, as well
as some basic entropy checks. These values are controlled in the file
`/etc/pam.d/common-password`, which is outlined below.

    password        [success=1 default=ignore]      pam_unix.so obscure sha512

If you would like to adjust the minimum length to 8 characters, change the
appropriate variable to min=8. The modification is outlined below.

    password        [success=1 default=ignore]      pam_unix.so obscure sha512 minlen=8

> **Note**
>
> Basic password entropy checks and minimum length rules do not apply to the
> administrator using sudo level commands to setup a new user.

### Password Expiration

When creating user accounts, you should make it a policy to have a minimum and
maximum password age forcing users to change their passwords when they expire.

-   To easily view the current status of a user account, use the following
    syntax:

        sudo chage -l username

    The output below shows interesting facts about the user account, namely
    that there are no policies applied:

        Last password change                                    : Jan 20, 2015
        Password expires                                        : never
        Password inactive                                       : never
        Account expires                                         : never
        Minimum number of days between password change          : 0
        Maximum number of days between password change          : 99999
        Number of days of warning before password expires       : 7

-   To set any of these values, simply use the following syntax, and follow
    the interactive prompts:

        sudo chage username

    The following is also an example of how you can manually change the
    explicit expiration date (-E) to 01/31/2015, minimum password age (-m) of
    5 days, maximum password age (-M) of 90 days, inactivity period (-I) of 5
    days after password expiration, and a warning time period (-W) of 14 days
    before password expiration:

        sudo chage -E 01/31/2015 -m 5 -M 90 -I 30 -W 14 username

-   To verify changes, use the same syntax as mentioned previously:

        sudo chage -l username

    The output below shows the new policies that have been established for the
    account:

        Last password change                                    : Jan 20, 2015
        Password expires                                        : Apr 19, 2015
        Password inactive                                       : May 19, 2015
        Account expires                                         : Jan 31, 2015
        Minimum number of days between password change          : 5
        Maximum number of days between password change          : 90
        Number of days of warning before password expires       : 14

## Other Security Considerations

Many applications use alternate authentication mechanisms that can be easily
overlooked by even experienced system administrators. Therefore, it is
important to understand and control how users authenticate and gain access to
services and applications on your server.

### SSH Access by Disabled Users

Simply disabling/locking a user account will not prevent a user from logging
into your server remotely if they have previously set up RSA public key
authentication. They will still be able to gain shell access to the server,
without the need for any password. Remember to check the users home directory
for files that will allow for this type of authenticated SSH access, e.g.
`/home/username/.ssh/authorized_keys`.

Remove or rename the directory `.ssh/` in the user's home folder to prevent
further SSH authentication capabilities.

Be sure to check for any established SSH connections by the disabled user, as
it is possible they may have existing inbound or outbound connections. Kill
any that are found.

    who | grep username  (to get the pts/# terminal)
    sudo pkill -f pts/#

Restrict SSH access to only user accounts that should have it. For example,
you may create a group called "sshlogin" and add the group name as the value
associated with the `AllowGroups` variable located in the file
`/etc/ssh/sshd_config`.

    AllowGroups sshlogin

Then add your permitted SSH users to the group "sshlogin", and restart the SSH
service.

    sudo adduser username sshlogin
    sudo systemctl restart sshd.service

### External User Database Authentication {#external-db-auth}

Most enterprise networks require centralized authentication and access
controls for all system resources. If you have configured your server to
authenticate users against external databases, be sure to disable the user
accounts both externally and locally. This way you ensure that local fallback
authentication is not possible.

# Console Security

As with any other security barrier you put in place to protect your server, it
is pretty tough to defend against untold damage caused by someone with
physical access to your environment, for example, theft of hard drives, power
or service disruption, and so on. Therefore, console security should be
addressed merely as one component of your overall physical security strategy.
A locked "screen door" may deter a casual criminal, or at the very least slow
down a determined one, so it is still advisable to perform basic precautions
with regard to console security.

The following instructions will help defend your server against issues that
could otherwise yield very serious consequences.

## Disable Ctrl+Alt+Delete {#disable-ctrl-alt-delete}

Anyone that has physical access to the keyboard can simply use the <span
class="keycombo">Ctrl+Alt+Delete</span> key combination to reboot the server
without having to log on. While someone could simply unplug the power source,
you should still prevent the use of this key combination on a production
server. This forces an attacker to take more drastic measures to reboot the
server, and will prevent accidental reboots at the same time.

To disable the reboot action taken by pressing the <span
class="keycombo">Ctrl+Alt+Delete</span> key combination, run the following two
commands:

    sudo systemctl mask ctrl-alt-del.target
    sudo systemctl daemon-reload

# Firewall

## Introduction {#firewall-introduction}

The Linux kernel includes the *Netfilter* subsystem, which is used to
manipulate or decide the fate of network traffic headed into or through your
server. All modern Linux firewall solutions use this system for packet
filtering.

The kernel's packet filtering system would be of little use to administrators
without a userspace interface to manage it. This is the purpose of iptables:
When a packet reaches your server, it will be handed off to the Netfilter
subsystem for acceptance, manipulation, or rejection based on the rules
supplied to it from userspace via iptables. Thus, iptables is all you need to
manage your firewall, if you're familiar with it, but many frontends are
available to simplify the task.

## ufw - Uncomplicated Firewall {#firewall-ufw}

The default firewall configuration tool for Ubuntu is ufw. Developed to ease
iptables firewall configuration, ufw provides a user-friendly way to create an
IPv4 or IPv6 host-based firewall.

ufw by default is initially disabled. From the ufw man page:

“ufw is not intended to provide complete firewall functionality via its
command interface, but instead provides an easy way to add or remove simple
rules. It is currently mainly used for host-based firewalls.”

The following are some examples of how to use ufw:

-   First, ufw needs to be enabled. From a terminal prompt enter:

        sudo ufw enable

-   To open a port (SSH in this example):

        sudo ufw allow 22

-   Rules can also be added using a *numbered* format:

        sudo ufw insert 1 allow 80

-   Similarly, to close an opened port:

        sudo ufw deny 22

-   To remove a rule, use delete followed by the rule:

        sudo ufw delete deny 22

-   It is also possible to allow access from specific hosts or networks to
    a port. The following example allows SSH access from host 192.168.0.2 to
    any IP address on this host:

        sudo ufw allow proto tcp from 192.168.0.2 to any port 22

    Replace 192.168.0.2 with 192.168.0.0/24 to allow SSH access from the
    entire subnet.

-   Adding the *--dry-run* option to a *ufw* command will output the resulting
    rules, but not apply them. For example, the following is what would be
    applied if opening the HTTP port:

         sudo ufw --dry-run allow http

        *filter
        :ufw-user-input - [0:0]
        :ufw-user-output - [0:0]
        :ufw-user-forward - [0:0]
        :ufw-user-limit - [0:0]
        :ufw-user-limit-accept - [0:0]
        ### RULES ###

        ### tuple ### allow tcp 80 0.0.0.0/0 any 0.0.0.0/0
        -A ufw-user-input -p tcp --dport 80 -j ACCEPT

        ### END RULES ###
        -A ufw-user-input -j RETURN
        -A ufw-user-output -j RETURN
        -A ufw-user-forward -j RETURN
        -A ufw-user-limit -m limit --limit 3/minute -j LOG --log-prefix "[UFW LIMIT]: "
        -A ufw-user-limit -j REJECT
        -A ufw-user-limit-accept -j ACCEPT
        COMMIT
        Rules updated

-   ufw can be disabled by:

        sudo ufw disable

-   To see the firewall status, enter:

        sudo ufw status

-   And for more verbose status information use:

        sudo ufw status verbose

-   To view the *numbered* format:

        sudo ufw status numbered

> **Note**
>
> If the port you want to open or close is defined in `/etc/services`, you can
> use the port name instead of the number. In the above examples, replace *22*
> with *ssh*.

This is a quick introduction to using ufw. Please refer to the ufw man page
for more information.

### ufw Application Integration

Applications that open ports can include an ufw profile, which details the
ports needed for the application to function properly. The profiles are kept
in `/etc/ufw/applications.d`, and can be edited if the default ports have been
changed.

-   To view which applications have installed a profile, enter the following
    in a terminal:

        sudo ufw app list

-   Similar to allowing traffic to a port, using an application profile is
    accomplished by entering:

        sudo ufw allow Samba

-   An extended syntax is available as well:

        ufw allow from 192.168.0.0/24 to any app Samba

    Replace *Samba* and *192.168.0.0/24* with the application profile you are
    using and the IP range for your network.

    > **Note**
    >
    > There is no need to specify the *protocol* for the application, because
    > that information is detailed in the profile. Also, note that the *app*
    > name replaces the *port* number.

-   To view details about which ports, protocols, etc., are defined for an
    application, enter:

        sudo ufw app info Samba

Not all applications that require opening a network port come with ufw
profiles, but if you have profiled an application and want the file to be
included with the package, please file a bug against the package in Launchpad.

    ubuntu-bug nameofpackage

## IP Masquerading

The purpose of IP Masquerading is to allow machines with private, non-routable
IP addresses on your network to access the Internet through the machine doing
the masquerading. Traffic from your private network destined for the Internet
must be manipulated for replies to be routable back to the machine that made
the request. To do this, the kernel must modify the *source* IP address of
each packet so that replies will be routed back to it, rather than to the
private IP address that made the request, which is impossible over the
Internet. Linux uses *Connection Tracking* (conntrack) to keep track of which
connections belong to which machines and reroute each return packet
accordingly. Traffic leaving your private network is thus "masqueraded" as
having originated from your Ubuntu gateway machine. This process is referred
to in Microsoft documentation as Internet Connection Sharing.

### ufw Masquerading {#ip-masquerade-ufw}

IP Masquerading can be achieved using custom ufw rules. This is possible
because the current back-end for ufw is iptables-restore with the rules files
located in `/etc/ufw/*.rules`. These files are a great place to add legacy
iptables rules used without ufw, and rules that are more network gateway or
bridge related.

The rules are split into two different files, rules that should be executed
before ufw command line rules, and rules that are executed after ufw command
line rules.

-   First, packet forwarding needs to be enabled in ufw. Two configuration
    files will need to be adjusted, in `/etc/default/ufw` change the
    *DEFAULT\_FORWARD\_POLICY* to “ACCEPT”:

        DEFAULT_FORWARD_POLICY="ACCEPT"

    Then edit `/etc/ufw/sysctl.conf` and uncomment:

        net/ipv4/ip_forward=1

    Similarly, for IPv6 forwarding uncomment:

        net/ipv6/conf/default/forwarding=1

-   Now add rules to the `/etc/ufw/before.rules` file. The default rules only
    configure the *filter* table, and to enable masquerading the *nat* table
    will need to be configured. Add the following to the top of the file just
    after the header comments:

        # nat Table rules
        *nat
        :POSTROUTING ACCEPT [0:0]

        # Forward traffic from eth1 through eth0.
        -A POSTROUTING -s 192.168.0.0/24 -o eth0 -j MASQUERADE

        # don't delete the 'COMMIT' line or these nat table rules won't be processed
        COMMIT

    The comments are not strictly necessary, but it is considered good
    practice to document your configuration. Also, when modifying any of the
    *rules* files in `/etc/ufw`, make sure these lines are the last line for
    each table modified:

        # don't delete the 'COMMIT' line or these rules won't be processed
        COMMIT

    For each *Table* a corresponding *COMMIT* statement is required. In these
    examples only the *nat* and *filter* tables are shown, but you can also
    add rules for the *raw* and *mangle* tables.

    > **Note**
    >
    > In the above example replace *eth0*, *eth1*, and *192.168.0.0/24* with
    > the appropriate interfaces and IP range for your network.

-   Finally, disable and re-enable ufw to apply the changes:

        sudo ufw disable && sudo ufw enable

IP Masquerading should now be enabled. You can also add any additional FORWARD
rules to the `/etc/ufw/before.rules`. It is recommended that these additional
rules be added to the *ufw-before-forward* chain.

### iptables Masquerading {#ip-masquerading-iptables}

iptables can also be used to enable Masquerading.

-   Similar to ufw, the first step is to enable IPv4 packet forwarding by
    editing `/etc/sysctl.conf` and uncomment the following line:

        net.ipv4.ip_forward=1

    If you wish to enable IPv6 forwarding also uncomment:

        net.ipv6.conf.default.forwarding=1

-   Next, execute the sysctl command to enable the new settings in the
    configuration file:

        sudo sysctl -p

-   IP Masquerading can now be accomplished with a single iptables rule, which
    may differ slightly based on your network configuration:

        sudo iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o ppp0 -j MASQUERADE

    The above command assumes that your private address space is
    192.168.0.0/16 and that your Internet-facing device is ppp0. The syntax is
    broken down as follows:

    -   -t nat -- the rule is to go into the nat table

    -   -A POSTROUTING -- the rule is to be appended (-A) to the POSTROUTING
        chain

    -   -s 192.168.0.0/16 -- the rule applies to traffic originating from the
        specified address space

    -   -o ppp0 -- the rule applies to traffic scheduled to be routed through
        the specified network device

    -   -j MASQUERADE -- traffic matching this rule is to "jump" (-j) to the
        MASQUERADE target to be manipulated as described above

-   Also, each chain in the filter table (the default table, and where most or
    all packet filtering occurs) has a default *policy* of ACCEPT, but if you
    are creating a firewall in addition to a gateway device, you may have set
    the policies to DROP or REJECT, in which case your masqueraded traffic
    needs to be allowed through the FORWARD chain for the above rule to work:

        sudo iptables -A FORWARD -s 192.168.0.0/16 -o ppp0 -j ACCEPT
        sudo iptables -A FORWARD -d 192.168.0.0/16 -m state \
        --state ESTABLISHED,RELATED -i ppp0 -j ACCEPT

    The above commands will allow all connections from your local network to
    the Internet and all traffic related to those connections to return to the
    machine that initiated them.

-   If you want masquerading to be enabled on reboot, which you probably do,
    edit `/etc/rc.local` and add any commands used above. For example add the
    first command with no filtering:

        iptables -t nat -A POSTROUTING -s 192.168.0.0/16 -o ppp0 -j MASQUERADE

## Logs {#firewall-logs}

Firewall logs are essential for recognizing attacks, troubleshooting your
firewall rules, and noticing unusual activity on your network. You must
include logging rules in your firewall for them to be generated, though, and
logging rules must come before any applicable terminating rule (a rule with a
target that decides the fate of the packet, such as ACCEPT, DROP, or REJECT).

If you are using ufw, you can turn on logging by entering the following in a
terminal:

    sudo ufw logging on

To turn logging off in ufw, simply replace *on* with *off* in the above
command.

If using iptables instead of ufw, enter:

    sudo iptables -A INPUT -m state --state NEW -p tcp --dport 80 \
    -j LOG --log-prefix "NEW_HTTP_CONN: "

A request on port 80 from the local machine, then, would generate a log in
dmesg that looks like this (single line split into 3 to fit this document):

    [4304885.870000] NEW_HTTP_CONN: IN=lo OUT= MAC=00:00:00:00:00:00:00:00:00:00:00:00:08:00
    SRC=127.0.0.1 DST=127.0.0.1 LEN=60 TOS=0x00 PREC=0x00 TTL=64 ID=58288 DF PROTO=TCP
    SPT=53981 DPT=80 WINDOW=32767 RES=0x00 SYN URGP=0

The above log will also appear in `/var/log/messages`, `/var/log/syslog`, and
`/var/log/kern.log`. This behavior can be modified by editing
`/etc/syslog.conf` appropriately or by installing and configuring ulogd and
using the ULOG target instead of LOG. The ulogd daemon is a userspace server
that listens for logging instructions from the kernel specifically for
firewalls, and can log to any file you like, or even to a PostgreSQL or MySQL
database. Making sense of your firewall logs can be simplified by using a log
analyzing tool such as logwatch, fwanalog, fwlogwatch, or lire.

## Other Tools {#other-firewall-tools}

There are many tools available to help you construct a complete firewall
without intimate knowledge of iptables. For the GUI-inclined:

-   [fwbuilder] is very powerful and will look familiar to an administrator
    who has used a commercial firewall utility such as Checkpoint FireWall-1.

If you prefer a command-line tool with plain-text configuration files:

-   [Shorewall] is a very powerful solution to help you configure an advanced
    firewall for any network.

## References {#firewall-references}

-   The [Ubuntu Firewall] wiki page contains information on the development
    of ufw.

-   Also, the ufw manual page contains some very useful information:
    `man ufw`.

-   See the [packet-filtering-HOWTO] for more information on using iptables.

-   The [nat-HOWTO] contains further details on masquerading.

-   The [IPTables HowTo] in the Ubuntu wiki is a great resource.

# AppArmor

AppArmor is a Linux Security Module implementation of name-based mandatory
access controls. AppArmor confines individual programs to a set of listed
files and posix 1003.1e draft capabilities.

AppArmor is installed and loaded by default. It uses *profiles* of an
application to determine what files and permissions the application requires.
Some packages will install their own profiles, and additional profiles can be
found in the apparmor-profiles package.

To install the apparmor-profiles package from a terminal prompt:

    sudo apt install apparmor-profiles

AppArmor profiles have two modes of execution:

-   Complaining/Learning: profile violations are permitted and logged. Useful
    for testing and developing new profiles.

-   Enforced/Confined: enforces profile policy as well as logging
    the violation.

## Using AppArmor {#apparmor-usage}

> **Caution**
>
> This section is plagued by a bug ([LP \#1304134]) and instructions will not
> work as advertised.

The apparmor-utils package contains command line utilities that you can use to
change the AppArmor execution mode, find the status of a profile, create new
profiles, etc.

-   apparmor\_status is used to view the current status of AppArmor profiles.

        sudo apparmor_status

-   aa-complain places a profile into *complain* mode.

        sudo aa-complain /path/to/bin

-   aa-enforce places a profile into *enforce* mode.

        sudo aa-enforce /path/to/bin

-   The `/etc/apparmor.d` directory is where the AppArmor profiles
    are located. It can be used to manipulate the *mode* of all profiles.

    Enter the following to place all profiles into complain mode:

        sudo aa-complain /etc/apparmor.d/*

    To place all profiles in enforce mode:

        sudo aa-enforce /etc/apparmor.d/*

-   apparmor\_parser is used to load a profile into the kernel. It can also be
    used to reload a currently loaded profile using the *-r* option. To load a
    profile:

        cat /etc/apparmor.d/profile.name | sudo apparmor_parser -a

    To reload a profile:

        cat /etc/apparmor.d/profile.name | sudo apparmor_parser -r

-   `systemctl` can be used to *reload* all profiles:

        sudo systemctl reload apparmor.service

-   The `/etc/apparmor.d/disable` directory can be used along with the
    apparmor\_parser -R option to *disable* a profile.

        sudo ln -s /etc/apparmor.d/profile.name /etc/apparmor.d/disable/
        sudo apparmor_parser -R /etc/apparmor.d/profile.name

    To *re-enable* a disabled profile remove the symbolic link to the profile
    in `/etc/apparmor.d/disable/`. Then load the profile using the
    *-a* option.

        sudo rm /etc/apparmor.d/disable/profile.name
        cat /etc/apparmor.d/profile.name | sudo apparmor_parser -a

-   AppArmor can be disabled, and the kernel module unloaded by entering the
    following:

        sudo systemctl stop apparmor.service
        sudo update-rc.d -f apparmor remove

-   To re-enable AppArmor enter:

        sudo systemctl start apparmor.service
        sudo update-rc.d apparmor defaults

> **Note**
>
> Replace *profile.name* with the name of the profile you want to manipulate.
> Also, replace `/path/to/bin/` with the actual executable file path. For
> example for the ping command use `/bin/ping`

## Profiles {#apparmor-profiles}

AppArmor profiles are simple text files located in `/etc/apparmor.d/`. The
files are named after the full path to the executable they profile replacing
the "/" with ".". For example `/etc/apparmor.d/bin.ping` is the AppArmor
profile for the `/bin/ping` command.

There are two main type of rules used in profiles:

-   *Path entries:* detail which files an application can access in the
    file system.

-   *Capability entries:* determine what privileges a confined process is
    allowed to use.

As an example, take a look at `/etc/apparmor.d/bin.ping`:

    #include <tunables/global>
    /bin/ping flags=(complain) {
      #include <abstractions/base>
      #include <abstractions/consoles>
      #include <abstractions/nameservice>

      capability net_raw,
      capability setuid,
      network inet raw,
      
      /bin/ping mixr,
      /etc/modules.conf r,
    }

-   *\#include &lt;tunables/global&gt;:* include statements from other files.
    This allows statements pertaining to multiple applications to be placed in
    a common file.

-   */bin/ping flags=(complain):* path to the profiled program, also setting
    the mode to *complain*.

-   *capability net\_raw,:* allows the application access to the CAP\_NET\_RAW
    Posix.1e capability.

-   */bin/ping mixr,:* allows the application read and execute access to
    the file.

> **Note**
>
> After editing a profile file the profile must be reloaded. See
> [Using AppArmor] for details.

### Creating a Profile {#apparmor-profiles-new}

-   *Design a test plan:* Try to think about how the application should
    be exercised. The test plan should be divided into small test cases. Each
    test case should have a small description and list the steps to follow.

    Some standard test cases are:

    -   Starting the program.

    -   Stopping the program.

    -   Reloading the program.

    -   Testing all the commands supported by the init script.

-   *Generate the new profile:* Use aa-genprof to generate a new profile. From
    a terminal:

        sudo aa-genprof executable

    For example:

        sudo aa-genprof slapd

-   To get your new profile included in the apparmor-profiles package, file a
    bug in *Launchpad* against the [AppArmor] package:

    -   Include your test plan and test cases.

    -   Attach your new profile to the bug.

### Updating Profiles {#apparmor-profiles-update}

When the program is misbehaving, audit messages are sent to the log files. The
program aa-logprof can be used to scan log files for AppArmor audit messages,
review them and update the profiles. From a terminal:

    sudo aa-logprof

## References {#apparmor-references}

-   See the [AppArmor Administration Guide] for advanced
    configuration options.

<!-- -->

-   For details using AppArmor with other Ubuntu releases see the [AppArmor
    Community Wiki] page.

<!-- -->

-   The [OpenSUSE AppArmor] page is another introduction to AppArmor.

<!-- -->

-   A great place to ask for AppArmor assistance, and get involved with the
    Ubuntu Server community, is the *\#ubuntu-server* IRC channel on
    [freenode].

# Certificates {#certificates-and-security}

One of the most common forms of cryptography today is *public-key*
cryptography. Public-key cryptography utilizes a *public key* and a *private
key*. The system works by *encrypting* information using the public key. The
information can then only be *decrypted* using the private key.

A common use for public-key cryptography is encrypting application traffic
using a Secure Socket Layer (SSL) or Transport Layer Security (TLS)
connection. One example: configuring Apache to provide *HTTPS*, the HTTP
protocol over SSL. This allows a way to encrypt traffic using a protocol that
does not itself provide encryption.

A *Certificate* is a method used to distribute a *public key* and other
information about a server and the organization who is responsible for it.
Certificates can be digitally signed by a *Certification Authority*, or CA. A
CA is a trusted third party that has confirmed that the information contained
in the certificate is accurate.

## Types of Certificates

To set up a secure server using public-key cryptography, in most cases, you
send your certificate request (including your public key), proof of your
company's identity, and payment to a CA. The CA verifies the certificate
request and your identity, and then sends back a certificate for your secure
server. Alternatively, you can create your own *self-signed* certificate.

> **Note**
>
> Note that self-signed certificates should not be used in most production
> environments.

Continuing the HTTPS example, a CA-signed certificate provides two important
capabilities that a self-signed certificate does not:

-   Browsers (usually) automatically recognize the certificate and allow a
    secure connection to be made without prompting the user.

-   When a CA issues a signed certificate, it is guaranteeing the identity of
    the organization that is providing the web pages to the browser.

Most Web browsers, and computers, that support SSL have a list of CAs whose
certificates they automatically accept. If a browser encounters a certificate
whose authorizing CA is not in the list, the browser asks the user to either
accept or decline the connection. Also, other applications may generate an
error message when using a self-signed certificate.

The process of getting a certificate from a CA is fairly easy. A quick
overview is as follows:

1.  Create a private and public encryption key pair.

2.  Create a certificate request based on the public key. The certificate
    request contains information about your server and the company hosting it.

3.  Send the certificate request, along with documents proving your identity,
    to a CA. We cannot tell you which certificate authority to choose. Your
    decision may be based on your past experiences, or on the experiences of
    your friends or colleagues, or purely on monetary factors.

    Once you have decided upon a CA, you need to follow the instructions they
    provide on how to obtain a certificate from them.

4.  When the CA is satisfied that you are indeed who you claim to be, they
    send you a digital certificate.

5.  Install this certificate on your secure server, and configure the
    appropriate applications to use the certificate.

## Generating a Certificate Signing Request (CSR) {#generating-a-csr}

Whether you are getting a certificate from a CA or generating your own
self-signed certificate, the first step is to generate a key.

If the certificate will be used by service daemons, such as Apache, Postfix,
Dovecot, etc., a key without a passphrase is often appropriate. Not having a
passphrase allows the services to start without manual intervention, usually
the preferred way to start a daemon.

This section will cover generating a key with a passphrase, and one without.
The non-passphrase key will then be used to generate a certificate that can be
used with various service daemons.

> **Warning**
>
> Running your secure service without a passphrase is convenient because you
> will not need to enter the passphrase every time you start your secure
> service. But it is insecure and a compromise of the key means a compromise
> of the server as well.

To generate the *keys* for the Certificate Signing Request (CSR) run the
following command from a terminal prompt:

    openssl genrsa -des3 -out server.key 2048

    Generating RSA private key, 2048 bit long modulus
    ..........................++++++
    .......++++++
    e is 65537 (0x10001)
    Enter pass phrase for server.key:

You can now enter your passphrase. For best security, it should at least
contain eight characters. The minimum length when specifying -des3 is four
characters. It should include numbers and/or punctuation and not be a word in
a dictionary. Also remember that your passphrase is case-sensitive.

Re-type the passphrase to verify. Once you have re-typed it correctly, the
server key is generated and stored in the `server.key` file.

Now create the insecure key, the one without a passphrase, and shuffle the key
names:

    openssl rsa -in server.key -out server.key.insecure
    mv server.key server.key.secure
    mv server.key.insecure server.key

The insecure key is now named `server.key`, and you can use this file to
generate the CSR without passphrase.

To create the CSR, run the following command at a terminal prompt:

    openssl req -new -key server.key -out server.csr

It will prompt you enter the passphrase. If you enter the correct passphrase,
it will prompt you to enter Company Name, Site Name, Email Id, etc. Once you
enter all these details, your CSR will be created and it will be stored in the
`server.csr` file.

You can now submit this CSR file to a CA for processing. The CA will use this
CSR file and issue the certificate. On the other hand, you can create
self-signed certificate using this CSR.

## Creating a Self-Signed Certificate

To create the self-signed certificate, run the following command at a terminal
prompt:

    openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt

The above command will prompt you to enter the passphrase. Once you enter the
correct passphrase, your certificate will be created and it will be stored in
the `server.crt` file.

> **Warning**
>
> If your secure server is to be used in a production environment, you
> probably need a CA-signed certificate. It is not recommended to use
> self-signed certificate.

## Installing the Certificate

You can install the key file `server.key` and certificate file `server.crt`,
or the certificate file issued by your CA, by running following commands at a
terminal prompt:

    sudo cp server.crt /etc/ssl/certs
    sudo cp server.key /etc/ssl/private

Now simply configure any applications, with the ability to use public-key
cryptography, to use the *certificate* and *key* files. For example, Apache
can provide HTTPS, Dovecot can provide IMAPS and POP3S, etc.

## Certification Authority {#certificate-authority}

If the services on your network require more than a few self-signed
certificates it may be worth the additional effort to setup your own internal
*Certification Authority (CA)*. Using certificates signed by your own CA,
allows the various services using the certificates to easily trust other
services using certificates issued from the same CA.

First, create the directories to hold the CA certificate and related files:

    sudo mkdir /etc/ssl/CA
    sudo mkdir /etc/ssl/newcerts

The CA needs a few additional files to operate, one to keep track of the last
serial number used by the CA, each certificate must have a unique serial
number, and another file to record which certificates have been issued:

    sudo sh -c "echo '01' > /etc/ssl/CA/serial"
    sudo touch /etc/ssl/CA/index.txt

The third file is a CA configuration file. Though not strictly necessary, it
is very convenient when issuing multiple certificates. Edit
`/etc/ssl/openssl.cnf`, and in the *\[ CA\_default \]* change:

    dir             = /etc/ssl              # Where everything is kept
    database        = $dir/CA/index.txt     # database index file.
    certificate     = $dir/certs/cacert.pem # The CA certificate
    serial          = $dir/CA/serial        # The current serial number
    private_key     = $dir/private/cakey.pem# The private key

Next, create the self-signed root certificate:

    openssl req -new -x509 -extensions v3_ca -keyout cakey.pem -out cacert.pem -days 3650

You will then be asked to enter the details about the certificate.

Now install the root certificate and key:

    sudo mv cakey.pem /etc/ssl/private/
    sudo mv cacert.pem /etc/ssl/certs/

You are now ready to start signing certificates. The first item needed is a
Certificate Signing Request (CSR), see
[Generating a Certificate Signing Request (CSR)] for details. Once you have a
CSR, enter the following to generate a certificate signed by the CA:

    sudo openssl ca -in server.csr -config /etc/ssl/openssl.cnf

After entering the password for the CA key, you will be prompted to sign the
certificate, and again to commit the new certificate. You should then see a
somewhat large amount of output related to the certificate creation.

There should now be a new file, `/etc/ssl/newcerts/01.pem`, containing the
same output. Copy and paste everything beginning with the line: *-----BEGIN
CERTIFICATE-----* and continuing through the line: *----END CERTIFICATE-----*
lines to a file named after the hostname of the server where the certificate
will be installed. For example `mail.example.com.crt`, is a nice descriptive
name.

Subsequent certificates will be named `02.pem`, `03.pem`, etc.

> **Note**
>
> Replace *mail.example.com.crt* with your own descriptive name.

Finally, copy the new certificate to the host that needs it, and configure the
appropriate applications to use it. The default location to install
certificates is `/etc/ssl/certs`. This enables multiple services to use the
same certificate without overly complicated file permissions.

For applications that can be configured to use a CA certificate, you should
also copy the `/etc/ssl/certs/cacert.pem` file to the `/etc/ssl/certs/`
directory on each server.

## References {#certificate-references}

-   For more detailed instructions on using cryptography see the [SSL
    Certificates HOWTO] by tldp.org:

-   The Wikipedia [HTTPS] page has more information regarding HTTPS.

-   For more information on *OpenSSL* see the [OpenSSL Home Page].

-   Also, O'Reilly's [Network Security with OpenSSL] is a good
    in-depth reference.

# eCryptfs

*eCryptfs* is a POSIX-compliant enterprise-class stacked cryptographic
filesystem for Linux. Layering on top of the filesystem layer *eCryptfs*
protects files no matter the underlying filesystem, partition type, etc.

During installation there is an option to encrypt the `/home` partition. This
will automatically configure everything needed to encrypt and mount the
partition.

As an example, this section will cover configuring `/srv` to be encrypted
using *eCryptfs*.

## Using eCryptfs {#ecryptfs-usage}

First, install the necessary packages. From a terminal prompt enter:

    sudo apt install ecryptfs-utils

Now mount the partition to be encrypted:

    sudo mount -t ecryptfs /srv /srv

You will then be prompted for some details on how ecryptfs should encrypt the
data.

To test that files placed in `/srv` are indeed encrypted copy the
`/etc/default` folder to `/srv`:

    sudo cp -r /etc/default /srv

Now unmount `/srv`, and try to view a file:

    sudo umount /srv
    cat /srv/default/cron

Remounting `/srv` using ecryptfs will make the data viewable once again.

## Automatically Mounting Encrypted Partitions {#ecryptfs-automount}

There are a couple of ways to automatically mount an ecryptfs encrypted
filesystem at boot. This example will use a `/root/.ecryptfsrc` file
containing mount options, along with a passphrase file residing on a USB key.

First, create `/root/.ecryptfsrc` containing:

    key=passphrase:passphrase_passwd_file=/mnt/usb/passwd_file.txt
    ecryptfs_sig=5826dd62cf81c615
    ecryptfs_cipher=aes
    ecryptfs_key_bytes=16
    ecryptfs_passthrough=n
    ecryptfs_enable_filename_crypto=n

> **Note**
>
> Adjust the *ecryptfs\_sig* to the signature in
> `/root/.ecryptfs/sig-cache.txt`.

Next, create the `/mnt/usb/passwd_file.txt` passphrase file:

    passphrase_passwd=[secrets]

Now add the necessary lines to `/etc/fstab`:

    /dev/sdb1       /mnt/usb        ext3    ro      0 0
    /srv /srv ecryptfs defaults 0 0

Make sure the USB drive is mounted before the encrypted partition.

Finally, reboot and the `/srv` should be mounted using *eCryptfs*.

## Other Utilities {#ecryptfs-other-utils}

The ecryptfs-utils package includes several other useful utilities:

-   *ecryptfs-setup-private:* creates a `~/Private` directory to contain
    encrypted information. This utility can be run by unprivileged users to
    keep data private from other users on the system.

-   *ecryptfs-mount-private* and *ecryptfs-umount-private* will mount and
    unmount a user's `~/Private` directory.

-   *ecryptfs-add-passphrase:* adds a new passphrase to the kernel keyring.

-   *ecryptfs-manager:* manages eCryptfs objects such as keys.

-   *ecryptfs-stat:* allows you to view the ecryptfs meta information for
    a file.

## References {#eCryptfs-references}

-   For more information on *eCryptfs* see the [Launchpad project page].

-   There is also a [Linux Journal] article covering *eCryptfs*.

-   Also, for more ecryptfs options and details see the [ecryptfs man page].

  [fwbuilder]: http://www.fwbuilder.org/
  [Shorewall]: http://www.shorewall.net/
  [Ubuntu Firewall]: https://wiki.ubuntu.com/UncomplicatedFirewall
  [packet-filtering-HOWTO]: http://www.netfilter.org/documentation/HOWTO/packet-filtering-HOWTO.html
  [nat-HOWTO]: http://www.netfilter.org/documentation/HOWTO/NAT-HOWTO.html
  [IPTables HowTo]: https://help.ubuntu.com/community/IptablesHowTo
  [LP \#1304134]: https://bugs.launchpad.net/ubuntu/+source/apparmor/+bug/1304134
  [Using AppArmor]: #apparmor-usage
  [AppArmor]: https://bugs.launchpad.net/ubuntu/+source/apparmor/+filebug
  [AppArmor Administration Guide]: http://www.novell.com/documentation/apparmor/apparmor201_sp10_admin/index.html?page=/documentation/apparmor/apparmor201_sp10_admin/data/book_apparmor_admin.html
  [AppArmor Community Wiki]: https://help.ubuntu.com/community/AppArmor
  [OpenSUSE AppArmor]: http://en.opensuse.org/SDB:AppArmor_geeks
  [freenode]: http://freenode.net
  [Generating a Certificate Signing Request (CSR)]: #generating-a-csr
  [SSL Certificates HOWTO]: http://tldp.org/HOWTO/SSL-Certificates-HOWTO/index.html
  [HTTPS]: http://en.wikipedia.org/wiki/HTTPS
  [OpenSSL Home Page]: http://www.openssl.org/
  [Network Security with OpenSSL]: http://oreilly.com/catalog/9780596002701/
  [Launchpad project page]: https://launchpad.net/ecryptfs
  [Linux Journal]: http://www.linuxjournal.com/article/9400
  [ecryptfs man page]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man7/ecryptfs.7.html
