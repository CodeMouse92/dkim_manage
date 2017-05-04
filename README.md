# OpenDKIM Manager
A script to automate most tasks associated with OpenDKIM key rotation.

## Credits

- Written by Jason C. McDonald (CodeMouse92) for MousePaw Media.
- Perl code written by Kerin Millar (@kerframil)
- Additional feedback by Dorian Harmans (@woohooyeah).

## Setup

### System Requirements

OpenDKIM is designed to work in the Linux BASH (*not* SH) with minimal dependencies. It works with OpenDKIM, and therefore
depends on that being installed on the system. The script also uses Perl for one regex-replacement task.

### Installing

We recommend that the `dkim_manage` file be placed in a dedicated `scripts` folder, such as `/opt/scripts/root_scripts`.
Place `dkim_manage` into the directory in question, and then change its owner and permissions:

    $ chown root:root dkim_manage
    $ chmod +x dkim_manage
    
Next, create a new file *in that same folder* called `domains.conf`. (See our example in `domains.conf.example`).
In that file, list each of the domains that OpenDKIM is signing for, one per line. For example...

    example.com
    example.net
    
Save and close. Now, edit `dkim_manage`, and look for the following section. Edit each of the variables so it points to
the correct paths. **At minimum, you should change `CONFIG` to point to the `domains.conf` we just created.
If you aren't sure about the others, the defaults are probably correct, but we recommend double-checking.

    ### SYSTEM SPECIFIC VARIABLES - CHANGE TO MEET YOUR SYSTEM

    # Set the path to the configuration file for this script.
    CONFIG=/opt/scripts/root_scripts/domains.conf

    # Temporary directory for generating keys.
    TEMP=/tmp/dkim

    # The folder where OpenDKIM's keys are stored.
    PROD=/etc/opendkim/keys

    # The path to your OpenDKIM keytable.
    KEYTABLE=/etc/opendkim/key.table

    # The user for OpenDKIM, used for setting permissions on keys.
    USER=opendkim

    ### END VARIABLES

Save and close the file.

For convenience, you may consider adding the directory where `dkim_manage` lives to your system path.

## Using OpenDKIM Manager

(This guide assumes that the folder where `dkim_manage` resides exists in your system path.)

There are **four steps** to rotating your OpenDKIM keys. Be sure to follow each carefully, as it is possible to mess up
your OpenDKIM configuration or keys with improper use of this script.

This script MUST be run as root!

### Step 1: Generate New Keys

Be sure your `domain.conf` file is correct (see `Setup` above). Then, run...

    $ sudo dkim_manage -g

...to generate the new keys. These will be living in the temporary directory specified at the top of the script file
(`/tmp/dkim` by default), and OpenDKIM is not yet configured to use them.

### Step 2: Update DNS Records

To get the DNS records you need for your new OpenDKIM keys, run...

    $ sudo dkim_manage -d
    
This will display the new DNS record for each domain's key. Add these to your DNS TXT records, but *leave the old
records in place for now*, just in case something goes wrong. The displayed block of text is the **value** of the
record, while the record's name should be `YYYYMM._domainkey`, where `YYYY` is the current year, and `MM` is the
current month. For example, in January 2017, the record name would be `201701._domainkey`.

After updating your DNS records, you'll need to wait until the changes propegate publicly before continuing.

### Step 3: Testing the Keys

Once the DNS changes have propegated, you can ensure the new keys work using...

    $ sudo dkim_manage -t
    
Be sure *ALL* of the keys pass the test before continuing.

### Step 4: Deploy The Keys

**WARNING: Make sure you have carefully followed all the steps above before doing this! Moving the new keys into place
*will* completely and irreversably replace your old ones. Back up the old keys if you're paranoid.**

The last stage is to actually move the new keys into place. This involves moving them to the production directory
(specified by `PROD` in the script variables), and to update the keytable (specified by `KEYTABLE` in the script variables).
The key ownership will also be set (using the user and group specified by `USER` in the script variables).

    $ sudo dkim_manage -m
    
If you're **absolutely sure**, press "y" and ENTER. After a moment, the keys will have been moved, permissions updated,
keytable modified, and both OpenDKIM and Postfix restarted.

## Feedback

If you experience any problems with this script, please file a bug report on the Github (githubcom/CodeMouse92/dkim_manage).
If you have any suggestions for improving this script, please submit a pull request.
