# Mailserver

Simple mailserver that redirects mails to personal mails.

Sending mails: http://mailgun.com/

```bash
apt-get update

# Install postfix
# Internet Site -> Fully qualified domain name = "mydomain.com"
apt-get install postfix

# Set settings:
# myhostname = mydomain.com
# virtual_alias_maps = hash:/etc/postfix/virtual
# mynetworks = 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128
nano /etc/postfix/main.cf

# Add redirection mails
# support@mydomain.com personal.mail@some-provider.com
nano /etc/postfix/virtual

# Restart
postmap /etc/postfix/virtual
service postfix restart
```
