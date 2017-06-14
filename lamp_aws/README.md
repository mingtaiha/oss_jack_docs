# Setting up a LAMP Stack

This guide assumes that the reader (you) have root access to the machine on which you want to install the LAMP stack.
LAMP stands for Linux-Apache-MySQL-PHP. The following guide is RHEL 7.5 (Linux). 

### Installing HTTP Server Program
To install httpd, run `yum install httpd`

Run `systemctl status|start|restart httpd`, `status` to check the status of httpd, `start` or `restart` to start httpd
The above command is used to used to restart Apache server so website `http://<server_ip>` has up-to-date changes

Changed AWS Security Group Rules (read Firewall) to allow HTTP communication on port 80

In `/etc/httpd/conf.d/welcome.conf`, change `Options -Indexes` to `Option +Indexes` to see directory list,
By accessing `http://<server_ip>`, I will see a directory listing of `/var/www/html`. Since we are setting this up
for the first time, we should see an empty directory tree, i.e. no files stored in `/var/www/html`

### Installing PHP
To install PHP, install the following commands
`yum install php`
`yum install php-mysql`
`yum install php-pdo`
`yum install php-gd`

As a test file, run `echo "<?php phpinfo(); ?>" > /var/www/html/info.php`. After restarting `httpd` service, 
you will see a file, `info.php` from the directory `/var/www/html`. Without the above command, I will see not 
see `info.php` when accessing the website. 

The directory tree was set to the correct timezone for me (since I can see when `info.php` was last changed 
aka created). If this is not correct, you can change the timezone. To set the timezone, go into the file
`/etc/php.ini`. In the `[Date]` section, set `date.timezone = <Continent>/<City>` to define timezone. The 
supported timezones can be found [here](http://php.net/manual/en/timezones.php) 

### Installing MySQL
I was recommended to install MariaDB, which is backwards-compatible with MySQL. MariaDB is a fork of MySQL intended to
remain free under GNU GPL. This was because of Oracle's acquisition of MySQL. I get the feeling that Oracle is a 
terrible corporation, but what does this n00b know?

To install MariaDB, run `yum install mariadb-server mariadb`

To start MariaDB, run `systemctl start mysqld` to start mariadb

When running mysql_secure_installation, I get the following error:
```
mysqladmin: connect to server at 'localhost' failed error:
'Access denied for user 'root'@'localhost' (using password: YES)'
```

To overcome this, I did the following (according to 
[this](http://stackoverflow.com/questions/41645309/mysql-error-access-denied-for-user-rootlocalhost) guide).
As it turns out this, this method is quite insecure, but it was (unforunately) the first and most common solution
that I saw. The MySQL documentation lists other, safer [ways](https://dev.mysql.com/doc/refman/5.7/en/resetting-permissions.html) 
of solving the same problem for you to try if you are feeling like a 
[`G O O D B O Y`](http://s2.quickmeme.com/img/bc/bc597d2eb87ba74ab6844f9c3794c6b0eb11815047d003eb34747619791bc088.jpg).

- In /etc/my.cnf, insert `--skip-grant-tables` and `--skip-networking` under `[mysqld]`. `--skip-grant-tables` allows anyone to
log in from anywhere and do anything to the database. This is a very insecure way of doing things, yet it also seems more common.
Add the `--skip-networking`option to allow connections from `localhost`
- Then restart mysqld: `systemctl restart mysqld`
- Run `mysql -u root -p` to set password. When prompted for a password, just press enter since there is no password configured yet
- Run `flush privileges;` in mysql prompt `mysqld>`
- Set a new password by `ALTER USER 'root'@'localhost' IDENTIFIED BY '<new_password>';`
- Remove `--skip-grant-tables` and `--skip-networking` from /etc/my.cnf`. **This step is important**
- Restart mysqld

Now you can run `mysql -u root -p` as root. Run `mysql> show databases;`, and expect to get something like the following:
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                |
+--------------------+
4 rows in set (0.00 sec)
```

**NOTE** It was also recommended that I install phpmyadmin. However, it is not necessary and I would rather practice SQL syntax by
doing things via the mysql prompt.


I added the following lines to `/etc/rc.local` so that `mysqld` and  `httpd` start when AWS instance starts up when the AWS
instance is started.

### Sources
http://www.tecmint.com/install-lamp-in-centos-7/
http://stackoverflow.com/questions/41645309/mysql-error-access-denied-for-user-rootlocalhost


## Setting up SSL Certificate


# TODO: SET UP A WEB DIRECTORY
See guide for next time: http://howtolamp.com/lamp/httpd/2.4/customizing/per-user-web-directories/
