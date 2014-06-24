This page describes everything you need to know to begin development on the edX Developer Stack (also known as devstack).

### Installing devstack

The edX devstack is a Vagrant instance designed for local development. Before beginning development, read [edX Developer Stack](https://github.com/edx/configuration/wiki/edX-Developer-Stack) which describes how to install and run the edX software.

### Debugging the devstack image

Debugging with devstack is a little convoluted, as the source code lives on your local machine but the code is executing within the Vagrant image. One solution is to use [PyCharm](http://www.jetbrains.com/pycharm/) from JetBrains. It is able to debug devstack using its concept of [remote Python interpreters](http://blog.jetbrains.com/pycharm/2013/03/how-pycharm-helps-you-with-remote-development/). This only works for the Professional edition.
* If you want to experiment with PyCharm, you can [download a trial](http://www.jetbrains.com/pycharm/download/) of the professional edition.  This is different than the open-source edition which doesn't support the vagrant functionality.

Here are the steps to use PyCharm on MacOS (other Unix environments should be similar). Note that this has been tested with PyCharm 3.1, and that these steps will *not* work on the free PyCharm Community Edition.

* First, start up a terminal and give the 'edxapp' user a password
```
  vagrant ssh
  sudo passwd edxapp
  [type in a password of your choice]
```
* Edit `/etc/ssh/sshd_config` and change PasswordAuthentication to `yes`

* Next, create the .pycharm_helpers directory where PyCharm will store its remote debugging code
```
  sudo su edxapp
  mkdir .pycharm_helpers
```
* Create a PyCharm project for devstack (if you don't have one already):
  * Select "New Project..." from the "File" menu
  * Select your devstack folder as the "Location"
  * Choose "Empty project" for "Project type" (note: don't choose "Django project" as that can't currently be interpreted remotely)
  * Say "Yes" to the question about using existing sources for the project
* Create a PyCharm remote interpreter:

  (NOTE: If you are using PyCharm v3.4 or later, skip this section and use the steps described in [section below] (https://github.com/edx/edx-platform/wiki/Developing-on-the-edX-Developer-Stack#creating-a-remote-python-interpreter-pycharm-v34). When you finish with those steps continue back here with the [next section to create debug configurations] (https://github.com/edx/edx-platform/wiki/Developing-on-the-edX-Developer-Stack#create-debug-configuration-lms-studio-for-pycharm).)

  * Select "Preferences..." from the Apple menu
  * Expand "Project Interpreter" under "Project Settings"
  * Select "Python Interpreters"
  * Click the "+" button to add a new interpreter
  * Click on "Remote..." to choose the type of interpreter
  * Click on "Fill from vagrant config"
  * Specify your devstack directory and hit "OK"
  * Update the settings for your new interpreter
    * Specify "edxapp" for "User name"
    * Select "Password" for "Auth type"
    * Specify your edxapp password 
    * Check "Save password"
    * Specify "/edx/app/edxapp/venvs/edxapp/bin/python" for "Python interpreter path"
  * Click "Test connection..." to verify that the settings are correct
  * Click "OK"
  * Click "Yes" to the prompt "Do you want to set this interpreter as Project Interpreter?"
  * You should now have a remote interpreter, *do check*. Something is not fully understood here, it might be that you need to "apply" as well as "saving" at some point. If this step is not properly done, you won't find the "path mapping" step later in the process. Otherwise, you can consult [PyCharm's documentation for remote interpreters](http://www.jetbrains.com/pycharm/quickstart/configuring_interpreter.html#remote_vagrant) if you want more details
* (OPTIONAL) to make PyCharm's editor traverse imports in edx-platform python modules:
  * go to Preferences -> Project Structure
  * mark the following directories as Source Folders:
    * lms/djangoapps and lms/lib
    * cms/djangoapps and cms/lib
    * common/djangoapps and common/lib

### Create Debug Configuration (LMS, Studio) for PyCharm

* Create a debug configuration for LMS
  * Open the file 'edx-platform/manage.py' in your devstack folder
  * Right-click on the file and choose "Create 'manage'..." to create a debug configuration
  * Specify the settings for the new configuration
    * Change the name to something more memorable, e.g. "LMS"
    * Specify "./manage.py" for "Script" (thus using a relative path rather than an absolute one)
    * Specify "lms runserver --settings=devstack 0.0.0.0:8000" for "Script parameters"
    * Specify "/edx/app/edxapp/edx-platform" for "Working directory"
    * Click to edit the "Path mappings" property
      * Click the "+" button to add a new mapping
      * Specify the full local path to "edx-platform" for "Local path". This should be the edx-platform directly beneath your devstack folder, e.g. "/Users/andya/devstack/edx-platform"  (Be sure to specify "Users" rather than the shorthand "~" here.)
      * Specify "/edx/app/edxapp/edx-platform" for "Remote path"
      * Click "OK" to save the mappings
    * Deselect "Add content roots to PYTHONPATH"
    * Deselect "Add source roots to PYTHONPATH"
  * Click "OK" to save the new debug configuration
* Rake assets (ssh into vagrant first, and su as edxapp user). rake assets[lms,devstack]
* Debug the new "LMS" configuration
  * Choose "Run > Debug..."
  * Specify "LMS" and the Django instance should be started up
  * You should now be able to set breakpoints and hit them
* Create a debug configuration for Studio
  * Choose "Run > Edit Configurations..."
  * Select "LMS"
  * Click the "Copy configuration" button (next to the "-" button)
  * Change the name to "Studio"
  * Change the "Script parameters" to ```cms runserver --settings=devstack 0.0.0.0:8001```
  * Click "OK" to save the new configuration
* Remember to rake assets (ssh into vagrant first) before running Studio from Pycharm. rake assets[cms,devstack]
* To create other testing configurations (Bok Choy, Acceptance, CMS, ...) see the [next section] (https://github.com/edx/edx-platform/wiki/Developing-on-the-edX-Developer-Stack#debugging-your-tests).

Note: for more details, check out the [PyCharm remote debugging documentation](http://www.jetbrains.com/pycharm/webhelp/remote-debugging.html).

### Testing your changes

See the [[Test Engineering FAQ]] for all your questions about testing the edX platform.

### Debugging your tests

PyCharm can also be used to debug Python tests (Bok Choy, Acceptance, CMS...).

* Create a debug configuration for an edX common unit test
  * Choose "Run > Edit Configurations..."
  * Select the "Studio" configuration
  * Click the "Copy configuration" button (next to the "-" button)
  * Change the name to "Studio CommonTests"
  * Change the script to ```/edx/app/edxapp/venvs/edxapp/bin/nosetests```
  * Change the "Script parameters" to run the test:
    * e.g. ```common/lib/xmodule/xmodule/tests/test_resource_templates.py```
* Create a debug configuration for an edX CMS unit test
  * Choose "Run > Edit Configurations..."
  * Select the "Studio" configuration
  * Click the "Copy configuration" button (next to the "-" button)
  * Change the name to "Studio Tests"
  * Change the "Script parameters" to run the test:
    * e.g. ```cms --settings test test cms/djangoapps/contentstore/views/tests/test_helpers.py```
* Create a debug configuration for an edX acceptance test
  * Choose "Run > Edit Configurations..."
  * Select the "Studio" configuration
  * Click the "Copy configuration" button (next to the "-" button)
  * Change the name to "Studio Acceptance Tests"
  * Change the "Script parameters" to run the test:
    * e.g. ```cms --settings acceptance harvest --traceback --debug-mode --verbosity 2 --with-xunit --xunit-file /edx/app/edxapp/edx-platform/reports/acceptance/cms.xml cms/djangoapps/contentstore/features/problem-editor.feature```

Debugging Bok Choy tests is more complex, as several processes are started up at once including LMS and Studio. If one of these processes is already running then Bok Choy will use it instead. So, if you want to set a breakpoint in Studio, you need to create a clone of the Studio configuration that has the following Script parameter
* ```./manage.py cms runserver --settings bok_choy 0.0.0.0:8031```

<a name="theme"></a>
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



### Creating a remote python interpreter (PyCharm v3.4+)
In June of 2014 JetBrains released a new version of PyCharm. The new version handles the creation of a remote interpreter a bit differently than the steps described above for earlier versions. 
* Create a PyCharm remote interpreter:
  * Select "Preferences..." from the Apple menu
  * Expand "Project Interpreter" under "Project Settings"
  * Click the icon in the upper right corner of the dialog box--it looks like it may be a gear or a star.
  * In the popup dialog that appears, click on "Add Remote"
  * When the remote dialog appears, select the "Vagrant" radio button
  * Click on the "..." navigation icon to the right of the "Vagrant Instance Folder:" textbox
  * In the "Select Path" dialog box that appears, navigate to and select the directory where your ".vagrant" folder and "Vagrantfile" live 
  * You should see a small dialog with a progress bar and the message "Running Vagrant ssh-config"
  * When the process completes and the dialog disappears, test your connection by clicking on the ssh hyperlink indicated by "Vagrant Host URL:"
  * Specify "/edx/app/edxapp/venvs/edxapp/bin/python" for "Python interpreter path"
![PyCharm 3.4+ conf](https://drive.google.com/file/d/0BxQlaq542xl2Vlp6ZE0tY2JPa2s/edit?usp=sharing)





### The edx-platform database that devstack uses
To syncdb and migrate from the shell as the edxapp user:
```
./manage.py lms --settings=devstack syncdb
./manage.py lms --settings=devstack migrate
```

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