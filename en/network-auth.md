# Network Authentication

This section applies LDAP to network authentication and authorization.

# OpenLDAP Server

The Lightweight Directory Access Protocol, or LDAP, is a protocol for querying
and modifying a X.500-based directory service running over TCP/IP. The current
LDAP version is LDAPv3, as defined in [RFC4510], and the implementation in
Ubuntu is OpenLDAP."

So the LDAP protocol accesses LDAP directories. Here are some key concepts and
terms:

-   A LDAP directory is a tree of data *entries* that is hierarchical in
    nature and is called the Directory Information Tree (DIT).

-   An entry consists of a set of *attributes*.

-   An attribute has a *type* (a name/description) and one or more *values*.

-   Every attribute must be defined in at least one *objectClass*.

-   Attributes and objectclasses are defined in *schemas* (an objectclass is
    actually considered as a special kind of attribute).

-   Each entry has a unique identifier: its *Distinguished Name* (DN or dn).
    This, in turn, consists of a *Relative Distinguished Name* (RDN) followed
    by the parent entry's DN.

-   The entry's DN is not an attribute. It is not considered part of the
    entry itself.

> **Note**
>
> The terms *object*, *container*, and *node* have certain connotations but
> they all essentially mean the same thing as *entry*, the technically correct
> term.

For example, below we have a single entry consisting of 11 attributes where
the following is true:

-   DN is "cn=John Doe,dc=example,dc=com"

-   RDN is "cn=John Doe"

-   parent DN is "dc=example,dc=com"

<!-- -->

     dn: cn=John Doe,dc=example,dc=com
     cn: John Doe
     givenName: John
     sn: Doe
     telephoneNumber: +1 888 555 6789
     telephoneNumber: +1 888 555 1232
     mail: john@example.com
     manager: cn=Larry Smith,dc=example,dc=com
     objectClass: inetOrgPerson
     objectClass: organizationalPerson
     objectClass: person
     objectClass: top

The above entry is in *LDIF* format (LDAP Data Interchange Format). Any
information that you feed into your DIT must also be in such a format. It is
defined in [RFC2849].

Although this guide will describe how to use it for central authentication,
LDAP is good for anything that involves a large number of access requests to a
mostly-read, attribute-based (name:value) backend. Examples include an address
book, a list of email addresses, and a mail server's configuration.

## Installation {#openldap-server-installation}

Install the OpenLDAP server daemon and the traditional LDAP management
utilities. These are found in packages slapd and ldap-utils respectively.

The installation of slapd will create a working configuration. In particular,
it will create a database instance that you can use to store your data.
However, the suffix (or base DN) of this instance will be determined from the
domain name of the localhost. If you want something different, edit
`/etc/hosts` and replace the domain name with one that will give you the
suffix you desire. For instance, if you want a suffix of *dc=example,dc=com*
then your file would have a line similar to this:

    127.0.1.1       hostname.example.com    hostname

You can revert the change after package installation.

> **Note**
>
> This guide will use a database suffix of *dc=example,dc=com*.

Proceed with the install:

    sudo apt install slapd ldap-utils

Since Ubuntu 8.10 slapd is designed to be configured within slapd itself by
dedicating a separate DIT for that purpose. This allows one to dynamically
configure slapd without the need to restart the service. This configuration
database consists of a collection of text-based LDIF files located under
`/etc/ldap/slapd.d`. This way of working is known by several names: the
slapd-config method, the RTC method (Real Time Configuration), or the
cn=config method. You can still use the traditional flat-file method
(slapd.conf) but it's not recommended; the functionality will be eventually
phased out.

> **Note**
>
> Ubuntu now uses the *slapd-config* method for slapd configuration and this
> guide reflects that.

During the install you were prompted to define administrative credentials.
These are LDAP-based credentials for the *rootDN* of your database instance.
By default, this user's DN is *cn=admin,dc=example,dc=com*. Also by default,
there is no administrative account created for the slapd-config database and
you will therefore need to authenticate externally to LDAP in order to access
it. We will see how to do this later on.

Some classical schemas (cosine, nis, inetorgperson) come built-in with slapd
nowadays. There is also an included "core" schema, a pre-requisite for any
schema to work.

## Post-install Inspection {#openldap-server-postinstall}

The installation process set up 2 DITs. One for slapd-config and one for your
own data (dc=example,dc=com). Let's take a look.

-   This is what the slapd-config database/DIT looks like. Recall that this
    database is LDIF-based and lives under `/etc/ldap/slapd.d`:


            /etc/ldap/slapd.d/
            /etc/ldap/slapd.d/cn=config
            /etc/ldap/slapd.d/cn=config/cn=module{0}.ldif
            /etc/ldap/slapd.d/cn=config/cn=schema
            /etc/ldap/slapd.d/cn=config/cn=schema/cn={0}core.ldif
            /etc/ldap/slapd.d/cn=config/cn=schema/cn={1}cosine.ldif
            /etc/ldap/slapd.d/cn=config/cn=schema/cn={2}nis.ldif
            /etc/ldap/slapd.d/cn=config/cn=schema/cn={3}inetorgperson.ldif
            /etc/ldap/slapd.d/cn=config/cn=schema.ldif
            /etc/ldap/slapd.d/cn=config/olcBackend={0}hdb.ldif
            /etc/ldap/slapd.d/cn=config/olcDatabase={0}config.ldif
            /etc/ldap/slapd.d/cn=config/olcDatabase={-1}frontend.ldif
            /etc/ldap/slapd.d/cn=config/olcDatabase={1}hdb.ldif
            /etc/ldap/slapd.d/cn=config.ldif

    > **Note**
    >
    > Do not edit the slapd-config database directly. Make changes via the
    > LDAP protocol (utilities).

-   This is what the slapd-config DIT looks like via the LDAP protocol:

    > **Caution**
    >
    > On Ubuntu server 14.10, and possibly higher, the following command may
    > not work due to a [bug]

        sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=config dn

        dn: cn=config

        dn: cn=module{0},cn=config

        dn: cn=schema,cn=config

        dn: cn={0}core,cn=schema,cn=config

        dn: cn={1}cosine,cn=schema,cn=config

        dn: cn={2}nis,cn=schema,cn=config

        dn: cn={3}inetorgperson,cn=schema,cn=config

        dn: olcBackend={0}hdb,cn=config

        dn: olcDatabase={-1}frontend,cn=config

        dn: olcDatabase={0}config,cn=config

        dn: olcDatabase={1}hdb,cn=config

    Explanation of entries:

    -   *cn=config*: global settings

    -   *cn=module{0},cn=config*: a dynamically loaded module

    -   *cn=schema,cn=config*: contains hard-coded system-level schema

    -   *cn={0}core,cn=schema,cn=config*: the hard-coded core schema

    -   *cn={1}cosine,cn=schema,cn=config*: the cosine schema

    -   *cn={2}nis,cn=schema,cn=config*: the nis schema

    -   *cn={3}inetorgperson,cn=schema,cn=config*: the inetorgperson schema

    -   *olcBackend={0}hdb,cn=config*: the 'hdb' backend storage type

    -   *olcDatabase={-1}frontend,cn=config*: frontend database, default
        settings for other databases

    -   *olcDatabase={0}config,cn=config*: slapd configuration
        database (cn=config)

    -   *olcDatabase={1}hdb,cn=config*: your database
        instance (dc=examle,dc=com)

-   This is what the dc=example,dc=com DIT looks like:

        ldapsearch -x -LLL -H ldap:/// -b dc=example,dc=com dn

        dn: dc=example,dc=com

        dn: cn=admin,dc=example,dc=com

    Explanation of entries:

    -   *dc=example,dc=com*: base of the DIT

    -   *cn=admin,dc=example,dc=com*: administrator (rootDN) for this DIT (set
        up during package install)

## Modifying/Populating your Database {#openldap-server-populate}

Let's introduce some content to our database. We will add the following:

-   a node called *People* (to store users)

-   a node called *Groups* (to store groups)

-   a group called *miners*

-   a user called *john*

Create the following LDIF file and call it `add_content.ldif`:

    dn: ou=People,dc=example,dc=com
    objectClass: organizationalUnit
    ou: People

    dn: ou=Groups,dc=example,dc=com
    objectClass: organizationalUnit
    ou: Groups

    dn: cn=miners,ou=Groups,dc=example,dc=com
    objectClass: posixGroup
    cn: miners
    gidNumber: 5000

    dn: uid=john,ou=People,dc=example,dc=com
    objectClass: inetOrgPerson
    objectClass: posixAccount
    objectClass: shadowAccount
    uid: john
    sn: Doe
    givenName: John
    cn: John Doe
    displayName: John Doe
    uidNumber: 10000
    gidNumber: 5000
    userPassword: johnldap
    gecos: John Doe
    loginShell: /bin/bash
    homeDirectory: /home/john

> **Note**
>
> It's important that uid and gid values in your directory do not collide with
> local values. Use high number ranges, such as starting at 5000. By setting
> the uid and gid values in ldap high, you also allow for easier control of
> what can be done with a local user vs a ldap one. More on that later.

Add the content:

    ldapadd -x -D cn=admin,dc=example,dc=com -W -f add_content.ldif

    Enter LDAP Password: ********
    adding new entry "ou=People,dc=example,dc=com"

    adding new entry "ou=Groups,dc=example,dc=com"

    adding new entry "cn=miners,ou=Groups,dc=example,dc=com"

    adding new entry "uid=john,ou=People,dc=example,dc=com"

We can check that the information has been correctly added with the ldapsearch
utility:

    ldapsearch -x -LLL -b dc=example,dc=com 'uid=john' cn gidNumber

    dn: uid=john,ou=People,dc=example,dc=com
    cn: John Doe
    gidNumber: 5000

Explanation of switches:

-   *-x:* "simple" binding; will not use the default SASL method

-   *-LLL:* disable printing extraneous information

-   *uid=john:* a "filter" to find the john user

-   *cn gidNumber:* requests certain attributes to be displayed (the default
    is to show all attributes)

## Modifying the slapd Configuration Database {#openldap-configuration}

The slapd-config DIT can also be queried and modified. Here are a few
examples.

-   Use ldapmodify to add an "Index" (DbIndex attribute) to your
    {1}hdb,cn=config database (dc=example,dc=com). Create a file, call it
    `uid_index.ldif`, with the following contents:

        dn: olcDatabase={1}hdb,cn=config
        add: olcDbIndex
        olcDbIndex: uid eq,pres,sub

    Then issue the command:

        sudo ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f uid_index.ldif

        modifying entry "olcDatabase={1}hdb,cn=config"

    You can confirm the change in this way:

        sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
        cn=config '(olcDatabase={1}hdb)' olcDbIndex

        dn: olcDatabase={1}hdb,cn=config
        olcDbIndex: objectClass eq
        olcDbIndex: uid eq,pres,sub

-   Let's add a schema. It will first need to be converted to LDIF format. You
    can find unconverted schemas in addition to converted ones in the
    `/etc/ldap/schema` directory.

    > **Note**
    >
    > -   It is not trivial to remove a schema from the slapd-config database.
    >     Practice adding schemas on a test system.
    >
    > -   Before adding any schema, you should check which schemas are already
    >     installed (shown is a default, out-of-the-box output):
    >
    >         sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
    >         cn=schema,cn=config dn
    >
    >         dn: cn=schema,cn=config
    >
    >         dn: cn={0}core,cn=schema,cn=config
    >
    >         dn: cn={1}cosine,cn=schema,cn=config
    >
    >         dn: cn={2}nis,cn=schema,cn=config
    >
    >         dn: cn={3}inetorgperson,cn=schema,cn=config
    >
    In the following example we'll add the CORBA schema.

    Create the conversion configuration file `schema_convert.conf` containing
    the following lines:

        include /etc/ldap/schema/core.schema
        include /etc/ldap/schema/collective.schema
        include /etc/ldap/schema/corba.schema
        include /etc/ldap/schema/cosine.schema
        include /etc/ldap/schema/duaconf.schema
        include /etc/ldap/schema/dyngroup.schema
        include /etc/ldap/schema/inetorgperson.schema
        include /etc/ldap/schema/java.schema
        include /etc/ldap/schema/misc.schema
        include /etc/ldap/schema/nis.schema
        include /etc/ldap/schema/openldap.schema
        include /etc/ldap/schema/ppolicy.schema
        include /etc/ldap/schema/ldapns.schema
        include /etc/ldap/schema/pmi.schema

    Create the output directory `ldif_output`.

    Determine the index of the schema:

        slapcat -f schema_convert.conf -F ldif_output -n 0 | grep corba,cn=schema

        cn={1}corba,cn=schema,cn=config

    > **Note**
    >
    > When slapd ingests objects with the same parent DN it will create an
    > *index* for that object. An index is contained within braces: {X}.

    Use slapcat to perform the conversion:

        slapcat -f schema_convert.conf -F ldif_output -n0 -H \
        ldap:///cn={1}corba,cn=schema,cn=config -l cn=corba.ldif

    The converted schema is now in `cn=corba.ldif`

    Edit `cn=corba.ldif` to arrive at the following attributes:

        dn: cn=corba,cn=schema,cn=config
        ...
        cn: corba

    Also remove the following lines from the bottom:

        structuralObjectClass: olcSchemaConfig
        entryUUID: 52109a02-66ab-1030-8be2-bbf166230478
        creatorsName: cn=config
        createTimestamp: 20110829165435Z
        entryCSN: 20110829165435.935248Z#000000#000#000000
        modifiersName: cn=config
        modifyTimestamp: 20110829165435Z

    Your attribute values will vary.

    Finally, use ldapadd to add the new schema to the slapd-config DIT:

        sudo ldapadd -Q -Y EXTERNAL -H ldapi:/// -f cn\=corba.ldif

        adding new entry "cn=corba,cn=schema,cn=config"

    Confirm currently loaded schemas:

        sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config dn

        dn: cn=schema,cn=config

        dn: cn={0}core,cn=schema,cn=config

        dn: cn={1}cosine,cn=schema,cn=config

        dn: cn={2}nis,cn=schema,cn=config

        dn: cn={3}inetorgperson,cn=schema,cn=config

        dn: cn={4}corba,cn=schema,cn=config

> **Note**
>
> For external applications and clients to authenticate using LDAP they will
> each need to be specifically configured to do so. Refer to the appropriate
> client-side documentation for details.

## Logging {#openldap-server-logging}

Activity logging for slapd is indispensible when implementing an
OpenLDAP-based solution yet it must be manually enabled after software
installation. Otherwise, only rudimentary messages will appear in the logs.
Logging, like any other slapd configuration, is enabled via the slapd-config
database.

OpenLDAP comes with multiple logging subsystems (levels) with each one
containing the lower one (additive). A good level to try is *stats*. The
[slapd-config] man page has more to say on the different subsystems.

Create the file `logging.ldif` with the following contents:

    dn: cn=config
    changetype: modify
    replace: olcLogLevel
    olcLogLevel: stats

Implement the change:

    sudo ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f logging.ldif

This will produce a significant amount of logging and you will want to
throttle back to a less verbose level once your system is in production. While
in this verbose mode your host's syslog engine (rsyslog) may have a hard time
keeping up and may drop messages:

    rsyslogd-2177: imuxsock lost 228 messages from pid 2547 due to rate-limiting

You may consider a change to rsyslog's configuration. In `/etc/rsyslog.conf`,
put:

    # Disable rate limiting
    # (default is 200 messages in 5 seconds; below we make the 5 become 0)
    $SystemLogRateLimitInterval 0

And then restart the rsyslog daemon:

    sudo systemctl restart syslog.service

## Replication {#openldap-server-replication}

The LDAP service becomes increasingly important as more networked systems
begin to depend on it. In such an environment, it is standard practice to
build redundancy (high availability) into LDAP to prevent havoc should the
LDAP server become unresponsive. This is done through *LDAP replication*.

Replication is achieved via the *Syncrepl* engine. This allows changes to be
synchronized using a *Consumer* - *Provider* model. The specific kind of
replication we will implement in this guide is a combination of the following
modes: *refreshAndPersist* and *delta-syncrepl*. This has the Provider push
changed entries to the Consumer as soon as they're made but, in addition, only
actual changes will be sent, not entire entries.

### Provider Configuration {#openldap-provider-configuration}

Begin by configuring the *Provider*.

Create an LDIF file with the following contents and name it
`provider_sync.ldif`:

    # Add indexes to the frontend db.
    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: entryCSN eq
    -
    add: olcDbIndex
    olcDbIndex: entryUUID eq

    #Load the syncprov and accesslog modules.
    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: syncprov
    -
    add: olcModuleLoad
    olcModuleLoad: accesslog

    # Accesslog database definitions
    dn: olcDatabase={2}hdb,cn=config
    objectClass: olcDatabaseConfig
    objectClass: olcHdbConfig
    olcDatabase: {2}hdb
    olcDbDirectory: /var/lib/ldap/accesslog
    olcSuffix: cn=accesslog
    olcRootDN: cn=admin,dc=example,dc=com
    olcDbIndex: default eq
    olcDbIndex: entryCSN,objectClass,reqEnd,reqResult,reqStart

    # Accesslog db syncprov.
    dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
    changetype: add
    objectClass: olcOverlayConfig
    objectClass: olcSyncProvConfig
    olcOverlay: syncprov
    olcSpNoPresent: TRUE
    olcSpReloadHint: TRUE

    # syncrepl Provider for primary db
    dn: olcOverlay=syncprov,olcDatabase={1}hdb,cn=config
    changetype: add
    objectClass: olcOverlayConfig
    objectClass: olcSyncProvConfig
    olcOverlay: syncprov
    olcSpNoPresent: TRUE

    # accesslog overlay definitions for primary db
    dn: olcOverlay=accesslog,olcDatabase={1}hdb,cn=config
    objectClass: olcOverlayConfig
    objectClass: olcAccessLogConfig
    olcOverlay: accesslog
    olcAccessLogDB: cn=accesslog
    olcAccessLogOps: writes
    olcAccessLogSuccess: TRUE
    # scan the accesslog DB every day, and purge entries older than 7 days
    olcAccessLogPurge: 07+00:00 01+00:00

Change the rootDN in the LDIF file to match the one you have for your
directory.

The apparmor profile for slapd will not need to be adjusted for the accesslog
database location since `/etc/apparmor.d/local/usr.sbin.slapd` contains:

    /var/lib/ldap/ r,
    /var/lib/ldap/** rwk,

Create a directory, set up a databse config file, and reload the apparmor
profile:

    sudo -u openldap mkdir /var/lib/ldap/accesslog
    sudo -u openldap cp /var/lib/ldap/DB_CONFIG /var/lib/ldap/accesslog
    sudo systemctl reload apparmor.service

Add the new content and, due to the apparmor change, restart the daemon:

    sudo ldapadd -Q -Y EXTERNAL -H ldapi:/// -f provider_sync.ldif
    sudo systemctl restart slapd.service

The Provider is now configured.

### Consumer Configuration {#openldap-consumer-configuration}

And now configure the *Consumer*.

Install the software by going through [Installation]. Make sure the
slapd-config databse is identical to the Provider's. In particular, make sure
schemas and the databse suffix are the same.

Create an LDIF file with the following contents and name it
`consumer_sync.ldif`:

    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: syncprov

    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: entryUUID eq
    -
    add: olcSyncRepl
    olcSyncRepl: rid=0 provider=ldap://ldap01.example.com bindmethod=simple binddn="cn=admin,dc=example,dc=com" 
     credentials=secret searchbase="dc=example,dc=com" logbase="cn=accesslog" 
     logfilter="(&(objectClass=auditWriteObject)(reqResult=0))" schemachecking=on 
     type=refreshAndPersist retry="60 +" syncdata=accesslog
    -
    add: olcUpdateRef
    olcUpdateRef: ldap://ldap01.example.com

Ensure the following attributes have the correct values:

-   *provider* (Provider server's hostname -- ldap01.example.com in this
    example -- or IP address)

-   *binddn* (the admin DN you're using)

-   *credentials* (the admin DN password you're using)

-   *searchbase* (the database suffix you're using)

-   *olcUpdateRef* (Provider server's hostname or IP address)

-   *rid* (Replica ID, an unique 3-digit that identifies the replica. Each
    consumer should have at least one rid)

Add the new content:

    sudo ldapadd -Q -Y EXTERNAL -H ldapi:/// -f consumer_sync.ldif

You're done. The two databases (suffix: dc=example,dc=com) should now be
synchronizing.

### Testing {#openldap-testing}

Once replication starts, you can monitor it by running

    ldapsearch -z1 -LLLQY EXTERNAL -H ldapi:/// -s base -b dc=example,dc=com contextCSN

    dn: dc=example,dc=com
    contextCSN: 20120201193408.178454Z#000000#000#000000

on both the provider and the consumer. Once the output
(`20120201193408.178454Z#000000#000#000000` in the above example) for both
machines match, you have replication. Every time a change is done in the
provider, this value will change and so should the one in the consumer(s).

If your connection is slow and/or your ldap database large, it might take a
while for the consumer's *contextCSN* match the provider's. But, you will know
it is progressing since the consumer's *contextCSN* will be steadly
increasing.

If the consumer's *contextCSN* is missing or does not match the provider, you
should stop and figure out the issue before continuing. Try checking the slapd
(syslog) and the auth log files in the provider to see if the consumer's
authentication requests were successful or its requests to retrieve data (they
look like a lot of ldapsearch statements) return no errors.

To test if it worked simply query, on the Consumer, the DNs in the database:

    sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b dc=example,dc=com dn

You should see the user 'john' and the group 'miners' as well as the nodes
'People' and 'Groups'.

## Access Control {#openldap-server-acl}

The management of what type of access (read, write, etc) users should be
granted to resources is known as *access control*. The configuration
directives involved are called *access control lists* or ACL.

When we installed the slapd package various ACL were set up automatically. We
will look at a few important consequences of those defaults and, in so doing,
we'll get an idea of how ACLs work and how they're configured.

To get the effective ACL for an LDAP query we need to look at the ACL entries
of the database being queried as well as those of the special frontend
database instance. The ACLs belonging to the latter act as defaults in case
those of the former do not match. The frontend database is the second to be
consulted and the ACL to be applied is the first to match ("first match wins")
among these 2 ACL sources. The following commands will give, respectively, the
ACLs of the hdb database ("dc=example,dc=com") and those of the frontend
database:

    sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
    cn=config '(olcDatabase={1}hdb)' olcAccess

    dn: olcDatabase={1}hdb,cn=config
    olcAccess: {0}to attrs=userPassword,shadowLastChange by self write by anonymous
                  auth by dn="cn=admin,dc=example,dc=com" write by * none
    olcAccess: {1}to dn.base="" by * read
    olcAccess: {2}to * by self write by dn="cn=admin,dc=example,dc=com" write by *
      read

> **Note**
>
> The rootDN always has full rights to its database. Including it in an ACL
> does provide an explicit configuration but it also causes slapd to incur a
> performance penalty.

    sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
    cn=config '(olcDatabase={-1}frontend)' olcAccess

    dn: olcDatabase={-1}frontend,cn=config
    olcAccess: {0}to * by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,
                  cn=external,cn=auth manage by * break
    olcAccess: {1}to dn.exact="" by * read
    olcAccess: {2}to dn.base="cn=Subschema" by * read

The very first ACL is crucial:

    olcAccess: {0}to attrs=userPassword,shadowLastChange by self write by anonymous
                  auth by dn="cn=admin,dc=example,dc=com" write by * none

This can be represented differently for easier digestion:

    to attrs=userPassword
        by self write
        by anonymous auth
        by dn="cn=admin,dc=example,dc=com" write
        by * none

    to attrs=shadowLastChange
        by self write
        by anonymous auth
        by dn="cn=admin,dc=example,dc=com" write
        by * none

This compound ACL (there are 2) enforces the following:

-   Anonymous 'auth' access is provided to the *userPassword* attribute for
    the initial connection to occur. Perhaps counter-intuitively, 'by
    anonymous auth' is needed even when anonymous access to the DIT
    is unwanted. Once the remote end is connected, howerver, authentication
    can occur (see next point).

-   Authentication can happen because all users have 'read' (due to 'by
    self write') access to the *userPassword* attribute.

-   The *userPassword* attribute is otherwise unaccessible by all other users,
    with the exception of the rootDN, who has complete access to it.

-   In order for users to change their own password, using `passwd` or other
    utilities, the *shadowLastChange* attribute needs to be accessible once a
    user has authenticated.

This DIT can be searched anonymously because of 'by \* read' in this ACL:

    to *
        by self write
        by dn="cn=admin,dc=example,dc=com" write
        by * read

If this is unwanted then you need to change the ACLs. To force authentication
during a bind request you can alternatively (or in combination with the
modified ACL) use the 'olcRequire: authc' directive.

As previously mentioned, there is no administrative account created for the
slapd-config database. There is, however, a SASL identity that is granted full
access to it. It represents the localhost's superuser (root/sudo). Here it is:

    dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth 

The following command will display the ACLs of the slapd-config database:

    sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
    cn=config '(olcDatabase={0}config)' olcAccess

    dn: olcDatabase={0}config,cn=config
    olcAccess: {0}to * by dn.exact=gidNumber=0+uidNumber=0,cn=peercred,
                  cn=external,cn=auth manage by * break

Since this is a SASL identity we need to use a SASL *mechanism* when invoking
the LDAP utility in question and we have seen it plenty of times in this
guide. It is the EXTERNAL mechanism. See the previous command for an example.
Note that:

You must use *sudo* to become the root identity in order for the ACL to match.

The EXTERNAL mechanism works via *IPC* (UNIX domain sockets). This means you
must use the *ldapi* URI format.

A succinct way to get all the ACLs is like this:

    sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b \
    cn=config '(olcAccess=*)' olcAccess olcSuffix

There is much to say on the topic of access control. See the man page for
[slapd.access].

## TLS {#openldap-tls}

When authenticating to an OpenLDAP server it is best to do so using an
encrypted session. This can be accomplished using Transport Layer Security
(TLS).

Here, we will be our own *Certificate Authority* and then create and sign our
LDAP server certificate as that CA. Since slapd is compiled using the gnutls
library, we will use the certtool utility to complete these tasks.

Install the gnutls-bin and ssl-cert packages:

    sudo apt install gnutls-bin ssl-cert

Create a private key for the Certificate Authority:

    sudo sh -c "certtool --generate-privkey > /etc/ssl/private/cakey.pem"

Create the template/file `/etc/ssl/ca.info` to define the CA:

    cn = Example Company
    ca
    cert_signing_key

Create the self-signed CA certificate:

    sudo certtool --generate-self-signed \
    --load-privkey /etc/ssl/private/cakey.pem \ 
    --template /etc/ssl/ca.info \
    --outfile /etc/ssl/certs/cacert.pem

Make a private key for the server:

    sudo certtool --generate-privkey \
    --bits 1024 \
    --outfile /etc/ssl/private/ldap01_slapd_key.pem

> **Note**
>
> Replace *ldap01* in the filename with your server's hostname. Naming the
> certificate and key for the host and service that will be using them will
> help keep things clear.

Create the `/etc/ssl/ldap01.info` info file containing:

    organization = Example Company
    cn = ldap01.example.com
    tls_www_server
    encryption_key
    signing_key
    expiration_days = 3650

The above certificate is good for 10 years. Adjust accordingly.

Create the server's certificate:

    sudo certtool --generate-certificate \
    --load-privkey /etc/ssl/private/ldap01_slapd_key.pem \
    --load-ca-certificate /etc/ssl/certs/cacert.pem \
    --load-ca-privkey /etc/ssl/private/cakey.pem \
    --template /etc/ssl/ldap01.info \
    --outfile /etc/ssl/certs/ldap01_slapd_cert.pem

Create the file `certinfo.ldif` with the following contents (adjust
accordingly, our example assumes we created certs using
https://www.cacert.org):

    dn: cn=config
    add: olcTLSCACertificateFile
    olcTLSCACertificateFile: /etc/ssl/certs/cacert.pem
    -
    add: olcTLSCertificateFile
    olcTLSCertificateFile: /etc/ssl/certs/ldap01_slapd_cert.pem
    -
    add: olcTLSCertificateKeyFile
    olcTLSCertificateKeyFile: /etc/ssl/private/ldap01_slapd_key.pem

Use the ldapmodify command to tell slapd about our TLS work via the
slapd-config database:

    sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f /etc/ssl/certinfo.ldif

Contratry to popular belief, you do not need *ldaps://* in
`/etc/default/slapd` in order to use encryption. You should have just:

    SLAPD_SERVICES="ldap:/// ldapi:///"

> **Note**
>
> LDAP over TLS/SSL (ldaps://) is deprecated in favour of *StartTLS*. The
> latter refers to an existing LDAP session (listening on TCP port 389)
> becoming protected by TLS/SSL whereas LDAPS, like HTTPS, is a distinct
> encrypted-from-the-start protocol that operates over TCP port 636.

Tighten up ownership and permissions:

    sudo adduser openldap ssl-cert
    sudo chgrp ssl-cert /etc/ssl/private/ldap01_slapd_key.pem
    sudo chmod g+r /etc/ssl/private/ldap01_slapd_key.pem
    sudo chmod o-r /etc/ssl/private/ldap01_slapd_key.pem

Restart OpenLDAP:

    sudo systemctl restart slapd.service

Check your host's logs (/var/log/syslog) to see if the server has started
properly.

## Replication and TLS {#openldap-tls-replication}

If you have set up replication between servers, it is common practice to
encrypt (StartTLS) the replication traffic to prevent evesdropping. This is
distinct from using encryption with authentication as we did above. In this
section we will build on that TLS-authentication work.

The assumption here is that you have set up replication between Provider and
Consumer according to [Replication] and have configured TLS for authentication
on the Provider by following [TLS].

As previously stated, the objective (for us) with replication is high
availablity for the LDAP service. Since we have TLS for authentication on the
Provider we will require the same on the Consumer. In addition to this,
however, we want to encrypt replication traffic. What remains to be done is to
create a key and certificate for the Consumer and then configure accordingly.
We will generate the key/certificate on the Provider, to avoid having to
create another CA certificate, and then transfer the necessary material over
to the Consumer.

On the Provider,

Create a holding directory (which will be used for the eventual transfer) and
then the Consumer's private key:

    mkdir ldap02-ssl
    cd ldap02-ssl
    sudo certtool --generate-privkey \
    --bits 1024 \
    --outfile ldap02_slapd_key.pem

Create an info file, `ldap02.info`, for the Consumer server, adjusting its
values accordingly:

    organization = Example Company
    cn = ldap02.example.com
    tls_www_server
    encryption_key
    signing_key
    expiration_days = 3650

Create the Consumer's certificate:

    sudo certtool --generate-certificate \
    --load-privkey ldap02_slapd_key.pem \
    --load-ca-certificate /etc/ssl/certs/cacert.pem \
    --load-ca-privkey /etc/ssl/private/cakey.pem \
    --template ldap02.info \
    --outfile ldap02_slapd_cert.pem

Get a copy of the CA certificate:

    cp /etc/ssl/certs/cacert.pem .

We're done. Now transfer the `ldap02-ssl` directory to the Consumer. Here we
use scp (adjust accordingly):

    cd ..
    scp -r ldap02-ssl user@consumer:

On the Consumer,

Configure TLS authentication:

    sudo apt install ssl-cert
    sudo adduser openldap ssl-cert
    sudo cp ldap02_slapd_cert.pem cacert.pem /etc/ssl/certs
    sudo cp ldap02_slapd_key.pem /etc/ssl/private
    sudo chgrp ssl-cert /etc/ssl/private/ldap02_slapd_key.pem
    sudo chmod g+r /etc/ssl/private/ldap02_slapd_key.pem
    sudo chmod o-r /etc/ssl/private/ldap02_slapd_key.pem

Create the file `/etc/ssl/certinfo.ldif` with the following contents (adjust
accordingly):

    dn: cn=config
    add: olcTLSCACertificateFile
    olcTLSCACertificateFile: /etc/ssl/certs/cacert.pem
    -
    add: olcTLSCertificateFile
    olcTLSCertificateFile: /etc/ssl/certs/ldap02_slapd_cert.pem
    -
    add: olcTLSCertificateKeyFile
    olcTLSCertificateKeyFile: /etc/ssl/private/ldap02_slapd_key.pem

Configure the slapd-config database:

    sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f certinfo.ldif

Configure `/etc/default/slapd` as on the Provider (SLAPD\_SERVICES).

On the Consumer,

Configure TLS for Consumer-side replication. Modify the existing *olcSyncrepl*
attribute by tacking on some TLS options. In so doing, we will see, for the
first time, how to change an attribute's value(s).

Create the file `consumer_sync_tls.ldif` with the following contents:

    dn: olcDatabase={1}hdb,cn=config
    replace: olcSyncRepl
    olcSyncRepl: rid=0 provider=ldap://ldap01.example.com bindmethod=simple
     binddn="cn=admin,dc=example,dc=com" credentials=secret searchbase="dc=example,dc=com"
     logbase="cn=accesslog" logfilter="(&(objectClass=auditWriteObject)(reqResult=0))"
     schemachecking=on type=refreshAndPersist retry="60 +" syncdata=accesslog
     starttls=critical tls_reqcert=demand

The extra options specify, respectively, that the consumer must use StartTLS
and that the CA certificate is required to verify the Provider's identity.
Also note the LDIF syntax for changing the values of an attribute ('replace').

Implement these changes:

    sudo ldapmodify -Y EXTERNAL -H ldapi:/// -f consumer_sync_tls.ldif

And restart slapd:

    sudo systemctl restart slapd.service

On the Provider,

Check to see that a TLS session has been established. In `/var/log/syslog`,
providing you have 'conns'-level logging set up, you should see messages
similar to:

    slapd[3620]: conn=1047 fd=20 ACCEPT from IP=10.153.107.229:57922 (IP=0.0.0.0:389)
    slapd[3620]: conn=1047 op=0 EXT oid=1.3.6.1.4.1.1466.20037
    slapd[3620]: conn=1047 op=0 STARTTLS
    slapd[3620]: conn=1047 op=0 RESULT oid= err=0 text=
    slapd[3620]: conn=1047 fd=20 TLS established tls_ssf=128 ssf=128
    slapd[3620]: conn=1047 op=1 BIND dn="cn=admin,dc=example,dc=com" method=128
    slapd[3620]: conn=1047 op=1 BIND dn="cn=admin,dc=example,dc=com" mech=SIMPLE ssf=0
    slapd[3620]: conn=1047 op=1 RESULT tag=97 err=0 text

## LDAP Authentication {#openldap-auth-config}

Once you have a working LDAP server, you will need to install libraries on the
client that will know how and when to contact it. On Ubuntu, this has been
traditionally accomplished by installing the libnss-ldap package. This package
will bring in other tools that will assist you in the configuration step.
Install this package now:

    sudo apt install libnss-ldap

You will be prompted for details of your LDAP server. If you make a mistake
you can try again using:

    sudo dpkg-reconfigure ldap-auth-config

The results of the dialog can be seen in `/etc/ldap.conf`. If your server
requires options not covered in the menu edit this file accordingly.

Now configure the LDAP profile for NSS:

    sudo auth-client-config -t nss -p lac_ldap

Configure the system to use LDAP for authentication:

    sudo pam-auth-update

From the menu, choose LDAP and any other authentication mechanisms you need.

You should now be able to log in using LDAP-based credentials.

LDAP clients will need to refer to multiple servers if replication is in use.
In `/etc/ldap.conf` you would have something like:

    uri ldap://ldap01.example.com ldap://ldap02.example.com

The request will time out and the Consumer (ldap02) will attempt to be reached
if the Provider (ldap01) becomes unresponsive.

If you are going to use LDAP to store Samba users you will need to configure
the Samba server to authenticate using LDAP. See [Samba and LDAP] for details.

> **Note**
>
> An alternative to the libnss-ldap package is the libnss-ldapd package. This,
> however, will bring in the nscd package which is problably not wanted.
> Simply remove it afterwards.

## User and Group Management {#ldap-usergroup-management}

The ldap-utils package comes with enough utilities to manage the directory but
the long string of options needed can make them a burden to use. The
ldapscripts package contains wrapper scripts to these utilities that some
people find easier to use.

Install the package:

    sudo apt install ldapscripts

Then edit the file `/etc/ldapscripts/ldapscripts.conf` to arrive at something
similar to the following:

    SERVER=localhost
    BINDDN='cn=admin,dc=example,dc=com'
    BINDPWDFILE="/etc/ldapscripts/ldapscripts.passwd"
    SUFFIX='dc=example,dc=com'
    GSUFFIX='ou=Groups'
    USUFFIX='ou=People'
    MSUFFIX='ou=Computers'
    GIDSTART=10000
    UIDSTART=10000
    MIDSTART=10000

Now, create the `ldapscripts.passwd` file to allow rootDN access to the
directory:

    sudo sh -c "echo -n 'secret' > /etc/ldapscripts/ldapscripts.passwd"
    sudo chmod 400 /etc/ldapscripts/ldapscripts.passwd

> **Note**
>
> Replace “secret” with the actual password for your database's rootDN user.

The scripts are now ready to help manage your directory. Here are some
examples of how to use them:

-   Create a new user:

        sudo ldapadduser george example

    This will create a user with uid *george* and set the user's primary
    group (gid) to *example*

-   Change a user's password:

        sudo ldapsetpasswd george
        Changing password for user uid=george,ou=People,dc=example,dc=com
        New Password: 
        New Password (verify): 

-   Delete a user:

        sudo ldapdeleteuser george

-   Add a group:

        sudo ldapaddgroup qa

-   Delete a group:

        sudo ldapdeletegroup qa

-   Add a user to a group:

        sudo ldapaddusertogroup george qa

    You should now see a *memberUid* attribute for the *qa* group with a value
    of *george*.

-   Remove a user from a group:

        sudo ldapdeleteuserfromgroup george qa

    The *memberUid* attribute should now be removed from the *qa* group.

-   The ldapmodifyuser script allows you to add, remove, or replace a user's
    attributes. The script uses the same syntax as the ldapmodify utility. For
    example:

        sudo ldapmodifyuser george
        # About to modify the following entry :
        dn: uid=george,ou=People,dc=example,dc=com
        objectClass: account
        objectClass: posixAccount
        cn: george
        uid: george
        uidNumber: 1001
        gidNumber: 1001
        homeDirectory: /home/george
        loginShell: /bin/bash
        gecos: george
        description: User account
        userPassword:: e1NTSEF9eXFsTFcyWlhwWkF1eGUybVdFWHZKRzJVMjFTSG9vcHk=

        # Enter your modifications here, end with CTRL-D.
        dn: uid=george,ou=People,dc=example,dc=com
        replace: gecos
        gecos: George Carlin

    The user's *gecos* should now be “George Carlin”.

-   A nice feature of ldapscripts is the template system. Templates allow you
    to customize the attributes of user, group, and machine objects. For
    example, to enable the *user* template edit
    `/etc/ldapscripts/ldapscripts.conf` changing:

        UTEMPLATE="/etc/ldapscripts/ldapadduser.template"

    There are *sample* templates in the `/usr/share/doc/ldapscripts/examples`
    directory. Copy or rename the `ldapadduser.template.sample` file to
    `/etc/ldapscripts/ldapadduser.template`:

        sudo cp /usr/share/doc/ldapscripts/examples/ldapadduser.template.sample \
        /etc/ldapscripts/ldapadduser.template

    Edit the new template to add the desired attributes. The following will
    create new users with an objectClass of inetOrgPerson:

        dn: uid=<user>,<usuffix>,<suffix>
        objectClass: inetOrgPerson
        objectClass: posixAccount
        cn: <user>
        sn: <ask>
        uid: <user>
        uidNumber: <uid>
        gidNumber: <gid>
        homeDirectory: <home>
        loginShell: <shell>
        gecos: <user>
        description: User account
        title: Employee

    Notice the *&lt;ask&gt;* option used for the *sn* attribute. This will
    make ldapadduser prompt you for its value.

There are utilities in the package that were not covered here. Here is a
complete list:

    ldaprenamemachine
    ldapadduser
    ldapdeleteuserfromgroup
    ldapfinger
    ldapid
    ldapgid
    ldapmodifyuser
    ldaprenameuser
    lsldap
    ldapaddusertogroup
    ldapsetpasswd
    ldapinit
    ldapaddgroup
    ldapdeletegroup
    ldapmodifygroup
    ldapdeletemachine
    ldaprenamegroup
    ldapaddmachine
    ldapmodifymachine
    ldapsetprimarygroup
    ldapdeleteuser

## Backup and Restore {#ldap-backup}

Now we have ldap running just the way we want, it is time to ensure we can
save all of our work and restore it as needed.

What we need is a way to backup the ldap database(s), specifically the backend
(cn=config) and frontend (dc=example,dc=com). If we are going to backup those
databases into, say, `/export/backup`, we could use slapcat as shown in the
following script, called `/usr/local/bin/ldapbackup`:

    #!/bin/bash

    BACKUP_PATH=/export/backup
    SLAPCAT=/usr/sbin/slapcat

    nice ${SLAPCAT} -n 0 > ${BACKUP_PATH}/config.ldif
    nice ${SLAPCAT} -n 1 > ${BACKUP_PATH}/example.com.ldif
    nice ${SLAPCAT} -n 2 > ${BACKUP_PATH}/access.ldif
    chmod 640 ${BACKUP_PATH}/*.ldif

> **Note**
>
> These files are uncompressed text files containing everything in your ldap
> databases including the tree layout, usernames, and every password. So, you
> might want to consider making `/export/backup` an encrypted partition and
> even having the script encrypt those files as it creates them. Ideally you
> should do both, but that depends on your security requirements.

Then, it is just a matter of having a cron script to run this program as often
as we feel comfortable with. For many, once a day suffices. For others, more
often is required. Here is an example of a cron script called
`/etc/cron.d/ldapbackup` that is run every night at 22:45h:

    MAILTO=backup-emails@domain.com
    45 22 * * *  root    /usr/local/bin/ldapbackup

Now the files are created, they should be copied to a backup server.

Assuming we did a fresh reinstall of ldap, the restore process could be
something like this:

    sudo systemctl stop slapd.service
    sudo mkdir /var/lib/ldap/accesslog
    sudo slapadd -F /etc/ldap/slapd.d -n 0 -l /export/backup/config.ldif
    sudo slapadd -F /etc/ldap/slapd.d -n 1 -l /export/backup/domain.com.ldif
    sudo slapadd -F /etc/ldap/slapd.d -n 2 -l /export/backup/access.ldif
    sudo chown -R openldap:openldap /etc/ldap/slapd.d/
    sudo chown -R openldap:openldap /var/lib/ldap/
    sudo systemctl start slapd.service

## Resources {#openldap-server-resources}

-   The primary resource is the upstream documentation: [www.openldap.org]

-   There are many man pages that come with the slapd package. Here are some
    important ones, especially considering the material presented in this
    guide:

        slapd
        slapd-config
        slapd.access
        slapo-syncprov

-   Other man pages:

        auth-client-config
        pam-auth-update

-   Zytrax's [LDAP for Rocket Scientists]; a less pedantic but comprehensive
    treatment of LDAP

-   A Ubuntu community [OpenLDAP wiki] page has a collection of notes

-   O'Reilly's [LDAP System Administration] (textbook; 2003)

-   Packt's [Mastering OpenLDAP] (textbook; 2007)

# Samba and LDAP {#samba-ldap}

This section covers the integration of Samba with LDAP. The Samba server's
role will be that of a "standalone" server and the LDAP directory will provide
the authentication layer in addition to containing the user, group, and
machine account information that Samba requires in order to function (in any
of its 3 possible roles). The pre-requisite is an OpenLDAP server configured
with a directory that can accept authentication requests. See
[OpenLDAP Server] for details on fulfilling this requirement. Once this
section is completed, you will need to decide what specifically you want Samba
to do for you and then configure it accordingly.

## Software Installation {#samba-ldap-installation}

There are two packages needed when integrating Samba with LDAP: samba and
smbldap-tools.

Strictly speaking, the smbldap-tools package isn't needed, but unless you have
some other way to manage the various Samba entities (users, groups, computers)
in an LDAP context then you should install it.

Install these packages now:

    sudo apt install samba smbldap-tools

## LDAP Configuration {#samba-ldap-openldap-configuration}

We will now configure the LDAP server so that it can accomodate Samba data. We
will perform three tasks in this section:

Import a schema

Index some entries

Add objects

### Samba schema {#samba-ldap-openldap-configuration-samba-schema}

In order for OpenLDAP to be used as a backend for Samba, logically, the DIT
will need to use attributes that can properly describe Samba data. Such
attributes can be obtained by introducing a Samba LDAP schema. Let's do this
now.

> **Note**
>
> For more information on schemas and their installation see
> [Modifying the slapd Configuration Database].

The schema is found in the now-installed samba package. It needs to be
unzipped and copied to the `/etc/ldap/schema` directory:

    sudo cp /usr/share/doc/samba/examples/LDAP/samba.schema.gz /etc/ldap/schema
    sudo gzip -d /etc/ldap/schema/samba.schema.gz

Have the configuration file `schema_convert.conf` that contains the following
lines:

    include /etc/ldap/schema/core.schema
    include /etc/ldap/schema/collective.schema
    include /etc/ldap/schema/corba.schema
    include /etc/ldap/schema/cosine.schema
    include /etc/ldap/schema/duaconf.schema
    include /etc/ldap/schema/dyngroup.schema
    include /etc/ldap/schema/inetorgperson.schema
    include /etc/ldap/schema/java.schema
    include /etc/ldap/schema/misc.schema
    include /etc/ldap/schema/nis.schema
    include /etc/ldap/schema/openldap.schema
    include /etc/ldap/schema/ppolicy.schema
    include /etc/ldap/schema/ldapns.schema
    include /etc/ldap/schema/pmi.schema
    include /etc/ldap/schema/samba.schema

Have the directory `ldif_output` hold output.

Determine the index of the schema:

    slapcat -f schema_convert.conf -F ldif_output -n 0 | grep samba,cn=schema

    dn: cn={14}samba,cn=schema,cn=config

Convert the schema to LDIF format:

    slapcat -f schema_convert.conf -F ldif_output -n0 -H \
    ldap:///cn={14}samba,cn=schema,cn=config -l cn=samba.ldif

Edit the generated `cn=samba.ldif` file by removing index information to
arrive at:

    dn: cn=samba,cn=schema,cn=config
    ...
    cn: samba

Remove the bottom lines:

    structuralObjectClass: olcSchemaConfig
    entryUUID: b53b75ca-083f-102d-9fff-2f64fd123c95
    creatorsName: cn=config
    createTimestamp: 20080827045234Z
    entryCSN: 20080827045234.341425Z#000000#000#000000
    modifiersName: cn=config
    modifyTimestamp: 20080827045234Z

Your attribute values will vary.

Add the new schema:

    sudo ldapadd -Q -Y EXTERNAL -H ldapi:/// -f cn\=samba.ldif

To query and view this new schema:

    sudo ldapsearch -Q -LLL -Y EXTERNAL -H ldapi:/// -b cn=schema,cn=config 'cn=*samba*'

### Samba indices {#samba-ldap-openldap-configuration-samba-indices}

Now that slapd knows about the Samba attributes, we can set up some indices
based on them. Indexing entries is a way to improve performance when a client
performs a filtered search on the DIT.

Create the file `samba_indices.ldif` with the following contents:

    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: uidNumber eq
    olcDbIndex: gidNumber eq
    olcDbIndex: loginShell eq
    olcDbIndex: uid eq,pres,sub
    olcDbIndex: memberUid eq,pres,sub
    olcDbIndex: uniqueMember eq,pres
    olcDbIndex: sambaSID eq
    olcDbIndex: sambaPrimaryGroupSID eq
    olcDbIndex: sambaGroupType eq
    olcDbIndex: sambaSIDList eq
    olcDbIndex: sambaDomainName eq
    olcDbIndex: default sub

Using the ldapmodify utility load the new indices:

    sudo ldapmodify -Q -Y EXTERNAL -H ldapi:/// -f samba_indices.ldif

If all went well you should see the new indices using ldapsearch:

    sudo ldapsearch -Q -LLL -Y EXTERNAL -H \
    ldapi:/// -b cn=config olcDatabase={1}hdb olcDbIndex

### Adding Samba LDAP objects {#samba-ldap-openldap-configuration-populating}

Next, configure the smbldap-tools package to match your environment. The
package comes with a configuration helper script, smbldap-config.pl, that will
ask questions.

The smbldap-populate script will then add the LDAP objects required for Samba.
It is a good idea to first make a backup of your DIT using slapcat:

    sudo slapcat -l backup.ldif

Once you have a backup proceed to populate your directory:

    sudo smbldap-populate

You can create a LDIF file containing the new Samba objects by executing
`sudo smbldap-populate -e samba.ldif`. This allows you to look over the
changes making sure everything is correct. If it is, rerun the script without
the '-e' switch. Alternatively, you can take the LDIF file and import its data
per usual.

Your LDAP directory now has the necessary information to authenticate Samba
users.

## Samba Configuration {#samba-ldap-samba-configuration}

There are multiple ways to configure Samba. For details on some common
configurations see [???]. To configure Samba to use LDAP, edit its
configuration file `/etc/samba/smb.conf` commenting out the default *passdb
backend* parameter and adding some ldap-related ones:

    #   passdb backend = tdbsam

    # LDAP Settings
       passdb backend = ldapsam:ldap://hostname
       ldap suffix = dc=example,dc=com
       ldap user suffix = ou=People
       ldap group suffix = ou=Groups
       ldap machine suffix = ou=Computers
       ldap idmap suffix = ou=Idmap
       ldap admin dn = cn=admin,dc=example,dc=com
       ldap ssl = start tls
       ldap passwd sync = yes
    ...
       add machine script = sudo /usr/sbin/smbldap-useradd -t 0 -w "%u"

Change the values to match your environment.

Restart samba to enable the new settings:

    sudo systemctl restart smbd.service nmbd.service

Now inform Samba about the rootDN user's password (the one set during the
installation of the slapd package):

    sudo smbpasswd -w password

If you have existing LDAP users that you want to include in your new
LDAP-backed Samba they will, of course, also need to be given some of the
extra attributes. The smbpasswd utility can do this as well (your host will
need to be able to see (enumerate) those users via NSS; install and configure
either libnss-ldapd or libnss-ldap):

    sudo smbpasswd -a username

You will prompted to enter a password. It will be considered as the new
password for that user. Making it the same as before is reasonable.

To manage user, group, and machine accounts use the utilities provided by the
smbldap-tools package. Here are some examples:

-   To add a new user:

        sudo smbldap-useradd -a -P username

    The *-a* option adds the Samba attributes, and the *-P* option calls the
    smbldap-passwd utility after the user is created allowing you to enter a
    password for the user.

-   To remove a user:

        sudo smbldap-userdel username

    In the above command, use the *-r* option to remove the user's
    home directory.

-   To add a group:

        sudo smbldap-groupadd -a groupname

    As for smbldap-useradd, the *-a* adds the Samba attributes.

-   To make an existing user a member of a group:

        sudo smbldap-groupmod -m username groupname

    The *-m* option can add more than one user at a time by listing them in
    comma-separated format.

-   To remove a user from a group:

        sudo smbldap-groupmod -x username groupname

-   To add a Samba machine account:

        sudo smbldap-useradd -t 0 -w username

    Replace *username* with the name of the workstation. The *-t 0* option
    creates the machine account without a delay, while the *-w* option
    specifies the user as a machine account. Also, note the *add machine
    script* parameter in `/etc/samba/smb.conf` was changed to
    use smbldap-useradd.

There are utilities in the smbldap-tools package that were not covered here.
Here is a complete list:

    smbldap-groupadd
    smbldap-groupdel
    smbldap-groupmod
    smbldap-groupshow
    smbldap-passwd
    smbldap-populate
    smbldap-useradd
    smbldap-userdel
    smbldap-userinfo
    smbldap-userlist
    smbldap-usermod
    smbldap-usershow

## Resources {#samba-ldap-resources}

-   For more information on installing and configuring Samba see [???] of this
    Ubuntu Server Guide.

-   There are multiple places where LDAP and Samba is documented in the
    upstream [Samba HOWTO Collection].

-   Regarding the above, see specifically the [passdb section].

-   Although dated (2007), the [Linux Samba-OpenLDAP HOWTO] contains
    valuable notes.

-   The main page of the [Samba Ubuntu community documentation] has a plethora
    of links to articles that may prove useful.

# Kerberos

Kerberos is a network authentication system based on the principal of a
trusted third party. The other two parties being the user and the service the
user wishes to authenticate to. Not all services and applications can use
Kerberos, but for those that can, it brings the network environment one step
closer to being Single Sign On (SSO).

This section covers installation and configuration of a Kerberos server, and
some example client configurations.

## Overview {#kerberos-overview}

If you are new to Kerberos there are a few terms that are good to understand
before setting up a Kerberos server. Most of the terms will relate to things
you may be familiar with in other environments:

-   *Principal:* any users, computers, and services provided by servers need
    to be defined as Kerberos Principals.

-   *Instances:* are used for service principals and special
    administrative principals.

-   *Realms:* the unique realm of control provided by the Kerberos
    installation. Think of it as the domain or group your hosts and users
    belong to. Convention dictates the realm should be in uppercase. By
    default, ubuntu will use the DNS domain converted to
    uppercase (EXAMPLE.COM) as the realm.

-   *Key Distribution Center:* (KDC) consist of three parts, a database of all
    principals, the authentication server, and the ticket granting server. For
    each realm there must be at least one KDC.

-   *Ticket Granting Ticket:* issued by the Authentication Server (AS), the
    Ticket Granting Ticket (TGT) is encrypted in the user's password which is
    known only to the user and the KDC.

-   *Ticket Granting Server:* (TGS) issues service tickets to clients
    upon request.

-   *Tickets:* confirm the identity of the two principals. One principal being
    a user and the other a service requested by the user. Tickets establish an
    encryption key used for secure communication during the
    authenticated session.

-   *Keytab Files:* are files extracted from the KDC principal database and
    contain the encryption key for a service or host.

To put the pieces together, a Realm has at least one KDC, preferably more for
redundancy, which contains a database of Principals. When a user principal
logs into a workstation that is configured for Kerberos authentication, the
KDC issues a Ticket Granting Ticket (TGT). If the user supplied credentials
match, the user is authenticated and can then request tickets for Kerberized
services from the Ticket Granting Server (TGS). The service tickets allow the
user to authenticate to the service without entering another username and
password.

## Kerberos Server

### Installation {#kerberos-server-installation}

For this discussion, we will create a MIT Kerberos domain with the following
features (edit them to fit your needs):

-   *Realm:* EXAMPLE.COM

-   *Primary KDC:* kdc01.example.com (192.168.0.1)

-   *Secondary KDC:* kdc02.example.com (192.168.0.2)

-   *User principal:* steve

-   *Admin principal:* steve/admin

> **Note**
>
> It is *strongly* recommended that your network-authenticated users have
> their uid in a different range (say, starting at 5000) than that of your
> local users.

Before installing the Kerberos server a properly configured DNS server is
needed for your domain. Since the Kerberos Realm by convention matches the
domain name, this section uses the *EXAMPLE.COM* domain configured in [???][1]
of the DNS documentation.

Also, Kerberos is a time sensitive protocol. So if the local system time
between a client machine and the server differs by more than five minutes (by
default), the workstation will not be able to authenticate. To correct the
problem all hosts should have their time synchronized using the same *Network
Time Protocol (NTP)* server. For details on setting up NTP see [???][2].

The first step in creating a Kerberos Realm is to install the krb5-kdc and
krb5-admin-server packages. From a terminal enter:

    sudo apt install krb5-kdc krb5-admin-server

You will be asked at the end of the install to supply the hostname for the
Kerberos and Admin servers, which may or may not be the same server, for the
realm.

> **Note**
>
> By default the realm is created from the KDC's domain name.

Next, create the new realm with the kdb5\_newrealm utility:

    sudo krb5_newrealm

### Configuration {#kerberos-server-configuration}

The questions asked during installation are used to configure the
`/etc/krb5.conf` file. If you need to adjust the Key Distribution Center (KDC)
settings simply edit the file and restart the krb5-kdc daemon. If you need to
reconfigure Kerberos from scratch, perhaps to change the realm name, you can
do so by typing

    sudo dpkg-reconfigure krb5-kdc

Once the KDC is properly running, an admin user -- the *admin principal* -- is
needed. It is recommended to use a different username from your everyday
username. Using the kadmin.local utility in a terminal prompt enter:

    sudo kadmin.local
    Authenticating as principal root/admin@EXAMPLE.COM with password.
    kadmin.local: addprinc steve/admin
    WARNING: no policy specified for steve/admin@EXAMPLE.COM; defaulting to no policy
    Enter password for principal "steve/admin@EXAMPLE.COM": 
    Re-enter password for principal "steve/admin@EXAMPLE.COM": 
    Principal "steve/admin@EXAMPLE.COM" created.
    kadmin.local: quit

In the above example *steve* is the *Principal*, */admin* is an *Instance*,
and *@EXAMPLE.COM* signifies the realm. The *"every day"* Principal, a.k.a.
the *user principal*, would be *steve@EXAMPLE.COM*, and should have only
normal user rights.

> **Note**
>
> Replace *EXAMPLE.COM* and *steve* with your Realm and admin username.

Next, the new admin user needs to have the appropriate Access Control List
(ACL) permissions. The permissions are configured in the
`/etc/krb5kdc/kadm5.acl` file:

    steve/admin@EXAMPLE.COM        *

This entry grants *steve/admin* the ability to perform any operation on all
principals in the realm. You can configure principals with more restrictive
privileges, which is convenient if you need an admin principal that junior
staff can use in Kerberos clients. Please see the *kadm5.acl* man page for
details.

Now restart the krb5-admin-server for the new ACL to take affect:

    sudo systemctl restart krb5-admin-server.service

The new user principal can be tested using the kinit utility:

    kinit steve/admin
    steve/admin@EXAMPLE.COM's Password:

After entering the password, use the klist utility to view information about
the Ticket Granting Ticket (TGT):

    klist
    Credentials cache: FILE:/tmp/krb5cc_1000
            Principal: steve/admin@EXAMPLE.COM

      Issued           Expires          Principal
    Jul 13 17:53:34  Jul 14 03:53:34  krbtgt/EXAMPLE.COM@EXAMPLE.COM

Where the cache filename `krb5cc_1000` is composed of the prefix `krb5cc_` and
the user id (uid), which in this case is `1000`. You may need to add an entry
into the `/etc/hosts` for the KDC so the client can find the KDC. For example:

    192.168.0.1   kdc01.example.com       kdc01

Replacing *192.168.0.1* with the IP address of your KDC. This usually happens
when you have a Kerberos realm encompassing different networks separated by
routers.

The best way to allow clients to automatically determine the KDC for the Realm
is using DNS SRV records. Add the following to `/etc/named/db.example.com`:

    _kerberos._udp.EXAMPLE.COM.     IN SRV 1  0 88  kdc01.example.com.
    _kerberos._tcp.EXAMPLE.COM.     IN SRV 1  0 88  kdc01.example.com.
    _kerberos._udp.EXAMPLE.COM.     IN SRV 10 0 88  kdc02.example.com. 
    _kerberos._tcp.EXAMPLE.COM.     IN SRV 10 0 88  kdc02.example.com. 
    _kerberos-adm._tcp.EXAMPLE.COM. IN SRV 1  0 749 kdc01.example.com.
    _kpasswd._udp.EXAMPLE.COM.      IN SRV 1  0 464 kdc01.example.com.

> **Note**
>
> Replace *EXAMPLE.COM*, *kdc01*, and *kdc02* with your domain name, primary
> KDC, and secondary KDC.

See [???][3] for detailed instructions on setting up DNS.

Your new Kerberos Realm is now ready to authenticate clients.

## Secondary KDC {#kerberos-secondary-kdc}

Once you have one Key Distribution Center (KDC) on your network, it is good
practice to have a Secondary KDC in case the primary becomes unavailable.
Also, if you have Kerberos clients that are in different networks (possibly
separated by routers using NAT), it is wise to place a secondary KDC in each
of those networks.

First, install the packages, and when asked for the Kerberos and Admin server
names enter the name of the Primary KDC:

    sudo apt install krb5-kdc krb5-admin-server

Once you have the packages installed, create the Secondary KDC's host
principal. From a terminal prompt, enter:

    kadmin -q "addprinc -randkey host/kdc02.example.com"

> **Note**
>
> After, issuing any kadmin commands you will be prompted for your
> *username/admin@EXAMPLE.COM* principal password.

Extract the *keytab* file:

    kadmin -q "ktadd -norandkey -k keytab.kdc02 host/kdc02.example.com"

There should now be a `keytab.kdc02` in the current directory, move the file
to `/etc/krb5.keytab`:

    sudo mv keytab.kdc02 /etc/krb5.keytab

> **Note**
>
> If the path to the `keytab.kdc02` file is different adjust accordingly.

Also, you can list the principals in a Keytab file, which can be useful when
troubleshooting, using the klist utility:

    sudo klist -k /etc/krb5.keytab

The -k option indicates the file is a keytab file.

Next, there needs to be a `kpropd.acl` file on each KDC that lists all KDCs
for the Realm. For example, on both primary and secondary KDC, create
`/etc/krb5kdc/kpropd.acl`:

    host/kdc01.example.com@EXAMPLE.COM
    host/kdc02.example.com@EXAMPLE.COM

Create an empty database on the *Secondary KDC*:

    sudo kdb5_util -s create

Now start the kpropd daemon, which listens for connections from the kprop
utility. kprop is used to transfer dump files:

    sudo kpropd -S

From a terminal on the *Primary KDC*, create a dump file of the principal
database:

    sudo kdb5_util dump /var/lib/krb5kdc/dump

Extract the Primary KDC's *keytab* file and copy it to `/etc/krb5.keytab`:

    kadmin -q "ktadd -k keytab.kdc01 host/kdc01.example.com"
    sudo mv keytab.kdc01 /etc/krb5.keytab

> **Note**
>
> Make sure there is a *host* for *kdc01.example.com* before extracting the
> Keytab.

Using the kprop utility push the database to the Secondary KDC:

    sudo kprop -r EXAMPLE.COM -f /var/lib/krb5kdc/dump kdc02.example.com

> **Note**
>
> There should be a *SUCCEEDED* message if the propagation worked. If there is
> an error message check `/var/log/syslog` on the secondary KDC for more
> information.

You may also want to create a cron job to periodically update the database on
the Secondary KDC. For example, the following will push the database every
hour (note the long line has been split to fit the format of this document):

    # m h  dom mon dow   command
    0 * * * * /usr/sbin/kdb5_util dump /var/lib/krb5kdc/dump && 
    /usr/sbin/kprop -r EXAMPLE.COM -f /var/lib/krb5kdc/dump kdc02.example.com

Back on the *Secondary KDC*, create a *stash* file to hold the Kerberos master
key:

    sudo kdb5_util stash

Finally, start the krb5-kdc daemon on the Secondary KDC:

    sudo systemctl start krb5-kdc.service

The *Secondary KDC* should now be able to issue tickets for the Realm. You can
test this by stopping the krb5-kdc daemon on the Primary KDC, then by using
kinit to request a ticket. If all goes well you should receive a ticket from
the Secondary KDC. Otherwise, check `/var/log/syslog` and `/var/log/auth.log`
in the Secondary KDC.

## Kerberos Linux Client

This section covers configuring a Linux system as a Kerberos client. This will
allow access to any kerberized services once a user has successfully logged
into the system.

### Installation {#kerberos-client-installation}

In order to authenticate to a Kerberos Realm, the krb5-user and libpam-krb5
packages are needed, along with a few others that are not strictly necessary
but make life easier. To install the packages enter the following in a
terminal prompt:

    sudo apt install krb5-user libpam-krb5 libpam-ccreds auth-client-config

The auth-client-config package allows simple configuration of PAM for
authentication from multiple sources, and the libpam-ccreds will cache
authentication credentials allowing you to login in case the Key Distribution
Center (KDC) is unavailable. This package is also useful for laptops that may
authenticate using Kerberos while on the corporate network, but will need to
be accessed off the network as well.

### Configuration {#kerberos-client-configuration}

To configure the client in a terminal enter:

    sudo dpkg-reconfigure krb5-config

You will then be prompted to enter the name of the Kerberos Realm. Also, if
you don't have DNS configured with Kerberos *SRV* records, the menu will
prompt you for the hostname of the Key Distribution Center (KDC) and Realm
Administration server.

The dpkg-reconfigure adds entries to the `/etc/krb5.conf` file for your Realm.
You should have entries similar to the following:

    [libdefaults]
            default_realm = EXAMPLE.COM
    ...
    [realms]
            EXAMPLE.COM = {
                    kdc = 192.168.0.1
                    admin_server = 192.168.0.1
            }

> **Note**
>
> If you set the uid of each of your network-authenticated users to start at
> 5000, as suggested in [Installation][4], you can then tell pam to only try
> to authenticate using Kerberos users with uid &gt; 5000:
>
>     # Kerberos should only be applied to ldap/kerberos users, not local ones.
>     for i in common-auth common-session common-account common-password; do
>      sudo sed -i -r \ 
>      -e 's/pam_krb5.so minimum_uid=1000/pam_krb5.so minimum_uid=5000/' \ 
>      /etc/pam.d/$i 
>     done 
>
> This will avoid being asked for the (non-existent) Kerberos password of a
> locally authenticated user when changing its password using `passwd`.

You can test the configuration by requesting a ticket using the kinit utility.
For example:

    kinit steve@EXAMPLE.COM
    Password for steve@EXAMPLE.COM:

When a ticket has been granted, the details can be viewed using klist:

    klist
    Ticket cache: FILE:/tmp/krb5cc_1000
    Default principal: steve@EXAMPLE.COM

    Valid starting     Expires            Service principal
    07/24/08 05:18:56  07/24/08 15:18:56  krbtgt/EXAMPLE.COM@EXAMPLE.COM
            renew until 07/25/08 05:18:57


    Kerberos 4 ticket cache: /tmp/tkt1000
    klist: You have no tickets cached

Next, use the auth-client-config to configure the libpam-krb5 module to
request a ticket during login:

    sudo auth-client-config -a -p kerberos_example

You will should now receive a ticket upon successful login authentication.

## Resources {#kerberos-resources}

-   For more information on MIT's version of Kerberos, see the [MIT
    Kerberos] site.

-   The [Ubuntu Wiki Kerberos] page has more details.

-   O'Reilly's [Kerberos: The Definitive Guide] is a great reference when
    setting up Kerberos.

-   Also, feel free to stop by the *\#ubuntu-server* and *\#kerberos* IRC
    channels on [Freenode] if you have Kerberos questions.

# Kerberos and LDAP {#kerberos-ldap}

Most people will not use Kerberos by itself; once an user is authenticated
(Kerberos), we need to figure out what this user can do (authorization). And
that would be the job of programs such as LDAP.

Replicating a Kerberos principal database between two servers can be
complicated, and adds an additional user database to your network.
Fortunately, MIT Kerberos can be configured to use an LDAP directory as a
principal database. This section covers configuring a primary and secondary
kerberos server to use OpenLDAP for the principal database.

> **Note**
>
> The examples presented here assume MIT Kerberos and OpenLDAP.

## Configuring OpenLDAP {#kerberos-ldap-openldap}

First, the necessary *schema* needs to be loaded on an OpenLDAP server that
has network connectivity to the Primary and Secondary KDCs. The rest of this
section assumes that you also have LDAP replication configured between at
least two servers. For information on setting up OpenLDAP see
[OpenLDAP Server].

It is also required to configure OpenLDAP for TLS and SSL connections, so that
traffic between the KDC and LDAP server is encrypted. See [TLS] for details.

> **Note**
>
> `cn=admin,cn=config` is a user we created with rights to edit the ldap
> database. Many times it is the RootDN. Change its value to reflect your
> setup.

-   To load the schema into LDAP, on the LDAP server install the krb5-kdc-ldap
    package. From a terminal enter:

        sudo apt install krb5-kdc-ldap

-   Next, extract the `kerberos.schema.gz` file:

        sudo gzip -d /usr/share/doc/krb5-kdc-ldap/kerberos.schema.gz
        sudo cp /usr/share/doc/krb5-kdc-ldap/kerberos.schema /etc/ldap/schema/

-   The *kerberos* schema needs to be added to the *cn=config* tree. The
    procedure to add a new schema to slapd is also detailed in
    [Modifying the slapd Configuration Database].

    First, create a configuration file named `schema_convert.conf`, or a
    similar descriptive name, containing the following lines:

        include /etc/ldap/schema/core.schema
        include /etc/ldap/schema/collective.schema
        include /etc/ldap/schema/corba.schema
        include /etc/ldap/schema/cosine.schema
        include /etc/ldap/schema/duaconf.schema
        include /etc/ldap/schema/dyngroup.schema
        include /etc/ldap/schema/inetorgperson.schema
        include /etc/ldap/schema/java.schema
        include /etc/ldap/schema/misc.schema
        include /etc/ldap/schema/nis.schema
        include /etc/ldap/schema/openldap.schema
        include /etc/ldap/schema/ppolicy.schema
        include /etc/ldap/schema/kerberos.schema

    Create a temporary directory to hold the LDIF files:

        mkdir /tmp/ldif_output

    Now use slapcat to convert the schema files:

        slapcat -f schema_convert.conf -F /tmp/ldif_output -n0 -s \
        "cn={12}kerberos,cn=schema,cn=config" > /tmp/cn=kerberos.ldif

    Change the above file and path names to match your own if they
    are different.

    Edit the generated `/tmp/cn\=kerberos.ldif` file, changing the following
    attributes:

        dn: cn=kerberos,cn=schema,cn=config
        ...
        cn: kerberos

    And remove the following lines from the end of the file:

        structuralObjectClass: olcSchemaConfig
        entryUUID: 18ccd010-746b-102d-9fbe-3760cca765dc
        creatorsName: cn=config
        createTimestamp: 20090111203515Z
        entryCSN: 20090111203515.326445Z#000000#000#000000
        modifiersName: cn=config
        modifyTimestamp: 20090111203515Z

    The attribute values will vary, just be sure the attributes are removed.

    Load the new schema with ldapadd:

        ldapadd -x -D cn=admin,cn=config -W -f /tmp/cn\=kerberos.ldif

    Add an index for the *krb5principalname* attribute:

        ldapmodify -x -D cn=admin,cn=config -W
        Enter LDAP Password:
        dn: olcDatabase={1}hdb,cn=config
        add: olcDbIndex
        olcDbIndex: krbPrincipalName eq,pres,sub

        modifying entry "olcDatabase={1}hdb,cn=config"

    Finally, update the Access Control Lists (ACL):

        ldapmodify -x -D cn=admin,cn=config -W
        Enter LDAP Password: 
        dn: olcDatabase={1}hdb,cn=config
        replace: olcAccess
        olcAccess: to attrs=userPassword,shadowLastChange,krbPrincipalKey by
         dn="cn=admin,dc=example,dc=com" write by anonymous auth by self write by * none
        -
        add: olcAccess
        olcAccess: to dn.base="" by * read
        -
        add: olcAccess
        olcAccess: to * by dn="cn=admin,dc=example,dc=com" write by * read

        modifying entry "olcDatabase={1}hdb,cn=config"

That's it, your LDAP directory is now ready to serve as a Kerberos principal
database.

## Primary KDC Configuration {#kerberos-ldap-primary-kdc}

With OpenLDAP configured it is time to configure the KDC.

-   First, install the necessary packages, from a terminal enter:

        sudo apt install krb5-kdc krb5-admin-server krb5-kdc-ldap

-   Now edit `/etc/krb5.conf` adding the following options to under the
    appropriate sections:

        [libdefaults]
                default_realm = EXAMPLE.COM

        ...

        [realms]
                EXAMPLE.COM = {
                        kdc = kdc01.example.com
                        kdc = kdc02.example.com
                        admin_server = kdc01.example.com
                        admin_server = kdc02.example.com
                        default_domain = example.com
                        database_module = openldap_ldapconf
                }

        ...

        [domain_realm]
                .example.com = EXAMPLE.COM


        ...

        [dbdefaults]
                ldap_kerberos_container_dn = dc=example,dc=com

        [dbmodules]
                openldap_ldapconf = {
                        db_library = kldap
                        ldap_kdc_dn = "cn=admin,dc=example,dc=com"

                        # this object needs to have read rights on
                        # the realm container, principal container and realm sub-trees
                        ldap_kadmind_dn = "cn=admin,dc=example,dc=com"

                        # this object needs to have read and write rights on
                        # the realm container, principal container and realm sub-trees
                        ldap_service_password_file = /etc/krb5kdc/service.keyfile
                        ldap_servers = ldaps://ldap01.example.com ldaps://ldap02.example.com
                        ldap_conns_per_server = 5
                }

    > **Note**
    >
    > Change *example.com*, *dc=example,dc=com*, *cn=admin,dc=example,dc=com*,
    > and *ldap01.example.com* to the appropriate domain, LDAP object, and
    > LDAP server for your network.

-   Next, use the kdb5\_ldap\_util utility to create the realm:

        sudo kdb5_ldap_util -D  cn=admin,dc=example,dc=com create -subtrees \
        dc=example,dc=com -r EXAMPLE.COM -s -H ldap://ldap01.example.com

-   Create a stash of the password used to bind to the LDAP server. This
    password is used by the *ldap\_kdc\_dn* and *ldap\_kadmin\_dn* options in
    `/etc/krb5.conf`:

        sudo kdb5_ldap_util -D  cn=admin,dc=example,dc=com stashsrvpw -f \
        /etc/krb5kdc/service.keyfile cn=admin,dc=example,dc=com

-   Copy the CA certificate from the LDAP server:

        scp ldap01:/etc/ssl/certs/cacert.pem .
        sudo cp cacert.pem /etc/ssl/certs

    And edit `/etc/ldap/ldap.conf` to use the certificate:

        TLS_CACERT /etc/ssl/certs/cacert.pem

    > **Note**
    >
    > The certificate will also need to be copied to the Secondary KDC, to
    > allow the connection to the LDAP servers using LDAPS.

You can now add Kerberos principals to the LDAP database, and they will be
copied to any other LDAP servers configured for replication. To add a
principal using the kadmin.local utility enter:

    sudo kadmin.local
    Authenticating as principal root/admin@EXAMPLE.COM with password.
    kadmin.local:  addprinc -x dn="uid=steve,ou=people,dc=example,dc=com" steve
    WARNING: no policy specified for steve@EXAMPLE.COM; defaulting to no policy
    Enter password for principal "steve@EXAMPLE.COM": 
    Re-enter password for principal "steve@EXAMPLE.COM": 
    Principal "steve@EXAMPLE.COM" created.

There should now be krbPrincipalName, krbPrincipalKey, krbLastPwdChange, and
krbExtraData attributes added to the *uid=steve,ou=people,dc=example,dc=com*
user object. Use the kinit and klist utilities to test that the user is indeed
issued a ticket.

> **Note**
>
> If the user object is already created the *-x dn="..."* option is needed to
> add the Kerberos attributes. Otherwise a new *principal* object will be
> created in the realm subtree.

## Secondary KDC Configuration {#kerberos-ldap-secondary-kdc}

Configuring a Secondary KDC using the LDAP backend is similar to configuring
one using the normal Kerberos database.

First, install the necessary packages. In a terminal enter:

    sudo apt install krb5-kdc krb5-admin-server krb5-kdc-ldap

Next, edit `/etc/krb5.conf` to use the LDAP backend:

    [libdefaults]
            default_realm = EXAMPLE.COM

    ...

    [realms]
            EXAMPLE.COM = {
                    kdc = kdc01.example.com
                    kdc = kdc02.example.com
                    admin_server = kdc01.example.com
                    admin_server = kdc02.example.com
                    default_domain = example.com
                    database_module = openldap_ldapconf
            }

    ...

    [domain_realm]
            .example.com = EXAMPLE.COM

    ...

    [dbdefaults]
            ldap_kerberos_container_dn = dc=example,dc=com

    [dbmodules]
            openldap_ldapconf = {
                    db_library = kldap
                    ldap_kdc_dn = "cn=admin,dc=example,dc=com"

                    # this object needs to have read rights on
                    # the realm container, principal container and realm sub-trees
                    ldap_kadmind_dn = "cn=admin,dc=example,dc=com"

                    # this object needs to have read and write rights on
                    # the realm container, principal container and realm sub-trees
                    ldap_service_password_file = /etc/krb5kdc/service.keyfile
                    ldap_servers = ldaps://ldap01.example.com ldaps://ldap02.example.com
                    ldap_conns_per_server = 5
            }

Create the stash for the LDAP bind password:

    sudo kdb5_ldap_util -D  cn=admin,dc=example,dc=com stashsrvpw -f \
    /etc/krb5kdc/service.keyfile cn=admin,dc=example,dc=com

Now, on the *Primary KDC* copy the `/etc/krb5kdc/.k5.EXAMPLE.COM` *Master Key*
stash to the Secondary KDC. Be sure to copy the file over an encrypted
connection such as scp, or on physical media.

    sudo scp /etc/krb5kdc/.k5.EXAMPLE.COM steve@kdc02.example.com:~
    sudo mv .k5.EXAMPLE.COM /etc/krb5kdc/

> **Note**
>
> Again, replace *EXAMPLE.COM* with your actual realm.

Back on the *Secondary KDC*, (re)start the ldap server only,

    sudo systemctl restart slapd.service

Finally, start the krb5-kdc daemon:

    sudo systemctl start krb5-kdc.service

Verify the two ldap servers (and kerberos by extension) are in sync.

You now have redundant KDCs on your network, and with redundant LDAP servers
you should be able to continue to authenticate users if one LDAP server, one
Kerberos server, or one LDAP and one Kerberos server become unavailable.

## Resources {#kerberos-ldap-resources}

-   The [Kerberos Admin Guide] has some additional details.

-   For more information on kdb5\_ldap\_util see [Section 5.6] and the
    [kdb5\_ldap\_util man page].

-   Another useful link is the [krb5.conf man page].

-   Also, see the [Kerberos and LDAP] Ubuntu wiki page.

# SSSD and Active Directory {#sssd-ad}

This section describes the use of sssd to authenticate user logins against an
Active Directory via using sssd's "ad" provider. In previous versions of sssd,
it was possible to authenticate using the "ldap" provider. However, when
authenticating against a Microsoft Windows AD Domain Controller, it was
generally necessary to install the POSIX AD extensions on the Domain
Controller. The "ad" provider simplifies the configuration and requires no
modifications to the AD structure.

## Prerequisites, Assumptions, and Requirements {#sssd-ad-requirements}

-   This guide does not explain Active Directory, how it works, how to set one
    up, or how to maintain it. It may not provide “best practices” for
    your environment.

-   This guide assumes that a working Active Directory domain is
    already configured.

-   The domain controller is acting as an authoritative DNS server for
    the domain.

-   The domain controller is the primary DNS resolver as specified in
    `/etc/resolv.conf`.

-   The appropriate *\_kerberos*, *\_ldap*, *\_kpasswd*, etc. entries are
    configured in the DNS zone (see Resources section for external links).

-   System time is synchronized on the domain controller (necessary
    for Kerberos).

-   The domain used in this example is *myubuntu.example.com* .

## Software Installation {#sssd-ad-installation}

The following packages are needed: *krb5-user*, *samba*, *sssd*, and *ntp*.
Samba needs to be installed, even if the system is not exporting shares. The
Kerberos realm and FQDN or IP of the domain controllers are needed for this
step.

Install these packages now.

    sudo apt install krb5-user samba sssd ntp

See the next section for the answers to the questions asked by the *krb5-user*
postinstall script.

## Kerberos Configuration {#sssd-ad-kerberos}

The installation of *krb5-user* will prompt for the realm name (in ALL
UPPERCASE), the kdc server (i.e. domain controller) and admin server (also the
domain controller in this example.) This will write the \[realm\] and
\[domain\_realm\] sections in `/etc/krb5.conf`. These sections may not be
necessary if domain autodiscovery is working. If not, then both are needed.

If the domain is *myubuntu.example.com*, enter the realm as
*MYUBUNTU.EXAMPLE.COM*

Optionally, edit */etc/krb5.conf* with a few additional settings to specify
Kerberos ticket lifetime (these values are safe to use as defaults):

    [libdefaults]

    default_realm = MYUBUNTU.EXAMPLE.COM
    ticket_lifetime = 24h #
    renew_lifetime = 7d
            

If default\_realm is not specified, it may be necessary to log in with
“username@domain” instead of “username”.

The system time on the Active Directory member needs to be consistent with
that of the domain controller, or Kerberos authentication may fail. Ideally,
the domain controller server itself will provide the NTP service. Edit
`/etc/ntp.conf`:

    server dc.myubuntu.example.com

## Samba Configuration {#sssd-ad-samba}

Samba will be used to perform netbios/nmbd services related to Active
Directory authentication, even if no file shares are exported. Edit the file
/etc/samba/smb.conf and add the following to the *\[global\]* section:

    [global]

    workgroup = MYUBUNTU
    client signing = yes
    client use spnego = yes
    kerberos method = secrets and keytab
    realm = MYUBUNTU.EXAMPLE.COM
    security = ads

> **Note**
>
> Some guides specify that "password server" should be specified and pointed
> to the domain controller. This is only necessary if DNS is not properly set
> up to find the DC. By default, Samba will display a warning if "password
> server" is specified with "security = ads".

## SSSD Configuration {#sssd-ad-sssdconfig}

There is no default/example config file for `/etc/sssd/sssd.conf` included in
the sssd package. It is necessary to create one. This is a minimal working
config file:

    [sssd]
    services = nss, pam
    config_file_version = 2
    domains = MYUBUNTU.EXAMPLE.COM

    [domain/MYUBUNTU.EXAMPLE.COM]
    id_provider = ad
    access_provider = ad

    # Use this if users are being logged in at /.
    # This example specifies /home/DOMAIN-FQDN/user as $HOME.  Use with pam_mkhomedir.so
    override_homedir = /home/%d/%u

    # Uncomment if the client machine hostname doesn't match the computer object on the DC.
    # ad_hostname = mymachine.myubuntu.example.com

    # Uncomment if DNS SRV resolution is not working
    # ad_server = dc.mydomain.example.com

    # Uncomment if the AD domain is named differently than the Samba domain
    # ad_domain = MYUBUNTU.EXAMPLE.COM

    # Enumeration is discouraged for performance reasons.
    # enumerate = true

After saving this file, set the ownership to root and the file permissions to
600:

    sudo chown root:root /etc/sssd/sssd.conf

    sudo chmod 600 /etc/sssd/sssd.conf

If the ownership or permissions are not correct, sssd will refuse to start.

## Verify nsswitch.conf Configuration {#sssd-ad-nsswitch}

The post-install script for the sssd package makes some modifications to
/etc/nsswitch.conf automatically. It should look something like this:

    passwd:         compat sss
    group:          compat sss
    ...
    netgroup:       nis sss
    sudoers:        files sss

## Modify /etc/hosts {#sssd-ad-hosts}

Add an alias to the localhost entry in /etc/hosts specifying the FQDN. For
example:

    192.168.1.10 myserver myserver.myubuntu.example.com

This is useful in conjunction with dynamic DNS updates.

## Join the Active Directory {#sssd-ad-join}

Now, restart ntp and samba and start sssd.

    sudo systemctl restart ntp.service
    sudo systemctl restart smbd.service nmbd.service 
    sudo systemctl start sssd.service

Test the configuration by obtaining a Kerberos ticket:

    sudo kinit Administrator

Verify the ticket with:

    sudo klist

If there is a ticket with an expiration date listed, then it is time to join
the domain:

    sudo net ads join -k

A warning about "No DNS domain configured. Unable to perform DNS Update."
probably means that there is no (correct) alias in `/etc/hosts`, and the
system could not provide its own FQDN as part of the Active Directory update.
This is needed for dynamic DNS updates. Verify the alias in `/etc/hosts`
described in "Modify /etc/hosts" above.

(The message "NT\_STATUS\_UNSUCCESSFUL" indicates the domain join failed and
something is incorrect. Review the prior steps before proceeding).

Here are a couple of (optional) checks to verify that the domain join was
successful. Note that if the domain was successfully joined but one or both of
these steps fail, it may be necessary to wait 1-2 minutes and try again. Some
of the changes appear to be asynchronous.

Verification option \#1:

Check the default Organizational Unit for computer accounts in the Active
Directory to verify that the computer account was created. (Organizational
Units in Active Directory is a topic outside the scope of this guide).

Verification option \#2

Execute this command for a specific AD user (e.g. administrator)

    getent passwd username

> **Note**
>
> If *enumerate = true* is set in `sssd.conf`, *getent passwd* with no
> username argument will list all domain users. This may be useful for
> testing, but is slow and not recommended for production.

## Test Authentication {#sssd-ad-test}

It should now be possible to authenticate using an Active Directory User's
credentials:

    su - username

If this works, then other login methods (getty, ssh) should also work.

If the computer account was created, indicating that the system was "joined"
to the domain, but authentication is unsuccessful, it may be helpful to review
`/etc/pam.d` and `nssswitch.conf` as well as the file changes described
earlier in this guide.

## Home directories with pam\_mkhomedir (optional) {#sssd-ad-mkhomedir}

When logging in using an Active Directory user account, it is likely that user
has no home directory. This can be fixed with pam\_mkdhomedir.so, which will
create the user's home directory on login. Edit `/etc/pam.d/common-session`,
and add this line directly after *session required pam\_unix.so:*

    session    required    pam_mkhomedir.so skel=/etc/skel/ umask=0022

> **Note**
>
> This may also need *override\_homedir* in `sssd.conf` to function correctly,
> so make sure that's set.

## Desktop Ubuntu Authentication {#sssd-ad-desktop}

It is possible to also authenticate logins to Ubuntu Desktop using Active
Directory accounts. The AD accounts will not show up in the pick list with
local users, so lightdm will need to be modified. Edit the file
`/etc/lightdm/lightdm.conf.d/50-unity-greeter.conf` and append the following
two lines:

    greeter-show-manual-login=true
    greeter-hide-users=true

Reboot to restart lightdm. It should now be possible to log in using a domain
account using either *username* or *username/username@domain* format.

## Resources {#sssd-ad-resources}

-   [SSSD Project]

-   [DNS Server Configuration guidelines]

-   [Active Directory DNS Zone Entries]

-   [Kerberos config options]

  [RFC4510]: http://tools.ietf.org/html/rfc4510
  [RFC2849]: http://tools.ietf.org/html/rfc2849
  [bug]: https://bugs.launchpad.net/ubuntu/+source/apparmor/+bug/1392018
  [slapd-config]: http://manpages.ubuntu.com/manpages/en/man5/slapd-config.5.html
  [Installation]: #openldap-server-installation
  [slapd.access]: http://manpages.ubuntu.com/manpages/en/man5/slapd.access.5.html
  [Replication]: #openldap-server-replication
  [TLS]: #openldap-tls
  [Samba and LDAP]: #samba-ldap
  [www.openldap.org]: http://www.openldap.org/
  [LDAP for Rocket Scientists]: http://www.zytrax.com/books/ldap/
  [OpenLDAP wiki]: https://help.ubuntu.com/community/OpenLDAPServer
  [LDAP System Administration]: http://www.oreilly.com/catalog/ldapsa/
  [Mastering OpenLDAP]: http://www.packtpub.com/OpenLDAP-Developers-Server-Open-Source-Linux/book
  [OpenLDAP Server]: #openldap-server
  [Modifying the slapd Configuration Database]: #openldap-configuration
  [???]: #samba
  [Samba HOWTO Collection]: http://samba.org/samba/docs/man/Samba-HOWTO-Collection/
  [passdb section]: http://samba.org/samba/docs/man/Samba-HOWTO-Collection/passdb.html
  [Linux Samba-OpenLDAP HOWTO]: http://download.gna.org/smbldap-tools/docs/samba-ldap-howto/
  [Samba Ubuntu community documentation]: https://help.ubuntu.com/community/Samba#samba-ldap
  [1]: #dns-primarymaster-configuration
  [2]: #NTP
  [3]: #dns
  [4]: #kerberos-server-installation
  [MIT Kerberos]: http://web.mit.edu/Kerberos/
  [Ubuntu Wiki Kerberos]: https://help.ubuntu.com/community/Kerberos
  [Kerberos: The Definitive Guide]: http://oreilly.com/catalog/9780596004033/
  [Freenode]: http://freenode.net/
  [Kerberos Admin Guide]: http://web.mit.edu/Kerberos/krb5-1.6/krb5-1.6.3/doc/krb5-admin.html#Configuring-Kerberos-with-OpenLDAP-back_002dend
  [Section 5.6]: http://web.mit.edu/Kerberos/krb5-1.6/krb5-1.6.3/doc/krb5-admin.html#Global-Operations-on-the-Kerberos-LDAP-Database
  [kdb5\_ldap\_util man page]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man8/kdb5_ldap_util.8.html
  [krb5.conf man page]: http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/krb5.conf.5.html
  [Kerberos and LDAP]: https://help.ubuntu.com/community/Kerberos#kerberos-ldap
  [SSSD Project]: https://fedorahosted.org/sssd
  [DNS Server Configuration guidelines]: http://www.ucs.cam.ac.uk/support/windows-support/winsuptech/activedir/dnsconfig
  [Active Directory DNS Zone Entries]: https://technet.microsoft.com/en-us/library/cc759550%28v=ws.10%29.aspx
  [Kerberos config options]: http://web.mit.edu/kerberos/krb5-1.12/doc/admin/conf_files/krb5_conf.html
