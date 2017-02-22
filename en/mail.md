# Email Services

The process of getting an email from one person to another over a network or
the Internet involves many systems working together. Each of these systems
must be correctly configured for the process to work. The sender uses a *Mail
User Agent* (MUA), or email client, to send the message through one or more
*Mail Transfer Agents* (MTA), the last of which will hand it off to a *Mail
Delivery Agent* (MDA) for delivery to the recipient's mailbox, from which it
will be retrieved by the recipient's email client, usually via a POP3 or IMAP
server.

## Postfix
Postfix is the default Mail Transfer Agent (MTA) in Ubuntu. It attempts to be
fast and easy to administer and secure. It is compatible with the MTA
sendmail. This section explains how to install and configure postfix. It also
explains how to set it up as an SMTP server using a secure connection (for
sending emails securely).

!!! Note: This guide does not cover setting up Postfix *Virtual Domains*, for
information on Virtual Domains and other advanced configurations see
[References].

### Installation 
To install postfix run the following command:

```bash
sudo apt install postfix
```

Simply press return when the installation process asks questions, the
configuration will be done in greater detail in the next stage.

### Basic Configuration 
To configure postfix, run the following command:

```bash
sudo dpkg-reconfigure postfix
```

The user interface will be displayed. On each screen, select the following
values:

-   Internet Site

-   mail.example.com

-   steve

-   mail.example.com, localhost.localdomain, localhost

-   No

-   127.0.0.0/8 \[::ffff:127.0.0.0\]/104 \[::1\]/128 192.168.0.0/24

-   0

-   +

-   all

!!! Note: Replace mail.example.com with the domain for which you'll accept email,
192.168.0.0/24 with the actual network and class range of your mail server,
and steve with the appropriate username.

Now is a good time to decide which mailbox format you want to use. By default
Postfix will use **mbox** for the mailbox format. Rather than editing the
configuration file directly, you can use the `postconf` command to configure
all postfix parameters. The configuration parameters will be stored in
`/etc/postfix/main.cf` file. Later if you wish to re-configure a particular
parameter, you can either run the command or change it manually in the file.

To configure the mailbox format for **Maildir:**

```bash
sudo postconf -e 'home_mailbox = Maildir/'
```

!!! Note: This will place new mail in /home/*username*/Maildir so you will need to
configure your Mail Delivery Agent (MDA) to use the same path.

### SMTP Authentication 
SMTP-AUTH allows a client to identify itself through an authentication
mechanism (SASL). Transport Layer Security (TLS) should be used to encrypt the
authentication process. Once authenticated the SMTP server will allow the
client to relay mail.

Configure Postfix for SMTP-AUTH using SASL (Dovecot SASL):

```bash
sudo postconf -e 'smtpd_sasl_type = dovecot'
sudo postconf -e 'smtpd_sasl_path = private/auth'
sudo postconf -e 'smtpd_sasl_local_domain ='
sudo postconf -e 'smtpd_sasl_security_options = noanonymous'
sudo postconf -e 'broken_sasl_auth_clients = yes'
sudo postconf -e 'smtpd_sasl_auth_enable = yes'
sudo postconf -e 'smtpd_recipient_restrictions = \
permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination'
```

!!! Note: The *smtpd\_sasl\_path* configuration is a path relative to the Postfix
queue directory.

Next, generate or obtain a digital certificate for TLS. See [???] for details.
This example also uses a Certificate Authority (CA). For information on
generating a CA certificate see [???][1].

!!! Note: MUAs connecting to your mail server via TLS will need to recognize the
certificate used for TLS. This can either be done using a certificate from a
commercial CA or with a self-signed certificate that users manually
install/accept. For MTA to MTA TLS certficates are never validated without
advance agreement from the affected organizations. For MTA to MTA TLS,
unless local policy requires it, there is no reason not to use a self-signed
certificate. Refer to [???][2] for more details.

Once you have a certificate, configure Postfix to provide TLS encryption for
both incoming and outgoing mail:

```bash
sudo postconf -e 'smtp_tls_security_level = may'
sudo postconf -e 'smtpd_tls_security_level = may'
sudo postconf -e 'smtp_tls_note_starttls_offer = yes'
sudo postconf -e 'smtpd_tls_key_file = /etc/ssl/private/server.key'
sudo postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/server.crt'
sudo postconf -e 'smtpd_tls_loglevel = 1'
sudo postconf -e 'smtpd_tls_received_header = yes'
sudo postconf -e 'myhostname = mail.example.com'
```

If you are using your own *Certificate Authority* to sign the certificate
enter:

```bash
sudo postconf -e 'smtpd_tls_CAfile = /etc/ssl/certs/cacert.pem'
```

Again, for more details about certificates see [???].

!!! Note: After running all the commands, Postfix is configured for SMTP-AUTH and a
self-signed certificate has been created for TLS encryption.

Now, the file `/etc/postfix/main.cf` should look like this:

```bash
# See /usr/share/postfix/main.cf.dist for a commented, more complete
# version
```

```bash
smtpd_banner = $myhostname ESMTP $mail_name (Ubuntu)
biff = no
```

```bash
# appending .domain is the MUA's job.
append_dot_mydomain = no
```

```bash
# Uncomment the next line to generate "delayed mail" warnings
#delay_warning_time = 4h
```

```bash
myhostname = server1.example.com
alias_maps = hash:/etc/aliases
alias_database = hash:/etc/aliases
myorigin = /etc/mailname
mydestination = server1.example.com, localhost.example.com, localhost
relayhost =
mynetworks = 127.0.0.0/8
mailbox_command = procmail -a "$EXTENSION"
mailbox_size_limit = 0
recipient_delimiter = +
inet_interfaces = all
smtpd_sasl_local_domain =
smtpd_sasl_auth_enable = yes
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_recipient_restrictions =
permit_sasl_authenticated,permit_mynetworks,reject _unauth_destination
smtpd_tls_auth_only = no
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_key_file = /etc/ssl/private/smtpd.key
smtpd_tls_cert_file = /etc/ssl/certs/smtpd.crt
smtpd_tls_CAfile = /etc/ssl/certs/cacert.pem
smtpd_tls_loglevel = 1
smtpd_tls_received_header = yes
smtpd_tls_session_cache_timeout = 3600s
tls_random_source = dev:/dev/urandom
```

The postfix initial configuration is complete. Run the following command to
restart the postfix daemon:

```bash
sudo systemctl restart postfix.service
```

Postfix supports SMTP-AUTH as defined in [RFC2554]. It is based on [SASL].
However it is still necessary to set up SASL authentication before you can use
SMTP-AUTH.

### Configuring SASL 
Postfix supports two SASL implementations Cyrus SASL and Dovecot SASL. To
enable Dovecot SASL the dovecot-core package will need to be installed. From a
terminal prompt enter the following:

```bash
sudo apt install dovecot-core
```

Next you will need to edit `/etc/dovecot/conf.d/10-master.conf`. Change the
following:

```bash
service auth {
  # auth_socket_path points to this userdb socket by default. It's typically
  # used by dovecot-lda, doveadm, possibly imap process, etc. Its default
  # permissions make it readable only by root, but you may need to relax these
  # permissions. Users that have access to this socket are able to get a list
  # of all usernames and get results of everyone's userdb lookups.
  unix_listener auth-userdb {
    #mode = 0600
    #user = 
    #group = 
  }
```

```bash
  # Postfix smtp-auth
  unix_listener /var/spool/postfix/private/auth {
    mode = 0660
    user = postfix
    group = postfix
  }
```

In order to let Outlook clients use SMTP-AUTH, in the *authentication
mechanisms* section of /etc/dovecot/conf.d/10-auth.conf change this line:

```bash
auth_mechanisms = plain
```

To this:

```bash
auth_mechanisms = plain login
```

Once you have Dovecot configured restart it with:

```bash
sudo systemctl restart dovecot.service
```

### Mail-Stack Delivery
Another option for configuring Postfix for SMTP-AUTH is using the
mail-stack-delivery package (previously packaged as dovecot-postfix). This
package will install Dovecot and configure Postfix to use it for both SASL
authentication and as a Mail Delivery Agent (MDA). The package also configures
Dovecot for IMAP, IMAPS, POP3, and POP3S.

!!! Note: You may or may not want to run IMAP, IMAPS, POP3, or POP3S on your mail
server. For example, if you are configuring your server to be a mail
gateway, spam/virus filter, etc. If this is the case it may be easier to use
the above commands to configure Postfix for SMTP-AUTH.

To install the package, from a terminal prompt enter:

```bash
sudo apt install mail-stack-delivery
```

You should now have a working mail server, but there are a few options that
you may wish to further customize. For example, the package uses the
certificate and key from the ssl-cert package, and in a production environment
you should use a certificate and key generated for the host. See [???] for
more details.

Once you have a customized certificate and key for the host, change the
following options in `/etc/postfix/main.cf`:

```bash
smtpd_tls_cert_file = /etc/ssl/certs/ssl-mail.pem
smtpd_tls_key_file = /etc/ssl/private/ssl-mail.key
```

Then restart Postfix:

```bash
sudo systemctl restart postfix.service
```

### Testing 
SMTP-AUTH configuration is complete. Now it is time to test the setup.

To see if SMTP-AUTH and TLS work properly, run the following command:

```bash
telnet mail.example.com 25
```

After you have established the connection to the postfix mail server, type:

```bash
ehlo mail.example.com
```

If you see the following lines among others, then everything is working
perfectly. Type `quit` to exit.

```bash
250-STARTTLS
250-AUTH LOGIN PLAIN
250-AUTH=LOGIN PLAIN
250 8BITMIME
```

### Troubleshooting 
This section introduces some common ways to determine the cause if problems
arise.

#### Escaping chroot 
The Ubuntu postfix package will by default install into a *chroot* environment
for security reasons. This can add greater complexity when troubleshooting
problems.

To turn off the chroot operation locate for the following line in the
`/etc/postfix/master.cf` configuration file:

```bash
smtp      inet  n       -       -       -       -       smtpd
```

and modify it as follows:

```bash
smtp      inet  n       -       n       -       -       smtpd
```

You will then need to restart Postfix to use the new configuration. From a
terminal prompt enter:

```bash
sudo systemctl restart postfix.service
```

#### Smtps 
If you need smtps, edit `/etc/postfix/master.cf` and uncomment the following
line:

```bash
smtps     inet  n       -       -       -       -       smtpd
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_client_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```
```bash
      
```

#### Log Files 
Postfix sends all log messages to `/var/log/mail.log`. However error and
warning messages can sometimes get lost in the normal log output so they are
also logged to `/var/log/mail.err` and `/var/log/mail.warn` respectively.

To see messages entered into the logs in real time you can use the tail -f
command:

```bash
tail -f /var/log/mail.err
```

The amount of detail that is recorded in the logs can be increased. Below are
some configuration options for increasing the log level for some of the areas
covered above.

-   To increase *TLS* activity logging set the *smtpd\_tls\_loglevel* option
    to a value from 1 to 4.

```bash
    sudo postconf -e 'smtpd_tls_loglevel = 4'
```

-   If you are having trouble sending or receiving mail from a specific domain
    you can add the domain to the *debug\_peer\_list* parameter.

```bash
    sudo postconf -e 'debug_peer_list = problem.domain'
```

-   You can increase the verbosity of any Postfix daemon process by editing
    the `/etc/postfix/master.cf` and adding a *-v* after the entry. For
    example edit the *smtp* entry:

```bash
    smtp      unix  -       -       -       -       -       smtp -v
```

!!! Note: It is important to note that after making one of the logging changes above
the Postfix process will need to be reloaded in order to recognize the new
configuration: `sudo systemctl reload postfix.service`

-   To increase the amount of information logged when troubleshooting *SASL*
    issues you can set the following options in
    `/etc/dovecot/conf.d/10-logging.conf`

```bash
    auth_debug=yes
    auth_debug_passwords=yes
```

!!! Note: Just like Postfix if you change a Dovecot configuration the process will
need to be reloaded: `sudo systemctl reload dovecot.service`.

!!! Note: Some of the options above can drastically increase the amount of information
sent to the log files. Remember to return the log level back to normal after
you have corrected the problem. Then reload the appropriate daemon for the
new configuration to take affect.

#### References 
Administering a Postfix server can be a very complicated task. At some point
you may need to turn to the Ubuntu community for more experienced help.

A great place to ask for Postfix assistance, and get involved with the Ubuntu
Server community, is the *\#ubuntu-server* IRC channel on [freenode]. You can
also post a message to one of the [Web Forums].

For in depth Postfix information Ubuntu developers highly recommend: [The Book
of Postfix].

Finally, the [Postfix] website also has great documentation on all the
different configuration options available.

Also, the [Ubuntu Wiki Postfix] page has more information.

## Exim4
Exim4 is another Message Transfer Agent (MTA) developed at the University of
Cambridge for use on Unix systems connected to the Internet. Exim can be
installed in place of sendmail, although the configuration of exim is quite
different to that of sendmail.

### Installation 
To install exim4, run the following command:

```bash
sudo apt install exim4
```

### Configuration 
To configure Exim4, run the following command:

```bash
sudo dpkg-reconfigure exim4-config
```

The user interface will be displayed. The user interface lets you configure
many parameters. For example, In Exim4 the configuration files are split among
multiple files. If you wish to have them in one file you can configure
accordingly in this user interface.

All the parameters you configure in the user interface are stored in
`/etc/exim4/update-exim4.conf.conf` file. If you wish to re-configure, either
you re-run the configuration wizard or manually edit this file using your
favorite editor. Once you configure, you can run the following command to
generate the master configuration file:

```bash
sudo update-exim4.conf
```

The master configuration file, is generated and it is stored in
`/var/lib/exim4/config.autogenerated`.

!!! Warning: At any time, you should not edit the master configuration file,
`/var/lib/exim4/config.autogenerated` manually. It is updated automatically
every time you run `update-exim4.conf`

You can run the following command to start Exim4 daemon.

```bash
sudo systemctl start exim4.service
```

### SMTP Authentication 
This section covers configuring Exim4 to use SMTP-AUTH with TLS and SASL.

The first step is to create a certificate for use with TLS. Enter the
following into a terminal prompt:

```bash
sudo /usr/share/doc/exim4-base/examples/exim-gencert
```

Now Exim4 needs to be configured for TLS by editing
`/etc/exim4/conf.d/main/03_exim4-config_tlsoptions` add the following:

```bash
MAIN_TLS_ENABLE = yes
```

Next you need to configure Exim4 to use the saslauthd for authentication. Edit
`/etc/exim4/conf.d/auth/30_exim4-config_examples` and uncomment the
*plain\_saslauthd\_server* and *login\_saslauthd\_server* sections:

```bash
 plain_saslauthd_server:
   driver = plaintext
   public_name = PLAIN
   server_condition = ${if saslauthd{{$auth2}{$auth3}}{1}{0}}
   server_set_id = $auth2
   server_prompts = :
   .ifndef AUTH_SERVER_ALLOW_NOTLS_PASSWORDS
   server_advertise_condition = ${if eq{$tls_cipher}{}{}{*}}
   .endif
#
 login_saslauthd_server:
   driver = plaintext
   public_name = LOGIN
   server_prompts = "Username:: : Password::"
   # don't send system passwords over unencrypted connections
   server_condition = ${if saslauthd{{$auth1}{$auth2}}{1}{0}}
   server_set_id = $auth1
   .ifndef AUTH_SERVER_ALLOW_NOTLS_PASSWORDS
   server_advertise_condition = ${if eq{$tls_cipher}{}{}{*}}
   .endif
```

Additionally, in order for outside mail client to be able to connect to new
exim server, new user needs to be added into exim by using the following
commands.

```bash
sudo /usr/share/doc/exim4-base/examples/exim-adduser
```

Users should protect the new exim password files with the following commands.

```bash
sudo chown root:Debian-exim /etc/exim4/passwd
sudo chmod 640 /etc/exim4/passwd
```

Finally, update the Exim4 configuration and restart the service:

```bash
sudo update-exim4.conf
sudo systemctl restart exim4.service
```

### Configuring SASL 
This section provides details on configuring the saslauthd to provide
authentication for Exim4.

The first step is to install the sasl2-bin package. From a terminal prompt
enter the following:

```bash
sudo apt install sasl2-bin
```

To configure saslauthd edit the /etc/default/saslauthd configuration file and
set START=no to:

```bash
START=yes
```

Next the *Debian-exim* user needs to be part of the *sasl* group in order for
Exim4 to use the saslauthd service:

```bash
sudo adduser Debian-exim sasl
```

Now start the saslauthd service:

```bash
sudo systemctl start saslauthd.service
```

Exim4 is now configured with SMTP-AUTH using TLS and SASL authentication.

### References 
-   See [exim.org] for more information.

-   There is also an [Exim4 Book] available.

-   Another resource is the [Exim4 Ubuntu Wiki] page.

## Dovecot Server
Dovecot is a Mail Delivery Agent, written with security primarily in mind. It
supports the major mailbox formats: mbox or Maildir. This section explain how
to set it up as an imap or pop3 server.

### Installation 
To install dovecot, run the following command in the command prompt:

```bash
sudo apt install dovecot-imapd dovecot-pop3d
```

### Configuration 
To configure dovecot, you can edit the file `/etc/dovecot/dovecot.conf`. You
can choose the protocol you use. It could be pop3, pop3s (pop3 secure), imap
and imaps (imap secure). A description of these protocols is beyond the scope
of this guide. For further information, refer to the Wikipedia articles on
[POP3] and [IMAP].

IMAPS and POP3S are more secure that the simple IMAP and POP3 because they use
SSL encryption to connect. Once you have chosen the protocol, amend the
following line in the file `/etc/dovecot/dovecot.conf`:

```bash
protocols = pop3 pop3s imap imaps
```

Next, choose the mailbox you would like to use. Dovecot supports **maildir**
and **mbox** formats. These are the most commonly used mailbox formats. They
both have their own benefits and are discussed on [the Dovecot web site].

Once you have chosen your mailbox type, edit the file
`/etc/dovecot/conf.d/10-mail.conf` and change the following line:

```bash
mail_location = maildir:~/Maildir # (for maildir)
or
mail_location = mbox:~/mail:INBOX=/var/spool/mail/%u # (for mbox)
```

!!! Note: You should configure your Mail Transport Agent (MTA) to transfer the
incoming mail to this type of mailbox if it is different from the one you
have configured.

Once you have configured dovecot, restart the dovecot daemon in order to test
your setup:

```bash
sudo systemctl restart dovecot.service
```

If you have enabled imap, or pop3, you can also try to log in with the
commands `telnet localhost pop3` or `telnet localhost imap2`. If you see
something like the following, the installation has been successful:

```bash
bhuvan@rainbow:~$ telnet localhost pop3
Trying 127.0.0.1...
Connected to localhost.localdomain.
Escape character is '^]'.
+OK Dovecot ready.
```

### Dovecot SSL Configuration 
To configure dovecot to use SSL, you can edit the file
`/etc/dovecot/conf.d/10-ssl.conf` and amend following lines:


```bash
ssl = yes
ssl_cert = </etc/ssl/certs/dovecot.pem
ssl_key = </etc/ssl/private/dovecot.pem
```

You can get the SSL certificate from a Certificate Issuing Authority or you
can create self signed SSL certificate. The latter is a good option for email,
because SMTP clients rarely complain about "self-signed certificates". Please
refer to [???] for details about how to create self signed SSL certificate.
Once you create the certificate, you will have a key file and a certificate
file. Please copy them to the location pointed in the
`/etc/dovecot/conf.d/10-ssl.conf` configuration file.

### Firewall Configuration for an Email Server
To access your mail server from another computer, you must configure your
firewall to allow connections to the server on the necessary ports.

-   IMAP - 143

-   IMAPS - 993

-   POP3 - 110

-   POP3S - 995

### References 
-   See the [Dovecot website] for more information.

-   Also, the [Dovecot Ubuntu Wiki] page has more details.

## Mailman
Mailman is an open source program for managing electronic mail discussions and
e-newsletter lists. Many open source mailing lists (including all the [Ubuntu
mailing lists]) use Mailman as their mailing list software. It is powerful and
easy to install and maintain.

### Installation 
Mailman provides a web interface for the administrators and users, using an
external mail server to send and receive emails. It works perfectly with the
following mail servers:

-   Postfix

-   Exim

-   Sendmail

-   Qmail

We will see how to install and configure Mailman with, the Apache web server,
and either the Postfix or Exim mail server. If you wish to install Mailman
with a different mail server, please refer to the references section.

!!! Note: You only need to install one mail server and Postfix is the default Ubuntu
Mail Transfer Agent.

#### Apache2 
To install apache2 you refer to [???][3] for details.

#### Postfix 
For instructions on installing and configuring Postfix refer to [Postfix][4]

#### Exim4 
To install Exim4 refer to [Exim4].

Once exim4 is installed, the configuration files are stored in the
`/etc/exim4` directory. In Ubuntu, by default, the exim4 configuration files
are split across different files. You can change this behavior by changing the
following variable in the `/etc/exim4/update-exim4.conf` file:

```bash
dc_use_split_config='true'
```

#### Mailman 
To install Mailman, run following command at a terminal prompt:

```bash
sudo apt install mailman 
```

It copies the installation files in /var/lib/mailman directory. It installs
the CGI scripts in /usr/lib/cgi-bin/mailman directory. It creates *list* linux
user. It creates the *list* linux group. The mailman process will be owned by
this user.

### Configuration 
This section assumes you have successfully installed mailman, apache2, and
postfix or exim4. Now you just need to configure them.

#### Apache2 
An example Apache configuration file comes with Mailman and is placed in
`/etc/mailman/apache.conf`. In order for Apache to use the config file it
needs to be copied to `/etc/apache2/sites-available`:

```bash
sudo cp /etc/mailman/apache.conf /etc/apache2/sites-available/mailman.conf
```

This will setup a new Apache *VirtualHost* for the Mailman administration
site. Now enable the new configuration and restart Apache:

```bash
sudo a2ensite mailman.conf
sudo systemctl restart apache2.service
```

Mailman uses apache2 to render its CGI scripts. The mailman CGI scripts are
installed in the /usr/lib/cgi-bin/mailman directory. So, the mailman url will
be http://hostname/cgi-bin/mailman/. You can make changes to the
`/etc/apache2/sites-available/mailman.conf` file if you wish to change this
behavior.

#### Postfix 
For Postfix integration, we will associate the domain lists.example.com with
the mailing lists. Please replace *lists.example.com* with the domain of your
choosing.

You can use the postconf command to add the necessary configuration to
`/etc/postfix/main.cf`:

```bash
sudo postconf -e 'relay_domains = lists.example.com'
sudo postconf -e 'transport_maps = hash:/etc/postfix/transport'
sudo postconf -e 'mailman_destination_recipient_limit = 1'
```

In `/etc/postfix/master.cf` double check that you have the following
transport:

```bash
mailman   unix  -       n       n       -       -       pipe
  flags=FR user=list argv=/usr/lib/mailman/bin/postfix-to-mailman.py
  ${nexthop} ${user}
```

It calls the *postfix-to-mailman.py* script when a mail is delivered to a
list.

Associate the domain lists.example.com to the Mailman transport with the
transport map. Edit the file `/etc/postfix/transport`:

```bash
lists.example.com      mailman:
```

Now have Postfix build the transport map by entering the following from a
terminal prompt:

```bash
sudo postmap -v /etc/postfix/transport
```

Then restart Postfix to enable the new configurations:

```bash
sudo systemctl restart postfix.service
```

#### Exim4 
Once Exim4 is installed, you can start the Exim server using the following
command from a terminal prompt:

```bash
sudo systemctl start exim4.service
```

In order to make mailman work with Exim4, you need to configure Exim4. As
mentioned earlier, by default, Exim4 uses multiple configuration files of
different types. For details, please refer to the [Exim] web site. To run
mailman, we should add new a configuration file to the following configuration
types:

-   Main

-   Transport

-   Router

Exim creates a master configuration file by sorting all these mini
configuration files. So, the order of these configuration files is very
important.

#### Main 
All the configuration files belonging to the main type are stored in the
`/etc/exim4/conf.d/main/` directory. You can add the following content to a
new file, named `04_exim4-config_mailman`:

```bash
# start
# Home dir for your Mailman installation -- aka Mailman's prefix
# directory.
# On Ubuntu this should be "/var/lib/mailman"
# This is normally the same as ~mailman
MM_HOME=/var/lib/mailman
#
# User and group for Mailman, should match your --with-mail-gid
# switch to Mailman's configure script.  Value is normally "mailman"
MM_UID=list
MM_GID=list
#
# Domains that your lists are in - colon separated list
# you may wish to add these into local_domains as well
domainlist mm_domains=hostname.com
#
# -=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=-=
#
# These values are derived from the ones above and should not need
# editing unless you have munged your mailman installation
#
# The path of the Mailman mail wrapper script
MM_WRAP=MM_HOME/mail/mailman
#
# The path of the list config file (used as a required file when
# verifying list addresses)
MM_LISTCHK=MM_HOME/lists/${lc::$local_part}/config.pck
# end
```

#### Transport 
All the configuration files belonging to transport type are stored in the
`/etc/exim4/conf.d/transport/` directory. You can add the following content to
a new file named `
40_exim4-config_mailman`:

```bash
  mailman_transport:
   driver = pipe
   command = MM_WRAP \
               '${if def:local_part_suffix \
                     {${sg{$local_part_suffix}{-(\\w+)(\\+.*)?}{\$1}}} \
                     {post}}' \
               $local_part
    current_directory = MM_HOME
    home_directory = MM_HOME
    user = MM_UID
    group = MM_GID
```

#### Router 
All the configuration files belonging to router type are stored in the
`/etc/exim4/conf.d/router/` directory. You can add the following content in to
a new file named `101_exim4-config_mailman`:

```bash
  mailman_router:
   driver = accept
   require_files = MM_HOME/lists/$local_part/config.pck
   local_part_suffix_optional
   local_part_suffix = -bounces : -bounces+* : \
                       -confirm+* : -join : -leave : \
                       -owner : -request : -admin
   transport = mailman_transport
```

!!! Warning: The order of main and transport configuration files can be in any order.
But, the order of router configuration files must be the same. This
particular file must appear before the 200\_exim4-config\_primary file.
These two configuration files contain same type of information. The first
file takes the precedence. For more details, please refer to the references
section.

#### Mailman 
Once mailman is installed, you can run it using the following command:

```bash
sudo systemctl start mailman.service
```

Once mailman is installed, you should create the default mailing list. Run the
following command to create the mailing list:

```bash
sudo /usr/sbin/newlist mailman
```

```bash
  Enter the email address of the person running the list: bhuvan at ubuntu.com
  Initial mailman password:
  To finish creating your mailing list, you must edit your /etc/aliases (or
  equivalent) file by adding the following lines, and possibly running the
  `newaliases' program:
```

```bash
  ## mailman mailing list
  mailman:              "|/var/lib/mailman/mail/mailman post mailman"
  mailman-admin:        "|/var/lib/mailman/mail/mailman admin mailman"
  mailman-bounces:      "|/var/lib/mailman/mail/mailman bounces mailman"
  mailman-confirm:      "|/var/lib/mailman/mail/mailman confirm mailman"
  mailman-join:         "|/var/lib/mailman/mail/mailman join mailman"
  mailman-leave:        "|/var/lib/mailman/mail/mailman leave mailman"
  mailman-owner:        "|/var/lib/mailman/mail/mailman owner mailman"
  mailman-request:      "|/var/lib/mailman/mail/mailman request mailman"
  mailman-subscribe:    "|/var/lib/mailman/mail/mailman subscribe mailman"
  mailman-unsubscribe:  "|/var/lib/mailman/mail/mailman unsubscribe mailman"
```

```bash
  Hit enter to notify mailman owner...
```

```bash
  # 
```

We have configured either Postfix or Exim4 to recognize all emails from
mailman. So, it is not mandatory to make any new entries in `/etc/aliases`. If
you have made any changes to the configuration files, please ensure that you
restart those services before continuing to next section.

!!! Note: The Exim4 does not use the above aliases to forward mails to Mailman, as it
uses a *discover* approach. To suppress the aliases while creating the list,
you can add *MTA=None* line in Mailman configuration file,
`/etc/mailman/mm_cfg.py`.

### Administration 
We assume you have a default installation. The mailman cgi scripts are still
in the /usr/lib/cgi-bin/mailman/ directory. Mailman provides a web based
administration facility. To access this page, point your browser to the
following url:

http://hostname/cgi-bin/mailman/admin

The default mailing list, *mailman*, will appear in this screen. If you click
the mailing list name, it will ask for your authentication password. If you
enter the correct password, you will be able to change administrative settings
of this mailing list. You can create a new mailing list using the command line
utility (`/usr/sbin/newlist`). Alternatively, you can create a new mailing
list using the web interface.

### Users 
Mailman provides a web based interface for users. To access this page, point
your browser to the following url:

http://hostname/cgi-bin/mailman/listinfo

The default mailing list, *mailman*, will appear in this screen. If you click
the mailing list name, it will display the subscription form. You can enter
your email address, name (optional), and password to subscribe. An email
invitation will be sent to you. You can follow the instructions in the email
to subscribe.

### References 
[GNU Mailman - Installation Manual]

[HOWTO - Using Exim 4 and Mailman 2.1 together]

Also, see the [Mailman Ubuntu Wiki] page.

## Mail Filtering
One of the largest issues with email today is the problem of Unsolicited Bulk
Email (UBE). Also known as SPAM, such messages may also carry viruses and
other forms of malware. According to some reports these messages make up the
bulk of all email traffic on the Internet.

This section will cover integrating Amavisd-new, Spamassassin, and ClamAV with
the Postfix Mail Transport Agent (MTA). Postfix can also check email validity
by passing it through external content filters. These filters can sometimes
determine if a message is spam without needing to process it with more
resource intensive applications. Two common filters are opendkim and
python-policyd-spf.

-   Amavisd-new is a wrapper program that can call any number of content
    filtering programs for spam detection, antivirus, etc.

-   Spamassassin uses a variety of mechanisms to filter email based on the
    message content.

-   ClamAV is an open source antivirus application.

-   opendkim implements a Sendmail Mail Filter (Milter) for the DomainKeys
    Identified Mail (DKIM) standard.

-   python-policyd-spf enables Sender Policy Framework (SPF) checking
    with Postfix.

This is how the pieces fit together:

-   An email message is accepted by Postfix.

-   The message is passed through any external filters opendkim and
    python-policyd-spf in this case.

-   Amavisd-new then processes the message.

-   ClamAV is used to scan the message. If the message contains a virus
    Postfix will reject the message.

-   Clean messages will then be analyzed by Spamassassin to find out if the
    message is spam. Spamassassin will then add X-Header lines allowing
    Amavisd-new to further manipulate the message.

For example, if a message has a Spam score of over fifty the message could be
automatically dropped from the queue without the recipient ever having to be
bothered. Another, way to handle flagged messages is to deliver them to the
Mail User Agent (MUA) allowing the user to deal with the message as they see
fit.

### Installation 
See [Postfix][4] for instructions on installing and configuring Postfix.

To install the rest of the applications enter the following from a terminal
prompt:

```bash
sudo apt install amavisd-new spamassassin clamav-daemon
sudo apt install opendkim postfix-policyd-spf-python
```

There are some optional packages that integrate with Spamassassin for better
spam detection:

```bash
sudo apt install pyzor razor
```

Along with the main filtering applications compression utilities are needed to
process some email attachments:

```bash
sudo apt install arj cabextract cpio lha nomarch pax rar unrar unzip zip
```

!!! Note: If some packages are not found, check that the *multiverse* repository is
enabled in `/etc/apt/sources.list`
If you make changes to the file, be sure to run `sudo apt update` before
trying to install again.

### Configuration 
Now configure everything to work together and filter email.

#### ClamAV 
The default behaviour of ClamAV will fit our needs. For more ClamAV
configuration options, check the configuration files in `/etc/clamav`.

Add the *clamav* user to the *amavis* group in order for Amavisd-new to have
the appropriate access to scan files:

```bash
sudo adduser clamav amavis
sudo adduser amavis clamav
```

#### Spamassassin 
Spamassassin automatically detects optional components and will use them if
they are present. This means that there is no need to configure pyzor and
razor.

Edit `/etc/default/spamassassin` to activate the Spamassassin daemon. Change
*ENABLED=0* to:

```bash
ENABLED=1
```

Now start the daemon:

```bash
sudo systemctl start spamassassin.service
```

#### Amavisd-new 
First activate spam and antivirus detection in Amavisd-new by editing
`/etc/amavis/conf.d/15-content_filter_mode`:

```bash
use strict;
```

```bash
# You can modify this file to re-enable SPAM checking through spamassassin
# and to re-enable antivirus checking.
```

```bash
#
# Default antivirus checking mode
# Uncomment the two lines below to enable it
#
```

```bash
@bypass_virus_checks_maps = (
   \%bypass_virus_checks, \@bypass_virus_checks_acl, \$bypass_virus_checks_re);
```


```bash
#
# Default SPAM checking mode
# Uncomment the two lines below to enable it
#
```

```bash
@bypass_spam_checks_maps = (
   \%bypass_spam_checks, \@bypass_spam_checks_acl, \$bypass_spam_checks_re);
```

```bash
1;  # insure a defined return
```

Bouncing spam can be a bad idea as the return address is often faked. The
default behaviour is to instead discard. This is configured in
`/etc/amavis/conf.d/20-debian_defaults` where *\$final\_spam\_destiny* is set
to D\_DISCARD rather than D\_BOUNCE.

Additionally, you may want to adjust the following options to flag more
messages as spam:

```bash
$sa_tag_level_deflt = -999; # add spam info headers if at, or above that level
$sa_tag2_level_deflt = 6.0; # add 'spam detected' headers at that level
$sa_kill_level_deflt = 21.0; # triggers spam evasive actions
$sa_dsn_cutoff_level = 4; # spam level beyond which a DSN is not sent
```

If the server's *hostname* is different from the domain's MX record you may
need to manually set the *\$myhostname* option. Also, if the server receives
mail for multiple domains the *@local\_domains\_acl* option will need to be
customized. Edit the `/etc/amavis/conf.d/50-user` file:

```bash
$myhostname = 'mail.example.com';
@local_domains_acl = ( "example.com", "example.org" );
```

If you want to cover multiple domains you can use the following in
the`/etc/amavis/conf.d/50-user`

```bash
@local_domains_acl = qw(.);
```

After configuration Amavisd-new needs to be restarted:

```bash
sudo systemctl restart amavis.service
```

##### DKIM Whitelist 
Amavisd-new can be configured to automatically *Whitelist* addresses from
domains with valid Domain Keys. There are some pre-configured domains in the
`/etc/amavis/conf.d/40-policy_banks`.

There are multiple ways to configure the Whitelist for a domain:

-   *'example.com' =&gt; 'WHITELIST',*: will whitelist any address from the
    "example.com" domain.

-   *'.example.com' =&gt; 'WHITELIST',*: will whitelist any address from any
    *subdomains* of "example.com" that have a valid signature.

-   *'.example.com/@example.com' =&gt; 'WHITELIST',*: will whitelist
    subdomains of "example.com" that use the signature of *example.com* the
    parent domain.

-   *'./@example.com' =&gt; 'WHITELIST',*: adds addresses that have a valid
    signature from "example.com". This is usually used for discussion groups
    that sign their messages.

A domain can also have multiple Whitelist configurations. After editing the
file, restart amavisd-new:

```bash
sudo systemctl restart amavis.service
```

!!! Note: In this context, once a domain has been added to the Whitelist the message
will not receive any anti-virus or spam filtering. This may or may not be
the intended behavior you wish for a domain.

#### Postfix 
For Postfix integration, enter the following from a terminal prompt:

```bash
sudo postconf -e 'content_filter = smtp-amavis:[127.0.0.1]:10024'
```

Next edit `/etc/postfix/master.cf` and add the following to the end of the
file:

```bash
smtp-amavis     unix    -       -       -       -       2       smtp
        -o smtp_data_done_timeout=1200
        -o smtp_send_xforward_command=yes
        -o disable_dns_lookups=yes
        -o max_use=20
```

```bash
127.0.0.1:10025 inet    n       -       -       -       -       smtpd
        -o content_filter=
        -o local_recipient_maps=
        -o relay_recipient_maps=
        -o smtpd_restriction_classes=
        -o smtpd_delay_reject=no
        -o smtpd_client_restrictions=permit_mynetworks,reject
        -o smtpd_helo_restrictions=
        -o smtpd_sender_restrictions=
        -o smtpd_recipient_restrictions=permit_mynetworks,reject
        -o smtpd_data_restrictions=reject_unauth_pipelining
        -o smtpd_end_of_data_restrictions=
        -o mynetworks=127.0.0.0/8
        -o smtpd_error_sleep_time=0
        -o smtpd_soft_error_limit=1001
        -o smtpd_hard_error_limit=1000
        -o smtpd_client_connection_count_limit=0
        -o smtpd_client_connection_rate_limit=0
        -o receive_override_options=no_header_body_checks,no_unknown_recipient_checks,no_milters
```

Also add the following two lines immediately below the *"pickup"* transport
service:

```bash
         -o content_filter=
         -o receive_override_options=no_header_body_checks
```

This will prevent messages that are generated to report on spam from being
classified as spam.

Now restart Postfix:

```bash
sudo systemctl restart postfix.service
```

Content filtering with spam and virus detection is now enabled.

#### Amavisd-new and Spamassassin 
When integrating Amavisd-new with Spamassassin, if you choose to disable the
bayes filtering by editing `/etc/spamassassin/local.cf` and use cron to update
the nightly rules, the result can cause a situation where a large amount of
error messages are sent to the *amavis* user via the amavisd-new cron job.

There are several ways to handle this situation:

-   Configure your MDA to filter messages you do not wish to see.

-   Change `/usr/sbin/amavisd-new-cronjob` to check for *use\_bayes 0*. For
    example, edit `/usr/sbin/amavisd-new-cronjob` and add the following to the
    top before the *test* statements:

```bash
    egrep -q "^[ \t]*use_bayes[ \t]*0" /etc/spamassassin/local.cf && exit 0
```

### Testing 
First, test that the Amavisd-new SMTP is listening:

```bash
telnet localhost 10024
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 [127.0.0.1] ESMTP amavisd-new service ready
^]
```

In the Header of messages that go through the content filter you should see:

```bash
X-Spam-Level: 
X-Virus-Scanned: Debian amavisd-new at example.com
X-Spam-Status: No, hits=-2.3 tagged_above=-1000.0 required=5.0 tests=AWL, BAYES_00
X-Spam-Level: 
```

!!! Note: Your output will vary, but the important thing is that there are
*X-Virus-Scanned* and *X-Spam-Status* entries.

### Troubleshooting 
The best way to figure out why something is going wrong is to check the log
files.

-   For instructions on Postfix logging see the [Troubleshooting] section.

-   Amavisd-new uses Syslog to send messages to `/var/log/mail.log`. The
    amount of detail can be increased by adding the *\$log\_level* option to
    `/etc/amavis/conf.d/50-user`, and setting the value from 1 to 5.

```bash
    $log_level = 2;
```

```bash
> **Note**
>
> When the Amavisd-new log output is increased Spamassassin log output is
> also increased.
```

-   The ClamAV log level can be increased by editing `/etc/clamav/clamd.conf`
    and setting the following option:

```bash
    LogVerbose true
```

```bash
By default ClamAV will send log messages to `/var/log/clamav/clamav.log`.
```

!!! Note: After changing an applications log settings remember to restart the service
for the new settings to take affect. Also, once the issue you are
troubleshooting is resolved it is a good idea to change the log settings
back to normal.

### References 
For more information on filtering mail see the following links:

-   [Amavisd-new Documentation]

-   [ClamAV Documentation] and [ClamAV Wiki]

-   [Spamassassin Wiki]

-   [Pyzor Homepage]

-   [Razor Homepage]

-   [DKIM.org]

-   [Postfix Amavis New]

Also, feel free to ask questions in the *\#ubuntu-server* IRC channel on
[freenode].

  [References]: #postfix-references
  [???]: #certificates-and-security
  [1]: #certificate-authority
  [2]: #creating-a-self-signed-certificate
  [RFC2554]: http://www.ietf.org/rfc/rfc2554.txt
  [SASL]: http://www.ietf.org/rfc/rfc2222.txt
  [freenode]: http://freenode.net
  [Web Forums]: http://www.ubuntu.com/support/community/webforums
  [The Book of Postfix]: http://www.postfix-book.com/
  [Postfix]: http://www.postfix.org/documentation.html
  [Ubuntu Wiki Postfix]: https://help.ubuntu.com/community/Postfix
  [exim.org]: http://www.exim.org/
  [Exim4 Book]: http://www.uit.co.uk/content/exim-smtp-mail-server
  [Exim4 Ubuntu Wiki]: https://help.ubuntu.com/community/Exim4
  [POP3]: http://en.wikipedia.org/wiki/POP3
  [IMAP]: http://en.wikipedia.org/wiki/Internet_Message_Access_Protocol
  [the Dovecot web site]: http://wiki2.dovecot.org/MailboxFormat
  [Dovecot website]: http://www.dovecot.org/
  [Dovecot Ubuntu Wiki]: https://help.ubuntu.com/community/Dovecot
  [Ubuntu mailing lists]: http://lists.ubuntu.com
  [3]: #http-installation
  [4]: #postfix
  [Exim4]: #exim4
  [Exim]: http://www.exim.org
  [GNU Mailman - Installation Manual]: http://www.list.org/mailman-install/index.html
  [HOWTO - Using Exim 4 and Mailman 2.1 together]: http://www.exim.org/howto/mailman21.html
  [Mailman Ubuntu Wiki]: https://help.ubuntu.com/community/Mailman
  [Troubleshooting]: #postfix-troubleshooting
  [Amavisd-new Documentation]: http://www.ijs.si/software/amavisd/amavisd-new-docs.html
  [ClamAV Documentation]: http://www.clamav.net/doc/latest/html/
  [ClamAV Wiki]: http://wiki.clamav.net/Main/WebHome
  [Spamassassin Wiki]: http://wiki.apache.org/spamassassin/
  [Pyzor Homepage]: http://sourceforge.net/apps/trac/pyzor/
  [Razor Homepage]: http://razor.sourceforge.net/
  [DKIM.org]: http://dkim.org/
  [Postfix Amavis New]: https://help.ubuntu.com/community/PostfixAmavisNew
