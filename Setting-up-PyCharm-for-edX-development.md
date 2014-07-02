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

### Create a PyCharm remote interpreter (PyCharm v3.4+)
* Select "Preferences..." from the Apple menu
* Expand "Project Interpreter" under "Project Settings"
* Click the icon in the upper right corner of the dialog box--it looks like it may be a gear or a star.
* In the popup dialog that appears, click on "Add Remote"
* When the remote dialog appears, select the "SSH Credentials" radio button.
* Specify the following configurations:
![PyCharm 3.4+ conf](https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/pycharm_remote_conf.png)
*PyCharm will automatically detect the helper folder and upload the debugging files. The final configuration should look like this:
![PyCharm 3.4+ conf] (https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/pycharm_conf.jpg)
* After that, you should verify that PyCharms has copied all debugging materials into the ```.pycharm_helpers``` folder by ssh-ing into the vagrant instance and navigating to the helper folder. The directory content should look similar to this:
![PyCharm helper](https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/pycharm_helpers.png)
 

### Create Debug Configuration for LMS
After the PyCharm remote interpreter is configured we are ready to debug devstack.

* Create a debug configuration for LMS server
  * Open the file 'edx-platform/manage.py' in your devstack folder
  * Right-click on the file and choose "Create 'manage'..." to create a debug configuration
  * Specify the settings for the new configuration:
    * Change the name to something more memorable, e.g. "LMS".
    * Specify "./manage.py" for "Script" (thus using a relative path rather than an absolute one).
    * Specify "lms runserver --settings=devstack 0.0.0.0:8000" for "Script parameters".
    * Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
    * Specify "/edx/app/edxapp/edx-platform" for "Working directory".
    * Click to edit the "Path mappings" property
      * Click the "+" button to add a new mapping
      * Specify the full local path to "edx-platform" for "Local path". This should be the edx-platform directly beneath your devstack folder, e.g. "/Users/andya/devstack/edx-platform"  (Be sure to specify "Users" rather than the shorthand "~" here.)
      * Specify "/edx/app/edxapp/edx-platform" for "Remote path".
      * Click "OK" to save the mappings.
    * Deselect "Add content roots to PYTHONPATH".
    * Deselect "Add source roots to PYTHONPATH".
  * Click "OK" to save the new debug configuration.
  * Your LMS server configuration should look something similar to:
![LMS server configuration]
(https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/LMS_server.png)
* To build (rake) assets:
  * (ssh into vagrant first, and su as edxapp user).
  * ```paver update_assets lms --settings=devstack```
* Debug the new "LMS" configuration
  * Choose "Run > Debug..."
  * Specify "LMS" and the Django instance should be started up.
  * You should now be able to set breakpoints and hit them.

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

### Debugging your Paver tests in PyCharms

Now with the remote interpreter, we can use PyCharm debug Paver tests (JavaScript, Python, Bok Choy, Acceptance, CMS...). The process is fairly simple. Since almost all the test suits can be run using ```paver``` command with different parameters, we just need to supply the appropriate paver path and parameters.

### Setting up JavaScript Test Configuration
* Go to Run -> Edit Configurations -> Add New Configuration (usually a +sign on the left).
* Script: ```/edx/app/edxapp/venvs/edxapp/bin/paver```.
* Script parameters: ```test_js```. 
* Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
* Working directory: ```/Users/[username]/devstack/edx-platform```
* Path mappings: ```/Users/[username]/devstack/edx-platform=/edx/app/edxapp/edx-platform```

### Setting up Python Test Configuration
* Go to Run -> Edit Configurations -> Add New Configuration (usually a +sign on the left).
* Script: ```/edx/app/edxapp/venvs/edxapp/bin/paver```.
* Script parameters: ```test_python```. 
* Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
* Working directory: ```/Users/[username]/devstack/edx-platform```
* Path mappings: ```/Users/[username]/devstack/edx-platform=/edx/app/edxapp/edx-platform```

### Setting up Bokchoy Test Configuration
![Bokchoy Test Configuration]
(https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/bokchoy_server.png)
* Go to Run -> Edit Configurations -> Add New Configuration (usually a +sign on the left).
* Script: ```/edx/app/edxapp/venvs/edxapp/bin/paver```.
* Script parameters: ```test_bokchoy``` or ```test_bokchoy --fasttest``` (if you have already compiled the assets). 
* Choose the remote interpreter (usually named as "Remote Python 2.7.3 (ssh://edxapp.127.0.0.1:2222/...)").
* Working directory: ```/Users/[username]/devstack/edx-platform```
* Path mappings: ```/Users/[username]/devstack/edx-platform=/edx/app/edxapp/edx-platform```

### Visually debug your tests with XQuartz
You can setup your development environment such that you can visually interact with browsers and other GUIs in the vagrant machine from the host machine. To do this, you will need to install [XQuartz](http://xquartz.macosforge.org/landing/). For further information, see [Test Engineering FAQ](https://github.com/edx/edx-platform/wiki/Test-engineering-FAQ#im-working-with-devstack-and-want-to-debug-the-jasmine-or-acceptance-tests-in-the-browser-on-my-host-system-how-do-i-do-that).

### Integrate XQuartz into PyCharms
The following instructions are for Mac OS X but should work on any Unix-like environment.
* First, we need to enable XQuartz forward for all ssh agents. To do this, open ```/etc/ssh_config``` and uncomment ```ForwardX11 no``` and change to ```ForwardX11 yes```. We might need to restart the shell.
* We open a terminal inside PyCharms (Tools => Open Terminal...") and type ```ssh edxapp@127.0.0.1 -p 2222```. After logging in as edxapp user, try to open Firefox by typing ```firefox``` into the terminal. XQuartz should fire up Firefox immediately.
* Now we need to get the display port number from this terminal by ```echo $DISPLAY```. Depending on the number of existing ssh terminals, usually it says ```localhost:10.0``` if this is the only one.
* Open a debug configuration (such as Bokchoy) and add DISPLAY to the environment variable with the port number from the previous test. The string ```DISPLAY=localhost:10.0``` should be inside the environment variable string.
![Display Port Environmental Variable]
(https://1786529bf2dfcc9a4fc2736524bc8aea4a66cc50.googledrive.com/host/0BxQlaq542xl2V182QTM4ZF9kZlU/display_port.png)
* Now we can start debugging Paver tests with browser windows popping up in XQuartz.