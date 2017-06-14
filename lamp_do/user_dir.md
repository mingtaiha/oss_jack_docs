This document will show you how to setup user directories
`https://httpd.apache.org/docs/2.4/howto/public_html.html`

1) Check the `userdir.conf` file. On CentOS 7, the path to the file
is `/etc/httpd/conf.d/userdir.conf`. By default, all lines of the
file is commented out (the first character of each line is `#`).

Not counting the comments, the file should look like:
```
<IfModule mod_userdir.c>
    UserDir disabled root
    UserDir public_html
</IfModule>
```

The `mod_userdir.c` enables user directories. `UserDir disabled root`
disables user directories by default. Otherwise, it may be able
to confirm the presence of a user in the system (depending on the
home directory permissions of the user. The `UserDir public_html`
allows enables requests to `www.domainname.com/~username` to go
to the user's `public_html`

2) Next, we have to set up the directories which we want to share,
and the sharing permissions. In the `userdir.conf` file, or in
VirtualHost element associated with your domain name, include the
following:

```
<Directory "/home/*/public_html">
    AllowOverride FileInfo AuthConfig Limit Indexes
    Options MultiViews Indexes SymLinksIfOwnerMatch IncludesNoExec
    Require method GET POST OPTIONS
</Directory>
```

`<Directory "/home/*/public_html">` is a directive regarding the
`public_html` folders in the `home` directory of every user 
(`*`, wildcard). 

This configuration allows read-only access to the user's `public_html`
directory
