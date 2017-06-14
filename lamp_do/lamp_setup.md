This shows how to set up a LAMP stack. LAMP stands for Linux, 
Apache Server, MySQL, PHP.

This guide assumes that you have root (or `sudo`) access

Useful Sources:
https://www.digitalocean.com/community/tutorials/how-to-install-linux-apache-mysql-php-lamp-stack-on-centos-6

https://www.tecmint.com/install-lamp-in-centos-7/

The LAMP stack was set up on a DigitalOcean VM with 1 Core,
1 GB RAM, 20 GB SSD storage, 2TB data transfer

L - The Linux operating system chosen is CentOS 7. CentOS was
chosen because RHEL was not supported by DigitalOcean, and
is a standard OS for system administration

A - Apache HTTP Server
1) First, install Apache HTTP Server with `yum install httpd`
2) After that, start the Apache server with `service httpd start`
3) Enable HTTP traffic through the firewall with
    `firewall-cmd --add-service=http`. To do so permanently,
    `firewall-cmd --permanent --add-service=http`. This only
    takes effect when the firewall service is restarted

Note: At this point, I decided to enable TCP traffic on port
5000 in case the app will run on a webserver. I did so by
running `firewall-cmd --add-port=5000/tcp`

To show the firewall rules, run `iptables -S`

For more firewall and security notes:
https://www.digitalocean.com/community/tutorials/how-to-set-up-a-firewall-using-firewalld-on-centos-7
https://www.digitalocean.com/community/tutorials/how-to-migrate-from-firewalld-to-iptables-on-centos-7

Testing the setup: In the file `etc/httpd/conf.d/welcome.conf`
Enable Options in the LocationMatch element. Specifically,
`Options -Indexes` to `Options +Indexes`

To see changes to the webserver, you need to restart the
Apache service `service httpd (re)start`. Then, visit the
IP address in a browser

M - MariaDB (a drop-in that is fully compatible with MySQL)
Install with `yum install mariadb

Run `/usr/bin/mysql_secure_installation` to set up the
database. You will have to set a password. When presented
with options, I pressed (y).

Useful Source: https://www.digitalocean.com/community/tutorials/how-to-install-mariadb-on-centos-7


P - PHP (It really doesn't have to be PHP. Python can also
be used)
Install with `yum install php php-mysql php-pdo php-gd php-mbstring`

Testing PHP installation: 
Run `echo "<?php phpinfo(); ?>" > /var/www/html/info.php`
and restart httpd to see that PHP works with the webserver


Now your LAMP Stack is installed!
