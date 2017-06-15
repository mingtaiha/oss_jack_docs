This guide talks about how to set up a Virtual Host

Sources: 

https://www.digitalocean.com/community/tutorials/how-to-set-up-apache-virtual-hosts-on-centos-7
https://www.digitalocean.com/community/tutorials/how-to-configure-the-apache-web-server-on-an-ubuntu-or-debian-vps

This assumes that you have sudo access (or are acting as root)

1) Create a directory to store the content for your website.
The DocumentRoot for web content is `/var/www/` on CentOS 7,
and /var/www/html is a default directory used to store web
content. So, we will make a folder under `/var/www/html` for 
that website.

`sudo mkdir -p /var/www/html/domainname.com/public_html

`public_html` is a default convention for naming folders whose
content can be hosted. We will put content in `public_html`
because we will also set up the Apache server to output
logs in `/var/www/html/domainname.com`; that is information
we do NOT want the user to access.

2) Since we created the directroy as `root`, we will now
change ownership of the directory to a user to enable 
regular users to modify the content of the webservice.

`sudo chown -R <user>:<user> /var/www/html/domainname.com/public_html`

We also want to modify the permissions to enable read and execute
access to the general web directory, and all of the files and
folders inside

`sudo chmod -R 755 /var/www`

3) Create A Page for a Virtual Host

For /var/www/html/domainname.com/public`
because we will also set up the Apache server to output
logs in `/var/www/html/domainname.com`; that is information
we do NOT want the user to access.

2) Since we created the directroy as `root`, we will now
change ownership of the directory to a user to enable 
regular users to modify the content of the webservice.

`sudo chown -R <user>:<user> /var/www/html/domainname.com/public_html`

3) We want to make an `index.html` page for each site that specifies
the domain. By convention, Apache will serve this page when a user
visits `www.domainname.com`.

`sudo vim /var/www/html/domainname.com/public_html/index.html`

`index.html` is a simple HTML document. In that document,
we will add
```
<html>
    <p>Welcome to my webpage</p>
</html>
```

4) We will now create new Virtual Host Files.
To start, we will set up the folders used to hold configuration
files (`*.conf`) for each website in the ServerRoot directory.
On CentOS 7, the ServerRoot is `/etc/httpd`.

In `etc/httpd`, we will make a `sites-available` directory. This
directory will keep all of the virtual host files. We will also
create a `sites-enabled` directory to hold symbolic links to 
virtual hosts that we want to publish. This will allow us to
easily enable and disable virtual hosts.

We will then tell Apache to look for the virtual hosts in the
`sites-enabled` directory. To do so, we will edit Apache's
main config file, `/etc/httpd/conf/httpd.conf`

At the bottom of the file, add the line
`IncludeOptional sites-enabled/*.conf`

Now, we can create a Virtual Host File. In the `sites-available`
folder, create the file `www.domainname.com.conf`. In the
file we will add:

```
<VirtualHost domainname.com:80>
    ServerName www.domainanme.com
    ServerAlias domainname.com
    DocumentRoot /var/www/html/domainname.com/public_html
    ErrorLog /var/www/html/domainname.com/error.log
    CustomLog /var/www/html/domainname.com/requests.log combined
    LogLevel warn
</VirtualHost>
```

NOTE: In the first line (namely `<VirtualHost domainname.com:80>`),
it is possible to put `<VirtualHost *:80>` However, I added
`domainname.com` because doing so allows Apache to serve content
on a per-domain-name basis; Apache can serve be used to host
several websites using the SAME IP. There are also ways to
configure based on IP. For a more detailed discussion, check
out these links:

`https://httpd.apache.org/docs/2.4/vhosts/name-based.html#namevip`
`https://httpd.apache.org/docs/2.4/vhosts/examples.html`


5) Now, we can enable the new virtual host files.
In the `sites-enabled` directory, we will create a symbolic link
to the example website created.

`sudo ln -s /etc/httpd/sites-available/www.domainname.com.conf /etc/ httpd/sites-enabled/www.domainname.com.conf`

6) Once you are done, restart Apache
`systemctl restart httpd`

And visit the webiste to test the results!
