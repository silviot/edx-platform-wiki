The following instructions will help you to download and setup a virtual machine 
with a minimal amount of steps, using Vagrant. It is recommended for a first 
installation, as it will save you from many of the common pitfalls of the
installation process.

1. Install Git: http://git-scm.com/downloads
2. Install VirtualBox: https://www.virtualbox.org/wiki/Downloads
3. Install Vagrant: http://www.vagrantup.com/ (Vagrant 1.2.2 or later)
4. Open a terminal
5. Download the project: `git clone git://github.com/antoviaque/edx-platform.git` (Important! The feature is still being tested, so **the repository is different from the official one**, you need to re-download it)
6. Enter the project directory: `cd edx-platform/`
7. Start: `vagrant up`

The last step might require your administrator password to setup NFS. 

Afterwards, it will download an image, install all the dependencies and configure the 
VM. It will take a while, go grab a coffee.

Once completed, hopefully you should see a "Success!" message indicating that the 
installation went fine. (If not, refer to the Troubleshooting section below.)

Accessing the VM
----------------

Vagrant should automatically log you in the virtual machine once the installation
is finished. You can also type, from another terminal:

```
$ vagrant ssh
```

Note: This won't work from Windows, install install PuTTY from 
http://www.chiark.greenend.org.uk/%7Esgtatham/putty/download.html instead. Then 
connect to 127.0.0.1, port 2222, using vagrant/vagrant as a user/password.

Using edX
---------

Once inside the VM, you can start Studio and LMS with the following commands
(from the `/edx/edx-platform` folder):

Learning management system (LMS):

```
$ rake lms[cms.dev,0.0.0.0:8000]
```

Studio:

```
$ rake cms[dev,0.0.0.0:8001]
```

Once started, open the following URLs in your browser:

* Learning management system (LMS): http://192.168.20.40:8000/ 
* Studio (CMS): http://192.168.20.40:8001/

You can develop by editing the files directly in the `edx-platform/` directory you 
downloaded before, you don't need to connect to the VM to edit them (the VM uses
those files to run edX, mirroring the folder in `/edx/edx-platform`).

Stopping & starting
-------------------

To stop the VM (from your `edx-platform/` directory):

```
$ vagrant halt
```

To restart:

```
$ vagrant up
```

or, to start without attempting to update the dependencies:

```
$ vagrant up --no-provision
```

Troubleshooting
---------------

### Reinstalling

If something goes wrong, you can easily recreate the installation from scratch by 
typing:

```
$ vagrant destroy -f && vagrant up
```

This will delete the current VM, create a new VM, re-install all the dependencies,
and reconfigure.

### "Mounting NFS shared folders failed"

If you get the following error message when you run `$ vagrant up`:

```
It appears your machine doesn't support NFS, or there is not an
adapter to enable NFS on this machine for Vagrant. Please verify
that `nfsd` is installed on your machine, and try again. If you're
on Windows, NFS isn't supported.
```

You need to install NFS. Under Debian/Ubuntu, for example:

```
$ sudo apt-get install nfs-common nfs-kernel-server
```