This page is a short guide on the changes that will be introduced by the [pull request 6026](https://github.com/edx/edx-platform/pull/6026). This request changes the way the platform manages the static assets by switching the current workflow made in paver to a Grunt-based one using libsass and bower.

### Grunt
Grunt is a task runner built on node js. It lets us run tasks like the grunt-sass task, which in turn uses the Libsass library to compile the css assets, or the grunt-contrib-coffee to compile coffescript into javascript.

### Libsass
[Libsass](https://github.com/sass/libsass) is a C/C++ port of the Sass css precompiler. Using this library removes the dependecy on ruby to compile the static assets.

### Bower
Bower is a package manager for the frontend dependencies.

## Getting started
To get started, there is nothing in particular you need to do. Just use paver as you would usually do. On devstack:

        sudo su edxapp
        paver devstack lms

You will notice that while running the command `npm install` some extra packages are being downloaded. This will among other things also install bower, which in turn will download soem packages to the `bower_components` directory.

After the first time installation is done, paver continues running as usual and before running the django server it will call grunt to build the assets. You will see a message from grunt:
![Grunt time example](http://i.imgur.com/0ySYjCf.png)
That is it, browse to localhost:8000 and continue your development.

When you want to re-process the static files, run `grunt lms`. 


### Watch
If you want to modify the static assets managed by grunt (e.g. scss files or coffescripts), you may want to pass the --watch flag to paver

        paver devstack lms --watch
This way, grunt will listen for changes in the files and run again every time a file is saved.


## Advanced usage
In order to use more specific commands or magnify the effects on performance read on.

There are many tasks defined in the Gruntfile.js file, you can call each of them by running the `grunt <task>` command. Here is a list of commands and its effect.

* grunt lms -> build the css and js for the lms system.
* grunt lms:css -> build only the css of the lms system.
* grunt lms:js -> build only the javascript of the lms system.
* grunt lms:watch -> listen for changes in the locations where static assets are located and builds the javascript and css when a change is detected.
* grunt lms:dev -> runs the lms task and after it is done, watches for changes.
* grunt lms:dist -> deletes all the css and js files, then compiles and minifies them.


All this tasks are available for studio as well, just use 'studio' instead of 'lms'

### Performance
In order to speed up the build time(from 10-15 seconds to 1-3) on vagrant environments like devstack, there is an optimization available. This is due to the high performance penalties imposed by the shared folder implementation.

To bypass this, the solution is to run the grunt command from the native OS instead of the ubuntu OS installed in the virtual machine.

To do this you need to [install node in the native OS](https://github.com/joyent/node/wiki/installing-node.js-via-package-manager) and then install the grunt command line interface. On devstack:

        sudo npm install grunt-cli -g

After this is done, you can call all the grunt tasks directly from edx-platform directory in the console on your native OS. This will reduce the time it takes to compile the assets to around 2 seconds. Also the time it takes for the watch task to notice the changes in the files from around 5 seconds to almost nothing.  

