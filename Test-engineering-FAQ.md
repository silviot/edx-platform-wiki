## General questions
### How do I get started with edX development?

See [[Developing on the edX Developer Stack]] for details as to how to develop and debug the edX platform.

### Where is the general documentation on testing the edx_platform code?
Here is a link to doc included in the repo itself on [Writing and Running Tests](https://github.com/edx/edx-platform/blob/master/docs/en_us/internal/testing.md). Many questions are answered here.

### How do I run paver test for a single file?
To run single test, specify the name of the test file. For example:

```paver test_bokchoy -t test_lms.py```

To run single test faster by not repeating setup tasks:

```paver test_bokchoy -t test_lms.py --fasttest```

To test only a certain feature, specify the file and the testcase class:

```paver test_bokchoy -t test_lms.py:RegistrationTest```

To execute only a certain test case, specify the file name, class, and test case method:

```paver test_bokchoy -t test_lms.py:RegistrationTest.test_register```

If the test is in a subfolder, just specify the path:

```paver test_bokchoy -t video/test_video_module.py```

### Adding bash completion to paver
Courtesy of [Gregory Nicholas](https://groups.google.com/forum/#!topic/paver/Ba5YNXNhs9U)

```
_paver()
{
    local cur
    COMPREPLY=()
    # Variable to hold the current word
    cur="${COMP_WORDS[COMP_CWORD]}"

    # Build a list of the available tasks from: `paver --help --quiet`
    local cmds=$(paver -hq | awk '/^  ([a-zA-Z][a-zA-Z0-9_]+)/ {print $1}')

    # Generate possible matches and store them in the
    # array variable COMPREPLY
    COMPREPLY=($(compgen -W "${cmds}" $cur))
}

# Assign the auto-completion function for our command.

complete -F _paver paver
```

### I'm working with devstack and want to debug the Jasmine or Acceptance tests in the browser on my host system. How do I do that?

* First off, for Mac OS you will need [XQuartz](http://xquartz.macosforge.org/) installed to support X Windows. We have tested with version 2.7.5.
* XQuartz is useful because we can use it to visually interact with GUI from the vagrant instance on the host machine. 
* On your host machine, run `export VAGRANT_X11=1`. You may want to put this in your .bashrc, .bash_profile or similar file so it will always execute at startup. If you have already been running vagrant, do `vagrant reload` after you have set the VAGRANT_X11 system environment variable on your host system.
* To test XQuartz's installation, in your vagrant machine, type `firefox` in the terminal. This should bring up [Firefox on the host machine] (https://drive.google.com/a/edx.org/file/d/0BxQlaq542xl2QzBHNjU0WUNMRGM/edit?usp=sharing). If you have servers running on the vagrant machine (Studio, LMS, ...) you can use this Firefox instance to connect to those servers like they are being run on the host machine.
* Similarly, you can also run `rake test:bok_choy` (or if you have compiled the asset files, then `rake test:bok_choy:fast`) in vagrant's terminal. This should also bring up the browser in the host machine.
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

### I just got a new vagrant instance but some of my tests (Bok Choy, Acceptance, ...) seem to be failing. Should I be worried?
* Check the particular test's instruction to see if you have accidentally skipped a step. Possible solutions include:
  * Clean up the git repo: `git clean` and rebuild the static files: `rake assets[cms,devstack]`
  * Restart MongoDB server: 
`vagrant up
vagrant ssh
sudo rm /edx/var/mongo/mongodb/mongod.lock
sudo start mongodb`
  * In the worst case, get a fresh Vagrant instance.

### I got an error saying that Mongo is not running locally. What should I do?
Under Vagrant user, run:

```
sudo rm /edx/var/mongo/mongodb/mongod.lock
sudo start mongodb
```