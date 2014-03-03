## General questions
### How do I get started with edX development?

See [[Developing on the edX Developer Stack]] for details as to how to develop and debug the edX platform.

### Where is the general documentation on testing the edx_platform code?
Here is a link to doc included in the repo itself on [Writing and Running Tests](https://github.com/edx/edx-platform/blob/master/docs/en_us/internal/testing.md). Many questions are answered here.

### I'm working with devstack and want to debug the Jasmine or Acceptance tests in the browser on my host system. How do I do that?

* First off, for Mac OS you will need [XQuartz](http://xquartz.macosforge.org/) installed to support X Windows. We have tested with version 2.7.5.
* Make sure this stanza is in your Vagrantfile (in the devstack directory, from which you usually `vagrant up`). It should be, as this was [merged in on 2/28/2014](https://github.com/edx/configuration/commits/master/vagrant/release/devstack/Vagrantfile).
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