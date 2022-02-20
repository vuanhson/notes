# Config proxmox to send email via SMTP server

## Preparing
- Proxmox Virtual Environment (tested with v6 and v7)
- SMTP credentials
  - Email domain
  - Username
  - Password
  - SMTP server address/port

## Guide
### Edit config file
Open proxmox Shell, then open `/etc/postfix/main.cf` with text editor
```
nano /etc/postfix/main.cf
```

Edit myhostname to your mail domain and relayhost with smtp provider server and port

```
# your email domain, e.g. mail.abc.com
myhostname=
# SMTP server information with [smtp-address]:port format. e.g.: [smtp.sendgrid.net]:465
relayhost =
```

Add following smtp configuration
```
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_security_options = noanonymous
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_tls_CAfile = /etc/ssl/certs/ca-certificates.crt
smtp_tls_wrappermode = yes
smtp_tls_security_level = encrypt
```
Save file

### Create credential file

```
nano `/etc/postfix/sasl_passwd`
```

with following content
```
# SMTP credential 
# E.g. [smtp.sendgrid.net]:465 apikey:SG.someactualkey

[smtp-address]:port username:passwordorapikey
```

### Apply config and install module
Apply config
```
chmod 600 /etc/postfix/sasl_passwd
postmap /etc/postfix/sasl_passwd
```
Install libsasl2-module
```
apt-get update
apt-get install libsasl2-modules
```
Restart service
```
systemctl restart postfix.service
systemctl status postfix.service
```

### Test
Test using mail command
```
echo "Test mail from postfix" | mail -s "Test Postfix" hello@mail.abc.com
```
Test using pvemailforward (mail will be send to email address set in proxmox when installing)
```
echo "test" | /usr/bin/pvemailforward
```

### Log path for debug
```
/var/log/mail.warn
/var/log/mail.info
```
