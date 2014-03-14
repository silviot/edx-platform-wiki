This page describes everything you need to know to begin development on the edX Developer Stack (also known as devstack).

### Installing devstack

The edX devstack is a Vagrant instance designed for local development. Before beginning development, read [edX Developer Stack](https://github.com/edx/configuration/wiki/edX-Developer-Stack) which describes how to install and run the edX software.

### Debugging the devstack image

Debugging with devstack is a little convoluted, as the source code lives on your local machine but the code is executing within the Vagrant image. One solution is to use [PyCharm](http://www.jetbrains.com/pycharm/) from JetBrains. It is able to debug devstack using its concept of [remote Python interpreters](http://blog.jetbrains.com/pycharm/2013/03/how-pycharm-helps-you-with-remote-development/). This only works for the Professional edition.

Here are the steps to use PyCharm on MacOS (other Unix environments should be similar). Note that this has been tested with PyCharm 3.1, and that these steps will *not* work on the free PyCharm Community Edition.

* First, start up a terminal and give the 'edxapp' user a password
```
  vagrant ssh
  sudo passwd edxapp
  [type in a password of your choice]
```
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
  * You should now have a remote interpreter. See [PyCharm's documentation for remote interpreters](http://www.jetbrains.com/pycharm/quickstart/configuring_interpreter.html#remote_vagrant) if you want more details
* Create a debug configuration for LMS
  * Open the file 'edx-platform/manage.py' in your devstack folder
  * Right-click on the file and choose "Create 'manage'..." to create a debug configuration
  * Specify the settings for the new configuration
    * Change the name to something more memorable, e.g. "LMS"
    * Specify "lms runserver --settings=devstack 0.0.0.0:8000" for "Script parameters"
    * Specify "/edx/app/edxapp/edx-platform" for "Working directory"
    * Click to edit the "Path mappings" property
      * Click the "+" button to add a new mapping
      * Specify the full local path to "edx-platform" for "Local path". This should be the edx-platform directly beneath your devstack folder, e.g. "/Users/andya/devstack/edx-platform"
      * Specify "/edx/app/edxapp/edx-platform" for "Remote path"
      * Click "OK" to save the mappings
    * Deselect "Add content roots to PYTHONPATH"
    * Deselect "Add source roots to PYTHONPATH"
  * Click "OK" to save the new debug configuration
* Rake assets (ssh into vagrant first). rake assets[lms,devstack] 
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

Note: for more details, check out the [PyCharm remote debugging documentation](http://www.jetbrains.com/pycharm/webhelp/remote-debugging.html).

### Testing your changes

See the [[Test Engineering FAQ]] for all your questions about testing the edX platform.

### Debugging your tests

PyCharm can also be used to debug Python tests.

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
