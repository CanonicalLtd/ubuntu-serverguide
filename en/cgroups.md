# Control Groups {#cgroups}

Control groups (cgroups) are a kernel mechanism for grouping, tracking, and
limiting the resource usage of tasks. The kernel-provided administration
interface is through a virtual filesystem. Higher level cgroup administration
tools have been developed, including libcgroup and lmctfy. Additionally, there
is guidance at freedesktop.org for how applications can best cooperate using
the cgroup filesystem interface (see Resources).

As of Ubuntu 14.04, the cgroup manager (cgmanager) is available as another
cgroup administion interface. It's goal is to respond to dbus requests from
any user, allowing him to administer only those cgroups which have been
delegated to him.

[Overview] will describe cgroups in more detail. [Filesystem] will describe
the long-standing cgroups filesystem interface. [Manager] will describe the
cgroup manager.

# Overview {#cgroups-overview}

Cgroups are the generalized feature for grouping tasks. The actual resource
tracking and limits are implemented by subsystems. A hierarchy is a set of
subsystems mounted together. For instance, if the memory and devices
subsystems are mounted together under /sys/fs/cgroups/set1, then any task
which is in "/child1" will be subject to the corresponding limits of both
subsystems.

Each set of mounted subsystems consittutes a 'hierarchy'. With exceptions,
cgroups which are children of "/child1" will be subject to all limits placed
on "/child1", and their resource usage will be accounted to "/child1".

The existing subsystems include:

-   *cpusets*: fascilitate assigning a set of CPUS and memory nodes to
    cgroups. Tasks in a cpuset cgroup may only be scheduled on CPUS assigned
    to that cpuset.

-   *blkio*: limits per-cgroup block io.

-   *cpuacct*: provides per-cgroup cpu usage accounting.

-   *devices*: controls the ability of tasks to create or use devices nodes
    using either a blacklist or whitelist.

-   *freezer*: provides a way to 'freeze' and 'thaw' whole cgroups. Tasks in
    the cgroup will not be scheduled while they are frozen.

-   *hugetlb*: fascilitates limiting hugetlb usage per cgroup.

-   *memory*: allows memory, kernel memory, and swap usage to be tracked
    and limited.

-   *net\_cls*: provides an interface for tagging packets based on the
    sender cgroup. These tags can then be used by tc (traffic controller) to
    assign priorities.

-   *net\_prio*: allows setting network traffic priority on a
    per-cgroup basis.

-   *cpu*: enables setting of scheduling preferences on per-cgroup basis.

-   *perf\_event*: enables per-cpu mode to monitor only threads in
    certain cgroups.

In addition, named cgroups can be created with no bound subsystems for the
sake of process tracking. As an example, systemd does this to track services
and user sessions.

# Filesystem {#cgroups-fs}

A hierarchy is created by mounting an instance of the cgroup filesystem with
each of the desired subsystems listed as a mount option. For instance,

    mount -t cgroup -o devices,memory,freezer cgroup /cgroup1

would instantiate a hierarchy with the devices and memory cgroups comounted. A
child cgroup "child1" can be created using 'mkdir'

    mkdir /cgroup1/child1

and tasks can be moved into the new child cgroup by writing their process IDs
into the 'tasks' or 'cgroup.procs' file:

    sleep 100 &
    echo $! > /cgroup1/child1/cgroup.procs

Other administration is done through files in the cgroup directories. For
instance, to freeze all tasks in child1,

    echo FROZEN > /cgroup1/child1/freezer.state

A great deal of information about cgroups and its subsystems can be found
under the cgroups documentation directory in the kernel source tree (see
Resources).

# Delegation {#cgroups-delegation}

Cgroup files and directories can be owned by non-root users, enabling
delegation of cgroup administration. In general, the kernel enforces the
hierarchical constraints on limits, so that for instance if devices cgroup
`/child1` cannot access a disk drive, then `/child1/child2` cannot give itself
those rights.

As of Ubuntu 14.04, users are automatically placed in a set of cgroups which
they own, safely allowing them to contrain their own jobs using child cgroups.
This feature is relied upon, for instance, for unprivileged container creation
in lxc.

# Manager {#cgroups-manager}

The cgroup manager (cgmanager) provides a D-Bus service allowing programs and
users to administer cgroups without needing direct knowledge of or access to
the cgroup filesystem. For requests from tasks in the same namespaces as the
manager, the manager can directly perform the needed security checks to ensure
that requests are legitimate. For other requests - such as those from a task
in a container - enhanced D-Bus requests must be made, where process-, user-
and group-ids are passed as SCM\_CREDENTIALS, so that the kernel maps the
identifiers to their global host values.

To fascilitate the use of simple D-Bus calls from all users, a 'cgroup manager
proxy' (cgproxy) is automatically started when in a container. The proxy
accepts standard D-Bus requests from tasks in the same namespaces as itself,
and converts them to SCM-enhanced D-Bus requests which it passes on to the
cgmanager.

A simple example of creating a new cgroup in which to run a cpu-intensive
compile would look like:

    cgm create cpuset build1
    cgm movepid cpuset build1 $$
    cgm setvalue cpuset build1 cpuset.cpus 1
    make

# Resources {#cgroups-resources}

-   Manual pages referenced above can be found at:

        cgm
        cgconfig.conf
        cgmanager
        cgproxy

-   The upstream cgmanager project is hosted at [linuxcontainers.org].

-   The upstream kernel documentation page on cgroups can be seen [here].

-   The freedesktop.org control group usage guidelines can be seen [here][1].

  [Overview]: #cgroups-overview
  [Filesystem]: #cgroups-fs
  [Manager]: #cgroups-manager
  [linuxcontainers.org]: http://cgmanager.linuxcontainers.org
  [here]: https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/tree/Documentation/cgroups
  [1]: http://www.freedesktop.org/wiki/Software/systemd/PaxControlGroups/
