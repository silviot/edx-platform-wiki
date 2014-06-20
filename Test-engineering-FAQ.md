## General questions
### How do I get started with edX development?

See [[Developing on the edX Developer Stack]] for details as to how to develop and debug the edX platform.

### Where is the general documentation on testing the edx_platform code?
Here is a link to doc included in the repo itself on [Writing and Running Tests](https://github.com/edx/edx-platform/blob/master/docs/en_us/internal/testing.md). Many questions are answered here.

### I'm working with devstack and want to debug the Jasmine or Acceptance tests in the browser on my host system. How do I do that?

* First off, for Mac OS you will need [XQuartz](http://xquartz.macosforge.org/) installed to support X Windows. We have tested with version 2.7.5.
* XQuartz is useful because we can use it to visually interact with GUI from the vagrant instance on the host machine. 
* To test XQuartz's installation, in your vagrant machine, type "firefox" in the terminal. This should bring up Firefox on the host machine. If you have servers running on the vagrant machine (Studio, LMS, ...) you can use this Firefox instance to connect to those servers like they are being run on the host machine.
* On your host machine, run `export VAGRANT_X11=1`. You may want to put this in your .bashrc or similar file so it will always execute at startup. If you have already been running vagrant, do `vagrant reload` after you have set the VAGRANT_X11 system environment variable on your host system.
* We used to have to do a bunch of manual stuff to update the .Xauthority files, etc. This has been fixed programmatically in the configuration repo, prior to the release of 20140418-injera-devstack.box
* If you want to run the acceptance tests without browser windows popping up on your host system, as the edxapp user redirect the DISPLAY environment variable.
```
export DISPLAY=:1
```

* If you want to switch back to using your host system's display:
```
export DISPLAY=localhost:10.0
```

### Everything was working at first, but now my tests hang and fail without launching the host browser.  What happened?

If you are seeing messages like this on a Mac:
```
WebDriverException: Message: 'The browser appears to have exited before we could connect. The output was: Error: cannot open display: localhost:10.0\n'
```
then you may be experiencing X11 forwarding timeouts.  The problem and solution are discussed in detail [here](http://b.kl3in.com/2012/01/x11-display-forwarding-fails-after-some-time/), and the quick fix is to increase the value for `ForwardX11Timeout` in /etc/ssh_config to a nice high value (596h is the apparent maximum).


### I've changed some code and in the diff-cover report, those lines are coming up as uncovered. But I know I'm testing them!
* Changes to .coffee files will always show up as uncovered by diff-cover. This is because JsCover is the coverage reporter and it doesn't know about .coffee files, just .js files.
* Changes to /common/lib files that are covered with tests under /foo/djangoapps (where foo = common, cms, lms) are reported as uncovered. That's not a unit test. Your test code should also be somewhere under /common/lib, close to what you are testing. Common library functionality should not presume a Django implementation.