#Browser Sync
[Browser Sync](http://www.browsersync.io/) helps the web developer to multiple browsers & devices in sync when building websites. It also provides auto refresh for your files when they change.

This small tutorial is to help you install it and work on it with your devstack.

##Install `browser-sync`

    $ sudo npm install -g browser-sync

##Run the Server
Replace PLATFORM_PATH and THEME_NAME with your paths.

    $ cd PLATFORM_PATH/edx-platform/lms/
    $ browser-sync start \
        --port 8100 \
        --proxy='localhost:8000' \
        --files='static/**/*.css, templates/**/*.html, static/**/*.js, /edx/app/edxapp/themes/THEME_NAME/templates/**/*.html'
