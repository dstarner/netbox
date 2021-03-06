## Review the Release Notes

Prior to upgrading your NetBox instance, be sure to carefully review all [release notes](../../release-notes/) that have been published since your current version was released. Although the upgrade process typically does not involve additional work, certain releases may introduce breaking or backward-incompatible changes. These are called out in the release notes under the version in which the change went into effect.

## Install the Latest Code

As with the initial installation, you can upgrade NetBox by either downloading the latest release package or by cloning the `master` branch of the git repository. 

### Option A: Download a Release

Download the [latest stable release](https://github.com/netbox-community/netbox/releases) from GitHub as a tarball or ZIP archive. Extract it to your desired path. In this example, we'll use `/opt/netbox`.

Download and extract the latest version:

```no-highlight
# wget https://github.com/netbox-community/netbox/archive/vX.Y.Z.tar.gz
# tar -xzf vX.Y.Z.tar.gz -C /opt
# cd /opt/
# ln -sfn netbox-X.Y.Z/ netbox
```

Copy the 'configuration.py' you created when first installing to the new version:

```no-highlight
# cp netbox-X.Y.Z/netbox/netbox/configuration.py netbox/netbox/netbox/configuration.py
```

Be sure to replicate your uploaded media as well. (The exact action necessary will depend on where you choose to store your media, but in general moving or copying the media directory will suffice.)

```no-highlight
# cp -pr netbox-X.Y.Z/netbox/media/ netbox/netbox/
```

Also make sure to copy over any reports that you've made. Note that if you made them in a separate directory (`/opt/netbox-reports` for example), then you will not need to copy them - the config file that you copied earlier will point to the correct location.

```no-highlight
# cp -r /opt/netbox-X.Y.Z/netbox/reports /opt/netbox/netbox/reports/
```

If you followed the original installation guide to set up gunicorn, be sure to copy its configuration as well:

```no-highlight
# cp netbox-X.Y.Z/gunicorn_config.py netbox/gunicorn_config.py
```

Copy the LDAP configuration if using LDAP:

```no-highlight
# cp netbox-X.Y.Z/netbox/netbox/ldap_config.py netbox/netbox/netbox/ldap_config.py
```

### Option B: Clone the Git Repository (latest master release)

This guide assumes that NetBox is installed at `/opt/netbox`. Pull down the most recent iteration of the master branch:

```no-highlight
# cd /opt/netbox
# git checkout master
# git pull origin master
# git status
```

## Run the Upgrade Script

Once the new code is in place, run the upgrade script:

```no-highlight
# ./upgrade.sh
```

This script:

* Destroys and rebuilds the Python virtual environment
* Installs all required Python packages
* Applies any database migrations that were included in the release
* Collects all static files to be served by the HTTP service
* Deletes stale content types from the database
* Deletes all expired user sessions from the database
* Clears all cached data to prevent conflicts with the new release

!!! note
    It's possible that the upgrade script will display a notice warning of unreflected database migrations:

        Your models have changes that are not yet reflected in a migration, and so won't be applied.
        Run 'manage.py makemigrations' to make new migrations, and then re-run 'manage.py migrate' to apply them.

    This may occur due to semantic differences in environment, and can be safely ignored. Never attempt to create new migrations unless you are intentionally modifying the database schema.

## Restart the NetBox Services

!!! warning
    If you are upgrading from an installation that does not use a Python virtual environment, you'll need to update the systemd service files to reference the new Python and gunicorn executables before restarting the services. These are located in `/opt/netbox/venv/bin/`. See the example service files in `/opt/netbox/contrib/` for reference.

Finally, restart the gunicorn and RQ services:

```no-highlight
# sudo systemctl restart netbox netbox-rq
```

!!! note
    It's possible you are still using supervisord instead of systemd.  If so, please see the instructions for [migrating to systemd](migrating-to-systemd.md).
