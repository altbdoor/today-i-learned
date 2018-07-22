# Vagrant, NFS, Virtualbox and Windows

July 8, 2018

So, I tried to make NFS to work on Windows host, with Ubuntu guest. And this is the documentation of sorts. The steps are largely inspired from this [Intechgrity post](https://www.intechgrity.com/nfs-based-file-sharing-for-varying-vagrant-under-windows/), which was looking to make it work for Drupal.

> Windows users: NFS folders do not work on Windows hosts. Vagrant will ignore your request for NFS synced folders on Windows.
> 
> &mdash; [Vagrant Docs](https://www.vagrantup.com/docs/synced-folders/nfs.html)

Firstly, setting up NFS for Windows. To be clear, Vagrant does not officially support NFS on Windows. There is an effort to write a proper NFS implementation for Windows, which is [WinNFSd](https://github.com/winnfsd/winnfsd/). The latest release at time of writing is [v2.4.0](https://github.com/winnfsd/winnfsd/releases/tag/2.4.0), and it works well for me.

Since Windows might get wonky with the path issues and whatnot, I recommend making a folder as close to `C:\` as possible. Personally, I made a folder path with `C:\winnfsd`. Then, download the `WinNFSd.exe` and place it within that folder. An example of executing the executable:

```cmd
C:\Users\HP>C:\winnfsd\WinNFSd.exe
=====================================================
WinNFSd 2.4.0 [5f7f224]
Network File System server for Windows
Copyright (C) 2005 Ming-Yang Kao
Edited in 2011 by ZeWaren
Edited in 2013 by Alexander Schneider (Jankowfsky AG)
Edited in 2014 2015 by Yann Schepens
Edited in 2016 by Peter Philipp (Cando Image GmbH), Marc Harding
=====================================================

Usage: WinNFSd.exe [-id <uid> <gid>] [-log on | off] [-pathFile <file>] [-addr <ip>] [export path] [alias path]

...
```

Now, on the same folder (`C:\winnfsd`), create an `exports.txt` file, which specifies which folders you would like to share over NFS. Specify the folders in each line, like so:

```
C:\Users\HP\_projects
C:\Users\HP\Downloads
```

I personally only expose the `_projects` folder, but you get the idea.

And lastly, to actually run the NFS service, we create a batch file to make things easier. Also for convenience, its going into `C:\winnfsd\nfs_start.bat`.

```bat
@echo off
start C:\winnfsd\WinNFSd.exe -log off -pathFile C:\winnfsd\exports.txt
```

As a short recap, the folder `C:\winnfsd` should have:
- `exports.txt`, the file listing which folders to share over NFS
- `WinNFSd.exe`, yours truly
- `nfs_start.bat`, the batch script to kick it off

With a `cmd /c C:\winnfsd\nfs_start.bat`, the NFS server will run, and all will be well. At least for Windows, for now. To note, the Intechgrity post I referenced has a more [detailed batch script](https://wpquark.io/vagrant/windows-requirement/blob/master/winnfsd/nfsstart.bat) which ensures only one instance of WinNFSd is running at all times. I am just counting on the fact that I will never run it twice.

Onwards to Vagrant, we need to setup `Vagrantfile` such that it mounts the NFS folders. We will need a private network, and a `synced_folder` directive which uses NFS.

```ruby
Vagrant.configure("2") do |config|
  ...

  # set it as private network
  config.vm.network "private_network", type: "dhcp"

  # add a new synced folder
  config.vm.synced_folder "C:\\Users\\HP\\_projects", "/srv/projects",
    SharedFoldersEnableSymlinksCreate: false, owner: nil, group: nil,
    type: "nfs", mount_options: ['rw', 'vers=3', 'tcp', 'fsc' ,'actimeo=1']

  ...
```

Notice how the first parameter passed to `synced_folder` (`"C:\\Users\\HP\\_projects"`) corresponds to the folder path specified in `exports.txt` earlier. The second parameter is where it should be mounted in the Ubuntu guest. For the subsequent parameters:

- `SharedFoldersEnableSymlinksCreate: false`, ideas of using symbolic link in Windows is most dangerous, and thus disabled
- `owner: nil`, `group: nil`, I do not know what this does, but if you would like to read, it is specified in the [documentation](https://www.vagrantup.com/docs/synced-folders/basic_usage.html#options)
- `type: "nfs"`, is what specifies this as a NFS shared folder
- `mount_options`, are arguments for the NFS, which can be understood more in [its manual](https://linux.die.net/man/5/nfs)

Finally, navigate to where the `Vagrantfile` resides, and execute `vagrant plugin install vagrant-winnfsd`. Run `vagrant up`, and keep your fingers crossed.

If all goes well, you should be able to SSH and `cd` into `/srv/projects`. Voila!

And now there's still one more small problem. NFS unmounts itself after some time of inactivity. There are lots of way to tackle it, but I did it via an infinite timed loop Bash script, which writes to the NFS shared folder. Not the most elegant, but it works.

```sh
#!/bin/bash

if [[ -d /srv/projects/ ]]; then
    while true; do
        echo $(date) > /srv/projects/nfs.lock
        sleep 30
    done
fi
```

---

Update: July 23, 2018

It seems that setting up a private network on Vagrant would cause some issues. I could not be sure what is the real cause, but it boils down to `systemd-networkd-wait-online`, which is waiting for the network to get ready, but the network will never get ready. So Ubuntu will attempt to wait until the service times out, which is about 10 to 30 minutes.

The unofficial solution is to reduce the timeout duration in `/lib/systemd/system/systemd-networkd-wait-online.service`, at the `ExecStart` line.

```diff
- ExecStart=/lib/systemd/systemd-networkd-wait-online
+ ExecStart=/lib/systemd/systemd-networkd-wait-online --timeout 1
```

---

#### References

- https://www.intechgrity.com/nfs-based-file-sharing-for-varying-vagrant-under-windows/
- https://www.vagrantup.com/docs/synced-folders/basic_usage.html
- https://www.vagrantup.com/docs/synced-folders/nfs.html
- https://github.com/winnfsd/winnfsd/releases
- https://linux.die.net/man/5/nfs
