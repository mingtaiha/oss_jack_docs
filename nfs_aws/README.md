# Setting up an NFS server

*Starting with a t2.medium instance from EC2, using RedHat 7.3 image (don't have any spare computers)*

### Firewall
EC2 allows/requires the user to configure the firewall. Luckily, it can be done on the AWS EC2 webpage
Had to add to the Security Group rules that Inbound TCP traffic on ports 111, 2049 are allowed for
    my IP (or security group). Also, enable Inbound UDP traffic on ports 111, 32806

#### RUWireless
I added what I think is the RUWireless IP subnet. Using ifconfig, I found that the subnet mask 
is 255.255.240.0, which means we have a /20 subnet. From googling my public IP, I found the
my public IP was usually 128.6.[>32].*. So, I set the security to be 128.6.32.0/20. This
subnet mask is just a guess, since there's no guarantee that the subnet starts at 128.6.32.0.

Upon getting the terminal, I ran `sudo su` to get root access

Had to install nfs-utils and rpcbind with `yum install nfs-utils rpcbind`. After installing, had to
run `/sbin/service nfs stop` (just in case nfs-utils was running) and then run 
`/sbin/service nfs start` in order to make sure the proper daemons were running

To check the daemons, run `rpcinfo -p` to check if `portmapper`, `nfs`, `nlockmgr`, `mountd`
were running. It should look something like this

```
    [ec2-user@ip-172-31-62-37 home]$ rpcinfo -p
        program vers proto   port  service
        100000    4   tcp    111  portmapper
        100000    3   tcp    111  portmapper
        100000    2   tcp    111  portmapper
        100000    4   udp    111  portmapper
        100000    3   udp    111  portmapper
        100000    2   udp    111  portmapper
        100024    1   udp  44103  status
        100024    1   tcp  33140  status
        100005    1   udp  20048  mountd
        100005    1   tcp  20048  mountd
        100005    2   udp  20048  mountd
        100005    2   tcp  20048  mountd
        100005    3   udp  20048  mountd
        100005    3   tcp  20048  mountd
        100003    3   tcp   2049  nfs
        100003    4   tcp   2049  nfs
        100227    3   tcp   2049  nfs_acl
        100003    3   udp   2049  nfs
        100003    4   udp   2049  nfs
        100227    3   udp   2049  nfs_acl
        100021    1   udp  53599  nlockmgr
        100021    3   udp  53599  nlockmgr
        100021    4   udp  53599  nlockmgr
        100021    1   tcp  34068  nlockmgr
        100021    3   tcp  34068  nlockmgr
        100021    4   tcp  34068  nlockmgr
```


First edited `/etc/exports` to control the access to a host. Made two directories `nfs_test_dir` for read-only (`ro`) access, and `reads_and_writes` for read/write (`rw`) access. This was specified by: `/path/to/dir' `machine`(option1, option2, ...)`

The IPs which I used was the same as the subnet which I gave access to using AWS credentials.

Went into `/etc/hosts.deny` to deny access of certain services to all users. The services were the daemons involved in NFS: `portmap`, `lockd`, `mountd`, `rquotad`, `statd` In `/etc/hosts.deny`, added `(daemon, machine)` pairs, namely (`daemon`:ALL)

Went into /etc/hosts.allow to enable access to the daemons previously blocked. When a server gets
    a request, it checks the hosts.allow to see if the requesting machine matchines a rule listed

    in /etc/hosts.allow, allowing acccess if there is a match. If there is no match in hosts.allow,
    the server then checks /etc/hosts.deny to see if a client machines a rule listed, and denies
    access if so. If the client does not match any lists in either file, then it is allowed access

    So, I am adding the same daemons to the hosts.allow file, specifying the IP of the machine I
    want to enable

After making changes to exports, hosts.allow or hosts.deny file, I ran `exportfs -ra` to to update 
    the NFS server



# Setting up an NFS Client

Running on an Ubuntu 16.04 laptop.

Had to install rpcbind and nfs-common (client version), with 
    `sudo apt-get install nfs-common rpcbind`

On the client side, I ran `rpcinfo -p` and got

```
	   program vers proto   port  service
		100000    4   tcp    111  portmapper
		100000    3   tcp    111  portmapper
		100000    2   tcp    111  portmapper
		100000    4   udp    111  portmapper
		100000    3   udp    111  portmapper
		100000    2   udp    111  portmapper
```

I didn't see rpc.statd or rpc.lockd, so the current NFS client cannot use locking. I will come
    back to this later, but will continue without locking


Running `rpcinfo -p <nfs_server_ip>` on my local machine, I got the following output from the NFS
    server:

```
    [ec2-user@ip-172-31-62-37 home]$ rpcinfo -p
       program vers proto   port  service
        100000    4   tcp    111  portmapper
        100000    3   tcp    111  portmapper
        100000    2   tcp    111  portmapper
        100000    4   udp    111  portmapper
        100000    3   udp    111  portmapper
        100000    2   udp    111  portmapper
        100024    1   udp  44103  status
        100024    1   tcp  33140  status
        100005    1   udp  20048  mountd
        100005    1   tcp  20048  mountd
        100005    2   udp  20048  mountd
        100005    2   tcp  20048  mountd
        100005    3   udp  20048  mountd
        100005    3   tcp  20048  mountd
        100003    3   tcp   2049  nfs
        100003    4   tcp   2049  nfs
        100227    3   tcp   2049  nfs_acl
        100003    3   udp   2049  nfs
        100003    4   udp   2049  nfs
        100227    3   udp   2049  nfs_acl
        100021    1   udp  53599  nlockmgr
        100021    3   udp  53599  nlockmgr
        100021    4   udp  53599  nlockmgr
        100021    1   tcp  34068  nlockmgr
        100021    3   tcp  34068  nlockmgr
        100021    4   tcp  34068  nlockmgr
```

When trying to mount directories onto localhost using `sudo mount -t nfs -o nolock <host>:</path/to/dir> <local_dir>`
`-t` specifies type of filesystem, which is nfs
`-o `specifies options, nolock is to not use locking since client does cannot use locking (yet)


e.g: sudo mount -t nfs -o nolock 54.210.0.137:/home/ec2-user/nfs_test_dir /mnt/nfs_test_dir
I get the following errror: `mount.nfs: Connection timed out`

Running with the -v option, I see:
```
	mount.nfs: timeout set for Mon Mar 20 19:14:07 2017
	mount.nfs: trying text-based options 'nolock,vers=4,addr=54.210.0.137,clientaddr=172.27.96.68'
	mount.nfs: mount(2): Operation not permitted
	mount.nfs: trying text-based options 'nolock,addr=54.210.0.137'
	mount.nfs: prog 100003, trying vers=3, prot=6
	mount.nfs: trying 54.210.0.137 prog 100003 vers 3 prot TCP port 2049
	mount.nfs: prog 100005, trying vers=3, prot=17
	mount.nfs: trying 54.210.0.137 prog 100005 vers 3 prot UDP port 200
```


The immediate gut reaction was to just to enable incoming UDP on port 20048. After checking around
on the internet, I found that it is best to systematically assign IPs to different nfs daemons

First, I added new rules to my instance's firewall, enabling inbound UDP and TCP to ports 32608-32610
In the file `/etc/sysconfig/nfs`, I added variables:
```
        RPCMOUNTDOPTS="-p 32808"
        MOUNTD_PORT=32808
        STATD_PORT=32806
        STATD_OUTGOING_PORT=32807
        LOCKD_TCPPORT=32809
        LOCKD_UDPPORT=32810
```

After adding these variables, run `exportfs -ra`

This requires changes to the /etc/exports options. The first is to allow `no_root_squash`. It seems
that mounting filesystems requires sudo access, unless there are some changes made to /etc/fstab
file. For now, I am adding `no_root_squash`. The second option that needs to be added is `insecure`. 
The option `secure` is automatically added, which requires requests originate from ports less than 
`IPPORT_RESERVED` (`1024`). Since we set high port values (>1024), we need to  set `insecure`.

After that, it seems to work.


# Mounting without `sudo` option



Links used:
http://nfs.sourceforge.net/nfs-howto/ar01s03.html
http://hunterford.me/amazon-ec2-and-nfs/
http://www.troubleshooters.com/linux/nfs.htm
http://serverfault.com/questions/629496/rpc-timeout-between-2-linux-servers
https://help.ubuntu.com/community/SettingUpNFSHowTo
https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Storage_Administration_Guide/nfs-serverconfig.html
https://wiki.debian.org/SecuringNFS

