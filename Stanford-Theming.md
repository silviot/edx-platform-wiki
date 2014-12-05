# Provisioning method

To enable Stanford theming in a sandbox instance:

1) Add the following lines to /edx/app/edx_ansible/server-vars.yml

```
edxapp_use_custom_theme: true
edxapp_theme_name: 'default'
edxapp_theme_source_repo: 'https://github.com/Stanford-Online/edx-theme.git'
edxapp_theme_version: 'HEAD'
```

2) Re-run the provisioning scripts:

```
sudo /edx/bin/update edx-platform master
```

Please note: the stanford theme has some customized code implemented in the latest `edx-platform` and it would only be activated when `edx_theme_name` is set to `stanford`. Also another hack should be applied for successful compiling: `./static/sass/_stanford.scss` should be renamed to `stanford.scss`

# Manual method

Theming only works for the LMS. The CMS and the Wiki are left unchanged.

1) You should first modify `/edx/app/edxapp/lms.env.json`, and set FEATURES.USE_CUSTOM_THEME (True), THEME_NAME (the theme directory name) and PLATFORM_NAME (it will replace "edX" in many views).

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
