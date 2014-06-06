To enable Stanford theming in a sandbox instance:

1) Add the following lines to /edx/app/edx_ansible/server-vars.yml

```
edxapp_use_custom_theme: true
edxapp_theme_name: 'default'
edxapp_theme_source_repo: 'git://github.com/Stanford-Online/edx-default-theme.git'
edxapp_theme_version: 'HEAD'
```

2) Re-run the provisioning scripts:

```
sudo /edx/bin/update edx-platform master
```