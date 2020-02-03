# oncloud

https://websiteforstudents.com/install-owncloud-on-ubuntu-16-04-lts-with-apache2-mariadb-and-php-7-1-support/

https://docs.bitnami.com/oci/apps/owncloud/configuration/configure-fail2ban/

Follow the 9 steps

## Step 1: Install Apache2

```bash
sudo apt install apache2
sudo sed -i "s/Options Indexes FollowSymLinks/Options FollowSymLinks/" /etc/apache2/apache2.conf
sudo systemctl stop apache2.service
sudo systemctl start apache2.service
sudo systemctl enable apache2.service
```

## Step 2: Install MariaDB

```bash
sudo apt-get install mariadb-server mariadb-client
sudo systemctl stop mysql.service
sudo systemctl start mysql.service
sudo systemctl enable mysql.service
sudo mysql_secure_installation
```
<p>
When prompted, answer the questions below by following the guide.

    Enter current password for root (enter for none): Just press the Enter
    Set root password? [Y/n]: Y
    New password: Enter password
    Re-enter new password: Repeat password
    Remove anonymous users? [Y/n]: Y
    Disallow root login remotely? [Y/n]: Y
    Remove test database and access to it? [Y/n]:  Y
    Reload privilege tables now? [Y/n]:  Y

```bash
sudo systemctl restart mysql.service
```

## Step 3: Install PHP and Related Modules

```bash
sudo apt-get install software-properties-common
sudo add-apt-repository ppa:ondrej/php
PHP_VERSION=7.0
sudo apt install php${PHP_VERSION} libapache2-mod-php${PHP_VERSION} php${PHP_VERSION}-common libapache2-mod-php${PHP_VERSION} php${PHP_VERSION}-mbstring php${PHP_VERSION}-xmlrpc php${PHP_VERSION}-soap php${PHP_VERSION}-ldap php${PHP_VERSION}-gd php${PHP_VERSION}-xml php${PHP_VERSION}-intl php${PHP_VERSION}-json php${PHP_VERSION}-mysql php${PHP_VERSION}-cli php${PHP_VERSION}-mcrypt php${PHP_VERSION}-ldap php${PHP_VERSION}-zip php${PHP_VERSION}-curl
```

Update /etc/php/${PHP_VERSION}/apache2/php.ini

```bash
file_uploads = On
allow_url_fopen = On
memory_limit = 256M
upload_max_filesize = 64M
max_execution_time = 360
date.timezone = America/Chicago
```


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
