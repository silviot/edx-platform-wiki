#Browser Sync
[Browser Sync](http://www.browsersync.io/) helps the web developer to multiple browsers & devices in sync when building websites. It also provides auto refresh for your files when they change.

This small tutorial is to help you install it and work on it with your devstack.

##Install `browser-sync`

    $ sudo npm install -g browser-sync

##Run the Server
Please not that my theme is in the following directory, your mileage may vary:

    $ cd edx-platform/lms/
    $ browser-sync start \
        --port 8100 \
        --proxy='localhost:8000' \
        --files='static/**/*.css, templates/**/*.html, static/**/*.js, /edx/app/edxapp/themes/edraak/templates/**/*.html'
