# Appendix {#serverguide-appendix}

## Reporting Bugs in Ubuntu Server Edition 
The Ubuntu Project, and thus Ubuntu Server, uses [Launchpad] as its
bugtracker. In order to file a bug, you will need a Launchpad account. [Create
one here] if necessary.

### Reporting Bugs With apport-cli 
The preferred way to report a bug is with the apport-cli command. It must be
invoked on the machine affected by the bug because it collects information
from the system on which it is being run and publishes it to the bug report on
Launchpad. Getting that information to Launchpad can therefore be a challenge
if the system is not running a desktop environment in order to use a browser
(common with servers) or if it does not have Internet access. The steps to
take in these situations are described below.

!!! Note: The commands apport-cli and ubuntu-bug should give the same results on a CLI
server. The latter is actually a symlink to apport-bug which is intelligent
enough to know whether a desktop environment is in use and will choose
apport-cli if not. Since server systems tend to be CLI-only apport-cli was
chosen from the outset in this guide.

Bug reports in Ubuntu need to be filed against a specific software package, so
the name of the package (source package or program name/path) affected by the
bug needs to be supplied to apport-cli:

```bash
apport-cli PACKAGENAME
```

!!! Note: See [???] for more information about packages in Ubuntu.

Once apport-cli has finished gathering information you will be asked what to
do with it. For instance, to report a bug in vim:

```bash
apport-cli vim
```

```bash
*** Collecting problem information
```

```bash
The collected information can be sent to the developers to improve the
application. This might take a few minutes.
...
```

```bash
*** Send problem report to the developers?
```

```bash
After the problem report has been sent, please fill out the form in the
automatically opened web browser.
```

```bash
What would you like to do? Your options are:
  S: Send report (2.8 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C):
```

The first three options are described below:

-   **Send:** submits the collected information to Launchpad as part of the
    process of filing a new bug report. You will be given the opportunity to
    describe the bug in your own words.


```bash
    *** Uploading problem information
```

```bash
    The collected information is being sent to the bug tracking system.
    This might take a few minutes.
    94%
```

```bash
    *** To continue, you must visit the following URL:
```

```bash
      https://bugs.launchpad.net/ubuntu/+source/vim/+filebug/09b2495a-e2ab-11e3-879b-68b5996a96c8?
```

```bash
    You can launch a browser now, or copy this URL into a browser on another computer.
```


```bash
    Choices:
      1: Launch a browser now
      C: Cancel
    Please choose (1/C):  1
```

```bash
The browser that will be used when choosing '1' will be the one known on
the system as www-browser via the [Debian alternatives system]. Examples
of text-based browsers to install include links, elinks, lynx, and w3m.
You can also manually point an existing browser at the given URL.
```

-   **View:** displays the collected information on the screen for review.
    This can be a lot of information. Press 'Enter' to scroll by screenful.
    Press 'q' to quit and return to the choice menu.

-   **Keep:** writes the collected information to disk. The resulting file can
    be later used to file the bug report, typically after transferring it to
    another Ubuntu system.

```bash
    What would you like to do? Your options are:
      S: Send report (2.8 KB)
      V: View report
      K: Keep report file for sending later or copying to somewhere else
      I: Cancel and ignore future crashes of this program version
      C: Cancel
    Please choose (S/V/K/I/C): k
    Problem report file: /tmp/apport.vim.1pg92p02.apport
```

```bash
To report the bug, get the file onto an internet-enabled Ubuntu system and
apply apport-cli to it. This will cause the menu to appear immediately
(the information is already collected). You should then press 's' to send:
```

```bash
    apport-cli apport.vim.1pg92p02.apport
```

```bash
To directly save a report to disk (without menus) you can do:
```

```bash
    apport-cli vim --save apport.vim.test.apport
```

```bash
Report names should end in *.apport* .
```

```bash
> **Note**
>
> If this internet-enabled system is non-Ubuntu/Debian, apport-cli is not
> available so the bug will need to be created manually. An apport report
> is also not to be included as an attachment to a bug either so it is
> completely useless in this scenario.
```

### Reporting Application Crashes 
The software package that provides the apport-cli utility, apport, can be
configured to automatically capture the state of a crashed application. This
is enabled by default (in `/etc/default/apport`).

After an application crashes, if enabled, apport will store a crash report
under `/var/crash`:

```bash
-rw-r----- 1 peter    whoopsie 150K Jul 24 16:17 _usr_lib_x86_64-linux-gnu_libmenu-cache2_libexec_menu-cached.1000.crash
```

Use the apport-cli command without arguments to process any pending crash
reports. It will offer to report them one by one.

```bash
apport-cli
```

```bash
*** Send problem report to the developers?
```

```bash
After the problem report has been sent, please fill out the form in the
automatically opened web browser.
```

```bash
What would you like to do? Your options are:
  S: Send report (153.0 KB)
  V: View report
  K: Keep report file for sending later or copying to somewhere else
  I: Cancel and ignore future crashes of this program version
  C: Cancel
Please choose (S/V/K/I/C): s
```

If you send the report, as was done above, the prompt will be returned
immediately and the `/var/crash` directory will then contain 2 extra files:

```bash
-rw-r----- 1 peter    whoopsie 150K Jul 24 16:17 _usr_lib_x86_64-linux-gnu_libmenu-cache2_libexec_menu-cached.1000.crash
-rw-rw-r-- 1 peter    whoopsie    0 Jul 24 16:37 _usr_lib_x86_64-linux-gnu_libmenu-cache2_libexec_menu-cached.1000.upload
-rw------- 1 whoopsie whoopsie    0 Jul 24 16:37 _usr_lib_x86_64-linux-gnu_libmenu-cache2_libexec_menu-cached.1000.uploaded
```

Sending in a crash report like this will not immediately result in the
creation of a new public bug. The report will be made private on Launchpad,
meaning that it will be visible to only a limited set of bug triagers. These
triagers will then scan the report for possible private data before creating a
public bug.

### Resources 
-   See the [Reporting Bugs] Ubuntu wiki page.

-   Also, the [Apport] page has some useful information. Though some of it
    pertains to using a GUI.

  [Launchpad]: https://launchpad.net/
  [Create one here]: https://help.launchpad.net/YourAccount/NewAccount
  [???]: #package-management
  [Debian alternatives system]: http://manpages.ubuntu.com/manpages/en/man8/update-alternatives.8.html
  [Reporting Bugs]: https://help.ubuntu.com/community/ReportingBugs
  [Apport]: https://wiki.ubuntu.com/Apport
