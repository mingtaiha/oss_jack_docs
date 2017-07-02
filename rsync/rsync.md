# Setting up an rsync server

This guide assume that the server and client both have `rsync` installed. Most Linux
distros should have `rsync` installed.

### Setting up an rsync server

1) We set up a config file (`/etc/rsync.conf`). The guide to `rsync.conf` files can be found in the man pages. This is one example of a config file used:

```
motd file = /etc/rsycd.motd
log file = /var/log/rsyncd.log
pid file = /var/run/rsyncd.pid
lock file = /var/runrsync.lock
max connections = 2

[mingtha]
    path = /home/rsync/backup_laptop
    comment = laptop backup area
    uid = nobody
    gid = nobody
    read only = no
    list = yes
    auth users = rsync
    secrets file /etc/rsyncd.scrt
```

#### Global Parameters

The following are common global parameters used:

`motd file` is the file that holds message of the day. This file's contents are displayed when the client tries to access this machine. The message can be a greeting, a warning, or whatever you want. By default no `motd` file is specified

`log file` specifies a log file to which to send diagnostic and normal run-time mesasges

`pid file` specifies the file that contains the process ID (PID) number of the running rsync daemon

By default, the port to which the daemon will listen is 873, and the protocol used is TCP. If rsync is installed on your machine, then this service should be enabled. You can always check the `/etc/services` file to see if the port/protocol is there

#### Module Parameters

The following are module parameters. Module parameters are specified within a module. Modules can be created by specifying a module name in square brackets (e.g. `[module_name]`)

`lock file` is used to support the `max connections` parameter. The rsync daemon uses record locking on this file to ensure that the max connection limit is not exceeded.

`max connections` specifies the number of simultaneous connections we allow. Any client connecting when the maximum has been reached will receieve a message telling them to try again later. The default is `0`, which means no limit.

`path` specifies the path to which the client-specified files will be backed up

`comment` allows users to store information about what an rsync module does. Mainly for human-readability

`uid`, `gid` specifies the the user ID and group ID that file transfers to and from the module should take place as when the daemon runs as root. By default it is specified as `nobody`. This is to minimize the damage done to the rest of the system when a malicious user takes advantage of a vulnerable service.

`read only` determines wheter cliens can upload files or not. If true, then uploads are not allowed. Otherwise (no), uploads are allowed

`list` determins whether the module is listen when a client asks for a listing for all available modules. If false, the daemone will pretend the module does not exist when a client is denied by `hosts allow` or `hosts deny`. The default is for modules to be listable.

`auth users` specify the usernames that are allowed to connect to this module. The usernames do not need to exist on the local system. Wildcard arguments can be used, and groupnames can be specified in place of usernames by prepending `@` to the groupname (`username` vs `@groupname`). If `auth users` is set then the client will be challenged to supply a username and password. The username:password is stored in a file specified by the `secrets files` parameter. The default is for all users to be able to connect without a password

In order to allow only authorized users to rsync, you can set up passwordless SSH to avoid having to type the password. The advantage of setting this up is that you can authorize users AND automate rsync

`secrets file` stores the username:password for each user or group specified in the `auth user` field. Make sure that only the root can read and write to this file!!!


Once all that is specified, restart the rsync service (`service rsyncd restart`)


### Rsync on the client side

2) The user can run rsync either in the terminal, or set up a cron job that periodically backs up files. For more details about using `rsync`, check the man pages.

To push local files to remote directory
`rsync -e 'ssh -p 22' -avl --stats --progress --compress --recursive --times --links --delete <local_dir1> [<local_dir2 [ ... ]] <username>@<remote_ip>:<path_to_backup_dir>`

To pull files in remote directory to local directory, specify the remote locations before the local directory

`rsync` only checks the differences in files and makes changes based on the difference. This is better than having to copy the entire file over every time with scp, which can a huge pain.

While `rsync` can only copy files over to the backup folder, one can copy entire directories by specifying the `--recursive` option. Just be sure that when specifying the folder, the folder does not end in a `/`. Otherwise, `rsync` will rsync the files in the folder to the destination folder WITHOUT THE FOLDER

`-e` allows you to choose an alternative shell program for communication between local and remote copies of `rsync`. This is set to `ssh` by default, but you can configure options and arguments to `ssh`, or use a different shell

`--links` recreates the symlinks encountered locally in the desintation

`-a` means archive, and is a quick way of saying you want recursion and want to preserve almost everything (links, ...), though there are some restrictions. This is relieves us of the need to specify many of the subsequent options. The other options are mentioned in case you want more control, since `-a` does a lot

`-v` means verbose, and increases the amount of information given during the transfer

`--links` or `-l` recreates the symlinks encountered locally in the desintation. Many other options have two names. Specifying at least one is fine. Which one you use is up to you

`--times` tells rsync to transfer modification times along with the files and update them on the remote system.

`--delete` tells rsync to delete extraneous files from the receiving side (but aren't on the sending side), only for directories that are being synchronized

`--progress` tells rsync to print information on the progress of the transfer

`--compress` tells `rsync` to compress the data as it is sent to the destination machine, which reduces the amount fo data being transmitted

`--stats` tells rsync to print a verbose set of statistics on the file transfer
