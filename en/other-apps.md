# Other Useful Applications

There are many very useful applications developed by the Ubuntu Server Team,
and others that are well integrated with Ubuntu Server Edition, that might not
be well known. This chapter will showcase some useful applications that can
make administering an Ubuntu server, or many Ubuntu servers, that much easier.

# pam\_motd

When logging into an Ubuntu server you may have noticed the informative
Message Of The Day (MOTD). This information is obtained and displayed using a
couple of packages:

-   *landscape-common:* provides the core libraries of landscape-client, which
    is needed to manage systems with [Landscape] (proprietary). Yet the
    package also includes the landscape-sysinfo utility which is responsible
    for displaying core system data involving cpu, memory, disk space, etc.
    For instance:


              System load:  0.0               Processes:           76
              Usage of /:   30.2% of 3.11GB   Users logged in:     1
              Memory usage: 20%               IP address for eth0: 10.153.107.115
              Swap usage:   0%

              Graph this data and manage this system at https://landscape.canonical.com/

    > **Note**
    >
    > You can run landscape-sysinfo manually at any time.

-   *update-notifier-common:* provides information on available package
    updates, impending filesystem checks (fsck), and required reboots (e.g.:
    after a kernel upgrade).

pam\_motd executes the scripts in `/etc/update-motd.d` in order based on the
number prepended to the script. The output of the scripts is written to
`/var/run/motd`, keeping the numerical order, then concatenated with
`/etc/motd.tail`.

You can add your own dynamic information to the MOTD. For example, to add
local weather information:

-   First, install the weather-util package:

        sudo apt install weather-util

-   The weather utility uses METAR data from the National Oceanic and
    Atmospheric Administration and forecasts from the National
    Weather Service. In order to find local information you will need the
    4-character ICAO location indicator. This can be determined by browsing to
    the [National Weather Service] site.

    Although the National Weather Service is a United States government agency
    there are weather stations available world wide. However, local weather
    information for all locations outside the U.S. may not be available.

-   Create `/usr/local/bin/local-weather`, a simple shell script to use
    weather with your local ICAO indicator:

        #!/bin/sh
        #
        #
        # Prints the local weather information for the MOTD.
        #
        #

        # Replace KINT with your local weather station.
        # Local stations can be found here: http://www.weather.gov/tg/siteloc.shtml

        echo
        weather -i KINT
        echo

-   Make the script executable:

        sudo chmod 755 /usr/local/bin/local-weather

-   Next, create a symlink to `/etc/update-motd.d/98-local-weather`:

        sudo ln -s /usr/local/bin/local-weather /etc/update-motd.d/98-local-weather

-   Finally, exit the server and re-login to view the new MOTD.

You should now be greeted with some useful information, and some information
about the local weather that may not be quite so useful. Hopefully the
local-weather example demonstrates the flexibility of pam\_motd.

## Resources {#pam_motd-resources}

-   See the [update-motd man page] for more options available to update-motd.

-   The Debian Package of the Day [weather] article has more details about
    using the weatherutility.

# etckeeper

etckeeper allows the contents of `/etc` to be stored in a Version Control
System (VCS) repository. It integrates with APT and automatically commits
changes to `/etc` when packages are installed or upgraded. Placing `/etc`
under version control is considered an industry best practice, and the goal of
etckeeper is to make this process as painless as possible.

Install etckeeper by entering the following in a terminal:

    sudo apt install etckeeper

The main configuration file, `/etc/etckeeper/etckeeper.conf`, is fairly
simple. The main option is which VCS to use and by default etckeeper is
configured to use Bazaar. The repository is automatically initialized (and
committed for the first time) during package installation. It is possible to
undo this by entering the following command:

    sudo etckeeper uninit

By default, etckeeper will commit uncommitted changes made to /etc daily. This
can be disabled using the AVOID\_DAILY\_AUTOCOMMITS configuration option. It
will also automatically commit changes before and after package installation.
For a more precise tracking of changes, it is recommended to commit your
changes manually, together with a commit message, using:

    sudo etckeeper commit "..Reason for configuration change.."

Using bzr's VCS commands you can view log information:

    sudo bzr log /etc/passwd

To demonstrate the integration with the package management system (APT),
install postfix:

    sudo apt install postfix

When the installation is finished, all the postfix configuration files should
be committed to the repository:

    Committing to: /etc/
    added aliases.db
    modified group
    modified group-
    modified gshadow
    modified gshadow-
    modified passwd
    modified passwd-
    added postfix
    added resolvconf
    added rsyslog.d
    modified shadow
    modified shadow-
    added init.d/postfix
    added network/if-down.d/postfix
    added network/if-up.d/postfix
    added postfix/dynamicmaps.cf
    added postfix/main.cf
    added postfix/master.cf
    added postfix/post-install
    added postfix/postfix-files
    added postfix/postfix-script
    added postfix/sasl
    added ppp/ip-down.d
    added ppp/ip-down.d/postfix
    added ppp/ip-up.d/postfix
    added rc0.d/K20postfix
    added rc1.d/K20postfix
    added rc2.d/S20postfix
    added rc3.d/S20postfix
    added rc4.d/S20postfix
    added rc5.d/S20postfix
    added rc6.d/K20postfix
    added resolvconf/update-libc.d
    added resolvconf/update-libc.d/postfix
    added rsyslog.d/postfix.conf
    added ufw/applications.d/postfix
    Committed revision 2.

For an example of how etckeeper tracks manual changes, add new a host to
`/etc/hosts`. Using bzr you can see which files have been modified:

    sudo bzr status /etc/
    modified:
      hosts

Now commit the changes:

    sudo etckeeper commit "added new host"

For more information on bzr see [???].

## Resources {#etckeeper-resources}

-   See the [etckeeper] site for more details on using etckeeper.

-   For the latest news and information about bzr see the [bzr] web site.

# Byobu

One of the most useful applications for any system administrator is an xterm
multiplexor such as screen or tmux. It allows for the execution of multiple
shells in one terminal. To make some of the advanced multiplexor features more
user-friendly and provide some useful information about the system, the byobu
package was created. It acts as a wrapper to these programs. By default Byobu
uses tmux (if installed) but this can be changed by the user.

Invoke it simply with:

    byobu

Now bring up the configuration menu. By default this is done by pressing the
*F9* key. This will allow you to:

-   View the Help menu

-   Change Byobu's background color

-   Change Byobu's foreground color

-   Toggle status notifications

-   Change the key binding set

-   Change the escape sequence

-   Create new windows

-   Manage the default windows

-   Byobu currently does not launch at login (toggle on)

The *key bindings* determine such things as the escape sequence, new window,
change window, etc. There are two key binding sets to choose from *f-keys* and
*screen-escape-keys*. If you wish to use the original key bindings choose the
*none* set.

byobu provides a menu which displays the Ubuntu release, processor
information, memory information, and the time and date. The effect is similar
to a desktop menu.

Using the *"Byobu currently does not launch at login (toggle on)"* option will
cause byobu to be executed any time a terminal is opened. Changes made to
byobu are on a per user basis, and will not affect other users on the system.

One difference when using byobu is the *scrollback* mode. Press the *F7* key
to enter scrollback mode. Scrollback mode allows you to navigate past output
using *vi* like commands. Here is a quick list of movement commands:

-   *h* - Move the cursor left by one character

-   *j* - Move the cursor down by one line

-   *k* - Move the cursor up by one line

-   *l* - Move the cursor right by one character

-   *0* - Move to the beginning of the current line

-   *\$* - Move to the end of the current line

-   *G* - Moves to the specified line (defaults to the end of the buffer)

-   */* - Search forward

-   *?* - Search backward

-   *n* - Moves to the next match, either forward or backward

## Resources {#byobu-resources}

-   For more information on screen see the [screen web site].

-   And the [Ubuntu Wiki screen] page.

-   Also, see the byobu [project page] for more information.

  [Landscape]: http://landscape.canonical.com/
  [National Weather Service]: http://www.weather.gov/tg/siteloc.shtml
  [update-motd man page]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/update-motd.5.html
  [weather]: http://debaday.debian.net/2007/10/04/weather-check-weather-conditions-and-forecasts-on-the-command-line/
  [???]: #bazaar
  [etckeeper]: http://etckeeper.branchable.com/
  [bzr]: http://bazaar-vcs.org/
  [screen web site]: http://www.gnu.org/software/screen/
  [Ubuntu Wiki screen]: https://help.ubuntu.com/community/Screen
  [project page]: https://launchpad.net/byobu
