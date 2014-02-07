To enable Stanford theming in a sandbox instance:

1) Add the following lines to /edx/var/edx_ansible/server-vars.yml

```
edxapp_use_custom_theme: true
edxapp_theme_name: 'stanford'
edxapp_theme_source_repo: 'git://github.com/Stanford-Online/edx-theme.git'
edxapp_theme_version: 'HEAD'
```

2) Re-run the provisioning scripts:

```
sudo /edx/bin/update edx-platform master
```