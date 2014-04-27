If you have a custom theme called "customtheme" in a sandbox instance:

1) Add the following lines to /edx/app/edx_ansible/server-vars.yml

```
edxapp_use_custom_theme: true
edxapp_theme_name: 'customtheme'
edxapp_theme_source_repo: 'git://github.com/yourusername/custom-theme.git'
edxapp_theme_version: 'HEAD'
```
If the repository is private also add
```
edxapp_git_identity: '/edx/app/edxapp/tmp_id_rsa'
EDXAPP_LOCAL_GIT_IDENTITY: '/edx/app/edxapp/.ssh/id_rsa'
EDXAPP_USE_GIT_IDENTITY: true
```
where id_rsa is a key generated for edxapp user and deployed/configured in your git server

2) Re-run the provisioning scripts:

```
sudo /edx/bin/update edx-platform master