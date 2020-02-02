# oncloud

https://websiteforstudents.com/install-owncloud-on-ubuntu-16-04-lts-with-apache2-mariadb-and-php-7-1-support/


https://docs.bitnami.com/oci/apps/owncloud/configuration/configure-fail2ban/


## Fail2ban rules

Create the /etc/fail2ban/filter.d/owncloud.conf file with the following code:
```bash
[Definition]
failregex={"reqId":".*","remoteAddr":".*","app":"core","message":"Login failed: '.*' \(Remote IP: '<HOST>\)","level":2,"time":".*"}
ignoreregex =
```

Copy the /etc/fail2ban/jail.conf file to the /etc/fail2ban/jail.local file and add the code below:

```bash
#OwnCloud
[owncloud]
enabled  = true
filter   = owncloud
action = iptables-multiport[name=owncloud, port="http,https"]
logpath  = /var/www/html/owncloud/data/owncloud.log
maxretry = 5
findtime = 600
bantime = 600
```

Check the rule
```bash
sudo fail2ban-regex /opt/bitnami/apps/owncloud/data/owncloud.log /etc/fail2ban/filter.d/owncloud.conf
```

systemctl restart fail2ban
