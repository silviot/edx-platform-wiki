### Debugging the devstack image with PyCharm professional edition

Debugging with devstack is a little convoluted, as the source code lives on your local machine but the code is executing within the Vagrant image. One solution is to use [PyCharm Professional Edition](http://www.jetbrains.com/pycharm/) from JetBrains. It is able to debug devstack using its concept of [remote Python interpreters](http://blog.jetbrains.com/pycharm/2013/03/how-pycharm-helps-you-with-remote-development/). Similar setups should also work for any IDE with remote interpreter.

Here are the steps to use PyCharm on MacOS (other Unix environments should be similar). 

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

### Create a PyCharm remote interpreter (PyCharm v.3.3 and below):
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
* (OPTIONAL) to make PyCharm's editor traverse imports in edx-platform python modules, go to Preferences -> Project Structure and mark the following directories as Source Folders:
  * lms/djangoapps and lms/lib
  * cms/djangoapps and cms/lib
  * common/djangoapps and common/lib

### Create a PyCharm remote interpreter (PyCharm v3.4+)
* Select "Preferences..." from the Apple menu
* Expand "Project Interpreter" under "Project Settings"
* Click the icon in the upper right corner of the dialog box--it looks like it may be a gear or a star.
* In the popup dialog that appears, click on "Add Remote"
* When the remote dialog appears, select the "SSH Credentials" radio button.
* Specify the following configurations:
![PyCharm 3.4+ conf](https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/pycharm_conf.jpg)

### Create Debug Configuration for LMS
After the PyCharm remote interpreter is configured we are ready to debug devstack.

* Create a debug configuration for LMS server
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
  * Your LMS server configuration should look something similar to:
![LMS server configuration]
(https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/LMS_server.jpg)
* To build (rake) assets:
  * (ssh into vagrant first, and su as edxapp user).
  * ```paver update_assets lms --settings=devstack```
* Debug the new "LMS" configuration
  * Choose "Run > Debug..."
  * Specify "LMS" and the Django instance should be started up
  * You should now be able to set breakpoints and hit them

### Create a debug configuration for Studio (CMS) 
* The CMS configuration is very similar to LMS, so we need to clone LMS configuration and adjust a few parameters. 
  * Choose "Run > Edit Configurations..."
  * Select "LMS"
  * Click the "Copy configuration" button (next to the "-" button)
  * Change the name to "Studio"
  * Change the "Script parameters" to ```cms runserver --settings=devstack 0.0.0.0:8001```
  * Click "OK" to save the new configuration
* Remember to rake assets (ssh into vagrant first) before running Studio from Pycharm.
* ```paver update_assets cms --settings=devstack```
* If you have XQuartz installed, you can test the server directly by going logging in as edxapp and type ```firefox```.

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

### Setting up Bokchoy Test Configuration
![Bokchoy Test Configuration]
(https://lh5.googleusercontent.com/ZjmlhWeugiRD-gBfzTJcVLM3jvpJ4wRXZ9DpSguFGpVOla9qNseEsrLdeE_W1cVzCaIICA=w1416-h682)

### Visually debug your tests
You can setup your development environment such that you can visually interact with browsers and other GUIs in the vagrant machine from the host machine. To do this, you will need to install [XQuartz](http://xquartz.macosforge.org/landing/). For further information, see [Test Engineering FAQ](https://github.com/edx/edx-platform/wiki/Test-engineering-FAQ#im-working-with-devstack-and-want-to-debug-the-jasmine-or-acceptance-tests-in-the-browser-on-my-host-system-how-do-i-do-that).
