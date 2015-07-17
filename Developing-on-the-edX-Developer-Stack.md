<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](http://doctoc.herokuapp.com/)*

- [Installing devstack](#installing-devstack)
- [Debugging the devstack image](#debugging-the-devstack-image)
- [Testing your changes](#testing-your-changes)
- [Visually debug your tests](#visually-debug-your-tests)
- [Configuring Themes in Devstack](#configuring-themes-in-devstack)
- [The edx-platform database that devstack uses](#the-edx-platform-database-that-devstack-uses)
- [Making the local servers run faster](#making-the-local-servers-run-faster)
- [Other resources](#other-resources)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

This page describes everything you need to know to begin development on the edX Developer Stack (also known as devstack).

### Installing devstack

The edX devstack is a Vagrant instance designed for local development. Before beginning development, read [edX Developer Stack](https://github.com/edx/configuration/wiki/edX-Developer-Stack) which describes how to install and run the edX software.

### Debugging the devstack image

Debugging with devstack is a little convoluted, as the source code lives on your local machine but the code is executing within the Vagrant image. One solution is to use [PyCharm](http://www.jetbrains.com/pycharm/) from JetBrains. It is able to debug devstack using its concept of [remote Python interpreters](http://blog.jetbrains.com/pycharm/2013/03/how-pycharm-helps-you-with-remote-development/). This only works for the Professional edition.
* If you want to experiment with PyCharm, you can [download a trial](http://www.jetbrains.com/pycharm/download/) of the professional edition.  This is different than the open-source edition which doesn't support the vagrant functionality.

See [Setting up PyCharm for edX development](https://github.com/edx/edx-platform/wiki/Setting-up-PyCharm-for-edX-development).

### Testing your changes

See the [[Test Engineering FAQ]] for all your questions about testing the edX platform.

### Visually debug your tests
You can setup your development environment such that you can visually interact with browsers and other GUIs in the vagrant machine from the host machine. To do this, you will need to install [XQuartz](http://xquartz.macosforge.org/landing/). For further information, see [Test Engineering FAQ](https://github.com/edx/edx-platform/wiki/Test-engineering-FAQ#im-working-with-devstack-and-want-to-debug-the-jasmine-or-acceptance-tests-in-the-browser-on-my-host-system-how-do-i-do-that).

### Configuring Themes in Devstack

There are currently two ways to theme the platform:

* **"Stanford Theming"** -- This is a set of conditions in the code
  that key off a settings (```settings.FEATURES.USE_CUSTOM_THEME``` and
  ```settings.THEME_NAME```) to look aside to custom styling (assets)
  and templates.  It's a bit of a hack due to pipeline complexity
  (sass/ruby dependencies) and how Mako templates are named and loaded.

* **"Microsites"** -- These are separate conditions that change site
  appearance dynamicaly keyed off the hostname of the request.
  foo.edx.org can appear one way, and bar.edx.org can appear another.  

This section describes how to use the former with devstack, written by
Sef (<sef@stanford.edu>).

1. **Configure**. Change two settings in ```/edx/app/edxapp/lms.envs.json```

        "FEATURES": {
            ...
            "USE_CUSTOM_THEME": true
            ...
        },


    and

        ...
        "THEME_NAME": "default",
        ...

2. **Mount**. Make sure Vagrant mounts the "theme" directory from the host.
   This should be the case with Johnnycake or later Vagrant installations.  If
   you have an older Vagrantfile, see [PR #884](https://github.com/edx/configuration/pull/884).

3. **Check out**. The rest of the devstack code checks out repos as part
   of the provision step, but that tooling doesn't happen for themes.  You
   need to check out a theme yourself.  I recommend you start with the "null theme" 
   that we've made here at Stanford:

        cd themes
        git clone git@github.com:Stanford-Online/edx-default-theme.git default

4. **Gather assets**. The rest of the devstack methods seems to imply the
   service variant (lms or cms) automatically, but this doesn't work
   with the theme pipeline.  So to make this work I have to gather assets
   this way:

        vagrant ssh
        sudo su edxapp
        paver update_assets lms --settings=devstack

5. **From PyCharm**.  I created an external command to do this from
   PyCharm. 

    1. Open up "Remote SSH External Tools"
    2. Name = "lms assets"
    3. Connection settings = Default interpreter.  This assumes that
       you've setup up pycharm with the default interpreter to ssh into
       the edxapp user. 
    4. Program = ```bash```
    5. Parameters = ```-c "SERVICE_VARIANT=lms /edx/app/edxapp/.gem/bin/rake lms:gather_assets:devstack"```
    6. Working directory = ```/edx/app/edxapp/edx-platform```
    7. Screenshots:
       [configuring](image/devstack_theme_gather_config.png) and 
       [resulting menu item](image/devstack_theme_gather_menu.png).

There is nothing special about the name "default" in steps 1 and 3 above.  They just have to match up: the configuration file should have the same name as the directory in themes.  For example, I have both "stanford" and "default" subdirectories there and switching between them is just a matter of changing lms.envs.json, re-gathering assets, and restarting LMS.




### The edx-platform database that devstack uses
To syncdb and migrate from the shell as the edxapp user:
```
./manage.py lms --settings=devstack syncdb
./manage.py lms --settings=devstack migrate
```
If there are errors, you might be able to resolve them with 
```
./manage.py lms --settings=devstack migrate --merge
```
followed by syncdb'ing

Devstack uses mySQL as the DB engine for the edx-platform app.
You can tell this by starting up the Django shell with:

```
./manage.py lms shell --settings=devstack
```

Then in the Django shell:
```
from django.conf import settings
settings.DATABASES
```

You can manipulate the database with the mysql command line interface from within your vagrant terminal session:
```
mysql --user=root --port=3306  edxapp
```

If you want to use a GUI tool from your host system, that is also possible.
E.g. for MySQL Workbench, you would set up a new Server connection with these settings:
* Connection Method: Standard TCP/IP over SSH
* SSH Hostname: 192.168.33.10:22
* SSH Username: vagrant (the password is 'vagrant')
* MySQL Hostname: 127.0.0.1
* MySQL Server Port: 3306
* Username: root (no password)
* Default Schema: edxapp

Note: You may also need to specify the SSH Key File, which should be located at "~/.vagrant.d/insecure_private_key"

If you get the "Could not connect the SSH Tunnel; Authentication failed, please check credentials" error while trying to establish connection from a GUI tool, but you're able to access mysql from vagrant terminal session, check to see which SSH Key File your vagrant instance is using. 
You can do that by using the following command from your vagrant folder on the host machine: 
```
use vagrant ssh-config
```
In case your vagrant instance is not using "~/.vagrant.d/insecure_private_key" as the SSH Key File, you'll need to specify the key your vagrant instance is using.

### Making the local servers run faster

While running LMS and Studio locally, you may want to conditionally disable certain features in order to improve performance. To do so, create the files lms/envs/private.py and cms/envs/private.py and paste in this code:

```
DISABLE_DJANGO_TOOLBAR = True
DISABLE_CONTRACTS = True

if DISABLE_DJANGO_TOOLBAR:
    from .common import INSTALLED_APPS, MIDDLEWARE_CLASSES

    def tuple_without(source_tuple, exclusion_list):
        """Return new tuple excluding any entries in the exclusion list. Needed because tuples
        are immutable. Order preserved."""
        return tuple([i for i in source_tuple if i not in exclusion_list])

    INSTALLED_APPS = tuple_without(INSTALLED_APPS, ['debug_toolbar', 'debug_toolbar_mongo'])
    MIDDLEWARE_CLASSES = tuple_without(MIDDLEWARE_CLASSES, [
        'django_comment_client.utils.QueryCountDebugMiddleware',
        'debug_toolbar.middleware.DebugToolbarMiddleware',
    ])

    DEBUG_TOOLBAR_MONGO_STACKTRACES = False

if DISABLE_CONTRACTS:
    import contracts
    contracts.disable_all()
```

Disabling both the Django toolbar and contracts should speed up the your local LMS and Studio instances significantly. 

**IMPORTANT NOTE:** this may cause devstack to behave strangely in certain scenarios, such as running acceptance tests. If something on your devstack is unexplainably not working, try setting both DISABLE_ flags to False. 

### Other resources

There are a number of other useful documents on the edX platform wiki:

* [[How to Contribute]]
* [[How to Rebase a Pull Request]]
* [[Python Guidelines]]
* [[JavaScript Guidelines]]
* [[Test Engineering FAQ]]
* [[Hackathons]]
* [[Writing easily XBlock portable XModules]]
* [[Internationalization and Localization]]
* [[Split Testing]]
* [[Object Identities]]
* [[Data-Access,-Persistence]]
* [[Helpful Tools]]