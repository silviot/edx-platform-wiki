# Provisioning method

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

# Manual method

Theming only works for the LMS.

1) You should first modify `/edx/app/edxapp/lms.env.json`, and set USE_CUSTOM_THEME (True), THEME_NAME (the theme directory name) and PLATFORM_NAME (it will replace "edX" in many views).

2) You must put theme files in `/edx/app/edxapp/themes/<theme-name>/`.
Put your images in `themes/<theme-name>/static/images`.
The `themes/<theme-name>/static/sass` directory should at least contain a file named `_<theme-name>.scss` (can be empty).
The `themes/<theme-name>/templates` directory must contain 4 files :
- theme-head-extra.html (can be empty)
- theme-header.html
- theme-footer.html
- theme-google-analytics.html (can be empty)

3) You shall then [recompile the LMS assets](https://github.com/edx/configuration/wiki/edX-Managing-the-Production-Stack#compile-assets-manually).

If you want deeper customisations, you shall begin by looking at the [/edx/app/edxapp/edx-platform/lms/templates/main.html](https://github.com/edx/edx-platform/blob/master/lms/templates/main.html) file.
