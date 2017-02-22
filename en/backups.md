# Backups

There are many ways to backup an Ubuntu installation. The most important thing
about backups is to develop a *backup plan* consisting of what to backup,
where to back it up to, and how to restore it.

The following sections discuss various ways of accomplishing these tasks.

# Shell Scripts {#backup-shellscripts}

One of the simplest ways to backup a system is using a *shell script*. For
example, a script can be used to configure which directories to backup, and
pass those directories as arguments to the tar utility, which creates an
archive file. The archive file can then be moved or copied to another
location. The archive can also be created on a remote file system such as an
*NFS* mount.

The tar utility creates one archive file out of many files or directories. tar
can also filter the files through compression utilities, thus reducing the
size of the archive file.

## Simple Shell Script {#backup-shellscript}

The following shell script uses tar to create an archive file on a remotely
mounted NFS file system. The archive filename is determined using additional
command line utilities.

    #!/bin/bash
    ####################################
    #
    # Backup to NFS mount script.
    #
    ####################################

    # What to backup. 
    backup_files="/home /var/spool/mail /etc /root /boot /opt"

    # Where to backup to.
    dest="/mnt/backup"

    # Create archive filename.
    day=$(date +%A)
    hostname=$(hostname -s)
    archive_file="$hostname-$day.tgz"

    # Print start status message.
    echo "Backing up $backup_files to $dest/$archive_file"
    date
    echo

    # Backup the files using tar.
    tar czf $dest/$archive_file $backup_files

    # Print end status message.
    echo
    echo "Backup finished"
    date

    # Long listing of files in $dest to check file sizes.
    ls -lh $dest

-   *\$backup\_files:* a variable listing which directories you would like to
    backup. The list should be customized to fit your needs.

-   *\$day:* a variable holding the day of the week (Monday, Tuesday,
    Wednesday, etc). This is used to create an archive file for each day of
    the week, giving a backup history of seven days. There are other ways to
    accomplish this including using the date utility.

-   *\$hostname:* variable containing the *short* hostname of the system.
    Using the hostname in the archive filename gives you the option of placing
    daily archive files from multiple systems in the same directory.

-   *\$archive\_file:* the full archive filename.

-   *\$dest:* destination of the archive file. The directory needs to be
    created and in this case *mounted* before executing the backup script. See
    [???] for details of using *NFS*.

-   *status messages:* optional messages printed to the console using the
    echo utility.

-   *tar czf \$dest/\$archive\_file \$backup\_files:* the tar command used to
    create the archive file.

    -   *c:* creates an archive.

    -   *z:* filter the archive through the gzip utility compressing
        the archive.

    -   *f:* output to an archive file. Otherwise the tar output will be sent
        to STDOUT.

-   *ls -lh \$dest:* optional statement prints a *-l* long listing in *-h*
    human readable format of the destination directory. This is useful for a
    quick file size check of the archive file. This check should not replace
    testing the archive file.

This is a simple example of a backup shell script; however there are many
options that can be included in such a script. See [References] for links to
resources providing more in-depth shell scripting information.

## Executing the Script {#backup-executing-shellscript}

### Executing from a Terminal {#backup-script-execute-shell}

The simplest way of executing the above backup script is to copy and paste the
contents into a file. `backup.sh` for example. The file must be made
executable:

    chmod u+x backup.sh

Then from a terminal prompt:

    sudo ./backup.sh

This is a great way to test the script to make sure everything works as
expected.

### Executing with cron {#backup-script-execute-cron}

The cron utility can be used to automate the script execution. The cron daemon
allows the execution of scripts, or commands, at a specified time and date.

cron is configured through entries in a `crontab` file. `crontab` files are
separated into fields:

    # m h dom mon dow   command

-   *m:* minute the command executes on, between 0 and 59.

-   *h:* hour the command executes on, between 0 and 23.

-   *dom:* day of month the command executes on.

-   *mon:* the month the command executes on, between 1 and 12.

-   *dow:* the day of the week the command executes on, between 0 and 7.
    Sunday may be specified by using 0 or 7, both values are valid.

-   *command:* the command to execute.

To add or change entries in a `crontab` file the crontab -e command should be
used. Also, the contents of a `crontab` file can be viewed using the crontab
-l command.

To execute the backup.sh script listed above using cron. Enter the following
from a terminal prompt:

    sudo crontab -e

> **Note**
>
> Using sudo with the crontab -e command edits the *root* user's crontab. This
> is necessary if you are backing up directories only the root user has access
> to.

Add the following entry to the `crontab` file:

    # m h dom mon dow   command
    0 0 * * * bash /usr/local/bin/backup.sh

The backup.sh script will now be executed every day at 12:00 am.

> **Note**
>
> The backup.sh script will need to be copied to the `/usr/local/bin/`
> directory in order for this entry to execute properly. The script can reside
> anywhere on the file system, simply change the script path appropriately.

For more in-depth crontab options see [References].

## Restoring from the Archive {#backup-shellscript-archive-testing}

Once an archive has been created it is important to test the archive. The
archive can be tested by listing the files it contains, but the best test is
to *restore* a file from the archive.

-   To see a listing of the archive contents. From a terminal prompt type:

        tar -tzvf /mnt/backup/host-Monday.tgz

-   To restore a file from the archive to a different directory enter:

        tar -xzvf /mnt/backup/host-Monday.tgz -C /tmp etc/hosts

    The *-C* option to tar redirects the extracted files to the
    specified directory. The above example will extract the `/etc/hosts` file
    to `/tmp/etc/hosts`. tar recreates the directory structure that
    it contains.

    Also, notice the leading *"/"* is left off the path of the file
    to restore.

-   To restore all files in the archive enter the following:

        cd /
        sudo tar -xzvf /mnt/backup/host-Monday.tgz

> **Note**
>
> This will overwrite the files currently on the file system.

## References {#backup-shellscript-references}

-   For more information on shell scripting see the [Advanced Bash-Scripting
    Guide]

-   The book [Teach Yourself Shell Programming in 24 Hours] is available
    online and a great resource for shell scripting.

-   The [CronHowto Wiki Page] contains details on advanced cron options.

-   See the [GNU tar Manual] for more tar options.

-   The Wikipedia [Backup Rotation Scheme] article contains information on
    other backup rotation schemes.

-   The shell script uses tar to create the archive, but there many other
    command line utilities that can be used. For example:

    -   [cpio]: used to copy files to and from archives.

    -   [dd]: part of the coreutils package. A low level utility that can copy
        data from one format to another.

    -   [rsnapshot]: a file system snapshot utility used to create copies of
        an entire file system.

    -   [rsync]: a flexible utility used to create incremental copies
        of files.

# Archive Rotation {#backups-shellscripts-rotation}

The shell script in [Shell Scripts] only allows for seven different archives.
For a server whose data doesn't change often, this may be enough. If the
server has a large amount of data, a more complex rotation scheme should be
used.

## Rotating NFS Archives {#backups-nfs-rotation}

In this section, the shell script will be slightly modified to implement a
grandfather-father-son rotation scheme (monthly-weekly-daily):

-   The rotation will do a *daily* backup Sunday through Friday.

-   On Saturday a *weekly* backup is done giving you four weekly backups
    a month.

-   The *monthly* backup is done on the first of the month rotating two
    monthly backups based on if the month is odd or even.

Here is the new script:

    #!/bin/bash
    ####################################
    #
    # Backup to NFS mount script with
    # grandfather-father-son rotation.
    #
    ####################################

    # What to backup. 
    backup_files="/home /var/spool/mail /etc /root /boot /opt"

    # Where to backup to.
    dest="/mnt/backup"

    # Setup variables for the archive filename.
    day=$(date +%A)
    hostname=$(hostname -s)

    # Find which week of the month 1-4 it is.
    day_num=$(date +%d)
    if (( $day_num <= 7 )); then
            week_file="$hostname-week1.tgz"
    elif (( $day_num > 7 && $day_num <= 14 )); then
            week_file="$hostname-week2.tgz"
    elif (( $day_num > 14 && $day_num <= 21 )); then
            week_file="$hostname-week3.tgz"
    elif (( $day_num > 21 && $day_num < 32 )); then
            week_file="$hostname-week4.tgz"
    fi

    # Find if the Month is odd or even.
    month_num=$(date +%m)
    month=$(expr $month_num % 2)
    if [ $month -eq 0 ]; then
            month_file="$hostname-month2.tgz"
    else
            month_file="$hostname-month1.tgz"
    fi

    # Create archive filename.
    if [ $day_num == 1 ]; then
        archive_file=$month_file
    elif [ $day != "Saturday" ]; then
            archive_file="$hostname-$day.tgz"
    else 
        archive_file=$week_file
    fi

    # Print start status message.
    echo "Backing up $backup_files to $dest/$archive_file"
    date
    echo

    # Backup the files using tar.
    tar czf $dest/$archive_file $backup_files

    # Print end status message.
    echo
    echo "Backup finished"
    date

    # Long listing of files in $dest to check file sizes.
    ls -lh $dest/

The script can be executed using the same methods as in
[Executing the Script].

It is good practice to take backup media off-site in case of a disaster. In
the shell script example the backup media is another server providing an NFS
share. In all likelihood taking the NFS server to another location would not
be practical. Depending upon connection speeds it may be an option to copy the
archive file over a WAN link to a server in another location.

Another option is to copy the archive file to an external hard drive which can
then be taken off-site. Since the price of external hard drives continue to
decrease, it may be cost-effective to use two drives for each archive level.
This would allow you to have one external drive attached to the backup server
and one in another location.

## Tape Drives {#backup-shellscript-tapedrive}

A tape drive attached to the server can be used instead of an NFS share. Using
a tape drive simplifies archive rotation, and makes taking the media off-site
easier as well.

When using a tape drive, the filename portions of the script aren't needed
because the data is sent directly to the tape device. Some commands to
manipulate the tape are needed. This is accomplished using mt, a magnetic tape
control utility part of the cpio package.

Here is the shell script modified to use a tape drive:

    #!/bin/bash
    ####################################
    #
    # Backup to tape drive script.
    #
    ####################################

    # What to backup. 
    backup_files="/home /var/spool/mail /etc /root /boot /opt"

    # Where to backup to.
    dest="/dev/st0"

    # Print start status message.
    echo "Backing up $backup_files to $dest"
    date
    echo

    # Make sure the tape is rewound.
    mt -f $dest rewind

    # Backup the files using tar.
    tar czf $dest $backup_files

    # Rewind and eject the tape.
    mt -f $dest rewoffl

    # Print end status message.
    echo
    echo "Backup finished"
    date

> **Note**
>
> The default device name for a SCSI tape drive is `/dev/st0`. Use the
> appropriate device path for your system.

Restoring from a tape drive is basically the same as restoring from a file.
Simply rewind the tape and use the device path instead of a file path. For
example to restore the `/etc/hosts` file to `/tmp/etc/hosts`:

    mt -f /dev/st0 rewind
    tar -xzf /dev/st0 -C /tmp etc/hosts

# Bacula

Bacula is a backup program enabling you to backup, restore, and verify data
across your network. There are Bacula clients for Linux, Windows, and Mac OS X
- making it a cross-platform network wide solution.

## Overview {#bacula-overview}

Bacula is made up of several components and services used to manage which
files to backup and backup locations:

-   Bacula Director: a service that controls all backup, restore, verify, and
    archive operations.

-   Bacula Console: an application allowing communication with the Director.
    There are three versions of the Console:

    -   Text based command line version.

    -   Gnome based GTK+ Graphical User Interface (GUI) interface.

    -   wxWidgets GUI interface.

-   Bacula File: also known as the Bacula Client program. This application is
    installed on machines to be backed up, and is responsible for the data
    requested by the Director.

-   Bacula Storage: the programs that perform the storage and recovery of data
    to the physical media.

-   Bacula Catalog: is responsible for maintaining the file indexes and volume
    databases for all files backed up, enabling quick location and restoration
    of archived files. The Catalog supports three different databases MySQL,
    PostgreSQL, and SQLite.

-   Bacula Monitor: allows the monitoring of the Director, File daemons, and
    Storage daemons. Currently the Monitor is only available as a GTK+
    GUI application.

These services and applications can be run on multiple servers and clients, or
they can be installed on one machine if backing up a single disk or volume.

## Installation {#bacula-installation}

> **Note**
>
> If using MySQL or PostgreSQL as your database, you should already have the
> services available. Bacula will not install them for you.

There are multiple packages containing the different Bacula components. To
install Bacula, from a terminal prompt enter:

    sudo apt install bacula

By default installing the bacula package will use a MySQL database for the
Catalog. If you want to use SQLite or PostgreSQL, for the Catalog, install
bacula-director-sqlite3 or bacula-director-pgsql respectively.

During the install process you will be asked to supply credentials for the
database *administrator* and the *bacula* database *owner*. The database
administrator will need to have the appropriate rights to create a database,
see [???][1] for more information.

## Configuration {#bacula-configuration}

Bacula configuration files are formatted based on *resources* comprising of
*directives* surrounded by “{}” braces. Each Bacula component has an
individual file in the `/etc/bacula` directory.

The various Bacula components must authorize themselves to each other. This is
accomplished using the *password* directive. For example, the *Storage*
resource password in the `/etc/bacula/bacula-dir.conf` file must match the
*Director* resource password in `/etc/bacula/bacula-sd.conf`.

By default the backup job named *Client1* is configured to archive the Bacula
Catalog. If you plan on using the server to backup more than one client you
should change the name of this job to something more descriptive. To change
the name edit `/etc/bacula/bacula-dir.conf`:

    #
    # Define the main nightly save backup job
    #   By default, this job will back up to disk in 
    Job {
      Name = "BackupServer"
      JobDefs = "DefaultJob"
      Write Bootstrap = "/var/lib/bacula/Client1.bsr"
    }

> **Note**
>
> The example above changes the job name to *BackupServer* matching the
> machine's host name. Replace “BackupServer” with your appropriate hostname,
> or other descriptive name.

The *Console* can be used to query the *Director* about jobs, but to use the
Console with a *non-root* user, the user needs to be in the *bacula* group. To
add a user to the bacula group enter the following from a terminal:

    sudo adduser $username bacula

> **Note**
>
> Replace *\$username* with the actual username. Also, if you are adding the
> current user to the group you should log out and back in for the new
> permissions to take effect.

## Localhost Backup {#bacula-localhost-backup}

This section describes how to backup specified directories on a single host to
a local tape drive.

-   First, the *Storage* device needs to be configured. Edit
    `/etc/bacula/bacula-sd.conf` add:

        Device {
          Name = "Tape Drive"
          Device Type = tape
          Media Type = DDS-4
          Archive Device = /dev/st0
          Hardware end of medium = No;
          AutomaticMount = yes;               # when device opened, read it
          AlwaysOpen = Yes;
          RemovableMedia = yes;
          RandomAccess = no;
          Alert Command = "sh -c 'tapeinfo -f %c | grep TapeAlert'"
        }

    The example is for a *DDS-4* tape drive. Adjust the “Media Type” and
    “Archive Device” to match your hardware.

    You could also uncomment one of the other examples in the file.

-   After editing `/etc/bacula/bacula-sd.conf` the Storage daemon will need to
    be restarted:

        sudo systemctl restart bacula-sd.service

-   Now add a *Storage* resource in `/etc/bacula/bacula-dir.conf` to use the
    new Device:

        # Definition of "Tape Drive" storage device
        Storage {
          Name = TapeDrive
          # Do not use "localhost" here    
          Address = backupserver               # N.B. Use a fully qualified name here
          SDPort = 9103
          Password = "Cv70F6pf1t6pBopT4vQOnigDrR0v3LT3Cgkiyjc"
          Device = "Tape Drive"
          Media Type = tape
        }

    The *Address* directive needs to be the Fully Qualified Domain Name (FQDN)
    of the server. Change *backupserver* to the actual host name.

    Also, make sure the *Password* directive matches the password string in
    `/etc/bacula/bacula-sd.conf`.

-   Create a new *FileSet*, which will determine what directories to backup,
    by adding:

        # LocalhostBacup FileSet.
        FileSet {
          Name = "LocalhostFiles"
          Include {
            Options {
              signature = MD5
              compression=GZIP
            }
            File = /etc
            File = /home
          }
        }

    This *FileSet* will backup the `/etc` and `/home` directories. The
    *Options* resource directives configure the FileSet to create an MD5
    signature for each file backed up, and to compress the files using GZIP.

-   Next, create a new *Schedule* for the backup job:

        # LocalhostBackup Schedule -- Daily.
        Schedule {
          Name = "LocalhostDaily"
          Run = Full daily at 00:01
        }

    The job will run every day at 00:01 or 12:01 am. There are many other
    scheduling options available.

-   Finally create the *Job*:

        # Localhost backup.
        Job {
          Name = "LocalhostBackup"
          JobDefs = "DefaultJob"
          Enabled = yes
          Level = Full
          FileSet = "LocalhostFiles"
          Schedule = "LocalhostDaily"
          Storage = TapeDrive
          Write Bootstrap = "/var/lib/bacula/LocalhostBackup.bsr"
        }  

    The job will do a *Full* backup every day to the tape drive.

-   Each tape used will need to have a *Label*. If the current tape does not
    have a label Bacula will send an email letting you know. To label a tape
    using the Console enter the following from a terminal:

        bconsole

-   At the Bacula Console prompt enter:

        label

-   You will then be prompted for the *Storage* resource:


        Automatically selected Catalog: MyCatalog
        Using Catalog "MyCatalog"
        The defined Storage resources are:
             1: File
             2: TapeDrive
        Select Storage resource (1-2):2

-   Enter the new *Volume* name:


        Enter new Volume name: Sunday
        Defined Pools:
             1: Default
             2: Scratch

    Replace *Sunday* with the desired label.

-   Now, select the *Pool*:


        Select the Pool (1-2): 1
        Connecting to Storage daemon TapeDrive at backupserver:9103 ...
        Sending label command for Volume "Sunday" Slot 0 ...

Congratulations, you have now configured *Bacula* to backup the localhost to
an attached tape drive.

## Resources {#bacula-resources}

-   For more *Bacula* configuration options, refer to [Bacula's
    Documentation].

-   The [Bacula Home Page] contains the latest Bacula news and developments.

-   Also, see the [Bacula Ubuntu Wiki] page.

  [???]: #network-file-system
  [References]: #backup-shellscript-references
  [Advanced Bash-Scripting Guide]: http://tldp.org/LDP/abs/html/
  [Teach Yourself Shell Programming in 24 Hours]: http://safari.samspublishing.com/0672323583
  [CronHowto Wiki Page]: https://help.ubuntu.com/community/CronHowto
  [GNU tar Manual]: http://www.gnu.org/software/tar/manual/index.html
  [Backup Rotation Scheme]: http://en.wikipedia.org/wiki/Backup_rotation_scheme
  [cpio]: http://www.gnu.org/software/cpio/
  [dd]: http://www.gnu.org/software/coreutils/
  [rsnapshot]: http://www.rsnapshot.org/
  [rsync]: http://www.samba.org/ftp/rsync/rsync.html
  [Shell Scripts]: #backup-shellscripts
  [Executing the Script]: #backup-executing-shellscript
  [1]: #mysql
  [Bacula's Documentation]: http://blog.bacula.org/documentation/documentation/
  [Bacula Home Page]: http://www.bacula.org/
  [Bacula Ubuntu Wiki]: https://help.ubuntu.com/community/Bacula
