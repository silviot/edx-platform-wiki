## General questions
### Where is the general documentation on test the edx_platform code?
Here is a link to doc included in the repo itself on [Writing and Running Tests](https://github.com/edx/edx-platform/blob/master/docs/en_us/internal/testing.md). Many questions are answered here.

### I'm working with devstack and want to debug the Jasmine or Acceptance tests in the browser on my host system. How do I do that?

* Make sure that your Vagrantfile (in the devstack directory, from which you usually `vagrant up`) contains the following lines. They were added in either empanada or focaccia, I forget which, so unless you have a really old Vagrantfile it should contain these lines:
```
  # Enable X11 forwarding so we can interact with GUI applications
  if ENV['VAGRANT_X11']
      config.ssh.forward_x11 = true
  end
```
* If you had to change your Vagrantfile, you need to reload the image. Note that the reload will *not* reprovision your vagrant image if it has already been provisioned. It *will* redo the port forwarding and setting up of the file shares. See the vagrant docs for more info on the vagrant commands.
```
vagrant reload
```
* Set the VAGRANT_X11 environment variable on your host machine, then ssh into the vagrant image. Note you will be the 'vagrant' user. Then copy that user's ~/.Xauthority file somewhere that the 'edxapp' user can get to it (like the /tmp dir). Then as the 'edxapp' user copy that .Xauthority file to its home directory. Also as the 'edxapp' user, set the DISPLAY environment variable to the host display.
```
export VAGRANT_X11=1
vagrant ssh

cp .Xauthority /tmp
chmod 0666 /tmp/.Xauthority 
sudo su edxapp
cp /tmp/.Xauthority ~
export DISPLAY=localhost:10.0
```
