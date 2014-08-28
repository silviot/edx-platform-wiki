### Allow All Advanced Components (first time only) ###
Custom XBlocks are not allowed by default, you need to update settings.

Like all platform settings, you can either write inside `/edx/app/edx_ansible/server-vars.yml` and update the platform to generate the custom settings files.

    EDXAPP_FEATURES:
        ALLOW_ALL_ADVANCED_COMPONENTS: true

Or you can manually edit the custom settings, in `/edx/app/edxapp/cms.env.json`.

This will override the default value hardcoded in `/edx/app/edxapp/edx-platform/cms/envs/common.py`.

Once the file is updated, you need to reboot to reload the settings.

    sudo /edx/bin/supervisorctl -c /edx/etc/supervisord.conf restart edxapp:

### Install / Update a XBlock ###
To install a XBlock, you must download it, and then run the install script. To update an existing XBlock, use --upgrade.

    # Move to the folder where you want to download the XBlock
    cd /edx/app/edxapp
    # Download the XBlock
    sudo -u edxapp git clone https://github.com/xxx/yourXBlock.git
    # If not installed: Install the XBlock
    sudo -u edxapp /edx/bin/pip.edxapp install yourXBlock/
    # If installed: Upgrade the XBlock using --upgrade
    sudo -u edxapp /edx/bin/pip.edxapp install yourXBlock/ --upgrade
    # Remove the installation files
    sudo rm -r yourXBlock

### Reboot if something isn't right ###
In some cases, rebooting is necessary to use the XBlock.

    sudo /edx/bin/supervisorctl -c /edx/etc/supervisord.conf restart edxapp:

### Activate the XBlock in your course ###
Go to `Settings -> Advanced Settings` and set `advanced_modules` to `["xblockname"]`.

### Use the XBlock in a unit ###
Select `Advanced -> XBlock Name` in your unit.