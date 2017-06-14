This assumes that you have a LAMP stack (or equivalent) setup.
The instructions are specific to Apache and CentOS. See
https://certbot.eff.org/ for more instructions.

Here's the User Guide to CertBot. Useful for knowing where
the keys and certs are stored, etc ...
https://certbot.eff.org/docs/using.html#certbot-commands

This step also requires that you have a domain name, and have
associated the domain name with the IP hosting your webserver.

While it is possible to use only the IP, it is not recommended.
Certbot won't let you use an IP. For some more intro reading,
give these a look-see:
https://stackoverflow.com/questions/2043617/is-it-possible-to-have-ssl-certificate-for-ip-address-not-domain-name
https://timnash.co.uk/guessing-ssl-questions/

1) Download and Install the Certbot
`wget https://dl.eff.org/certbot-auto`
`chmod a+x certbot-auto`

2) Run the Certbot
`./path/to/certbot-auto --apache`

This part will ask for the domain name you would like to
certify. Alternately, you can also specify the domain
name for which you want to get a certificate. Use the
`-d` option per domain name. E.g.

`./path/to/certbot-auto --apache -d www.dname1.com -d www.dname2.com`

NOTE: If you did not set up a VirtualHost with your given
ServerName, you need to set one up. See the guide on hosting
a domain name in order to set one up

If successful, You now have a certicate. However, 
certificates expire and you will have to periodically 
renew your certificate. The location of the certs can
be found in the CertBot User Docs.

3) Renew certificate
`./path/to/certbot-auto renew`

Certificate renewals do not require the need to enter the
domain name

A cron job can be set up so that certificates are renewed
automatically. It is recommended by Certbot documentation
to renew twice a day.

Here's a nice little guide for cron:
https://www.centos.org/docs/5/html/Deployment_Guide-en-US/ch-autotasks.html


TODO: A nice extension of the guide would be something that
can renew multiple domains at once, or creating a single
server to manage all SSL certs (if possible)
