Note: you might also be interested in [[Developing on the edX Developer Stack]].

## General questions
### Where is the general documentation on test the edx_platform code?
Here is a link to doc included in the repo itself on [Writing and Running Tests](https://github.com/edx/edx-platform/blob/master/docs/en_us/internal/testing.md). Many questions are answered here.

### Debugging the devstack image

Debugging with devstack is a little more convoluted, as the source code lives on your local machine, but the code is executing within the Vagrant VM. [PyCharm](http://www.jetbrains.com/pycharm/) is able to debug devstack using its concept of [remote Python interpreters](http://blog.jetbrains.com/pycharm/2013/03/how-pycharm-helps-you-with-remote-development/).

Here are the steps to use PyCharm on MacOS (other Unix environments should be similar):
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
  * Click "Save"
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
* Debug the new "LMS" configuration
  * Choose "Run > Debug..."
  * Specify "LMS" and the Django instance should be started up
  * You should now be able to set breakpoints and hit them
* Create a debug configuration for Studio
  * Choose "Run > Edit Configurations..."
  * Select "LMS"
  * Click the "Copy configuration" button (next to the "-" button)
  * Change the name to "Studio"
  * Change the "Script parameters" to "cms runserver --settings=devstack 0.0.0.0:8001"
  * Click "OK" to save the new configuration

### I'm working with devstack and want to debug the Jasmine or Acceptance tests in the browser on my host system. How do I do that?

* First off, you need [XQuartz](http://xquartz.macosforge.org/) installed on your host *nix system. We have tested with version 2.7.5.
* Put this stanza into your Vagrantfile (in the devstack directory, from which you usually `vagrant up`) at the same level as the other config.vm.* commands. E.g. before or after the config.vm.network statements.
```
  # Enable X11 forwarding so we can interact with GUI applications
  if ENV['VAGRANT_X11']
      config.ssh.forward_x11 = true
  end
```
* Set the VAGRANT_X11 environment variable on your host machine, then reload the image. Note that the reload will *not* reprovision your vagrant image if it has already been provisioned. It *will* redo the port forwarding and setting up of the file shares. See the vagrant docs for more info on the vagrant commands.
```
export VAGRANT_X11=1
vagrant reload
```
* ssh into the vagrant image. Note you will be the 'vagrant' user. Start up firefox, **then quit it**. (It will be passed through to your host machine's display).
```
vagrant ssh

firefox
```
* Now copy the 'vagrant' user's ~/.Xauthority file somewhere that the 'edxapp' user can get to it (like the /tmp dir). Then as the 'edxapp' user copy that .Xauthority file to its home directory. Also as the 'edxapp' user, set the DISPLAY environment variable to the host display.
```
cp .Xauthority /tmp
chmod 0666 /tmp/.Xauthority 
sudo su edxapp
cp /tmp/.Xauthority ~
export DISPLAY=localhost:10.0
```

* Once this has been set up, if you want to run the acceptance tests without browser windows popping up, redirect the DISPLAY environment variable.
```
export DISPLAY=:1
```