# owncloud

https://websiteforstudents.com/install-owncloud-on-ubuntu-16-04-lts-with-apache2-mariadb-and-php-7-1-support/

https://docs.bitnami.com/oci/apps/owncloud/configuration/configure-fail2ban/

Android applications: 

* https://play.google.com/store/apps/details?id=org.dmfs.carddav.sync
* https://play.google.com/store/apps/details?id=com.ocloud24.android

* There are 9 Steps

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

## Step 4: Create OwnCloud Database

```bash
sudo mysql -u root -p
CREATE DATABASE owncloud;
CREATE USER 'ownclouduser'@'localhost' IDENTIFIED BY 'new_password_here';
GRANT ALL ON owncloud.* TO 'ownclouduser'@'localhost' IDENTIFIED BY 'user_password_here' WITH GRANT OPTION;
FLUSH PRIVILEGES;
EXIT;
```

## Step 5: Download OwnCloud Latest Release

```bash
cd /tmp && wget https://download.owncloud.org/community/owncloud-10.0.3.zip
unzip owncloud-10.0.3.zip
sudo mv owncloud /var/www/html/owncloud/
```

### Set permissions

```bash
sudo chown -R www-data:www-data /var/www/html/owncloud/
sudo chmod -R 755 /var/www/html/owncloud/
```

## Step 6: Generate the certificate and the private key for your domain myexample.com

* See [https://github.com/davidboukari/ssl]

## Step 7: Configure Apache2

```bash
<VirtualHost *:443>
     ServerAdmin admin@myexample.com
     DocumentRoot /var/www/html/owncloud/
     ServerName myexample.com
     ServerAlias www.myexample.com
  
     Alias /owncloud "/var/www/html/owncloud/"

     <Directory /var/www/html/owncloud/>
        Options +FollowSymlinks
        AllowOverride All
        Require all granted
          <IfModule mod_dav.c>
            Dav off
          </IfModule>
        SetEnv HOME /var/www/html/owncloud
        SetEnv HTTP_HOME /var/www/html/owncloud
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/error.log
     CustomLog ${APACHE_LOG_DIR}/access.log combined

SSLCertificateFile /etc/ssl/certs/myexample.com.crt
SSLCertificateKeyFile /etc/ssl/private/myexample.com.key
</VirtualHost>
```

```bash
systemctl restart apache2
```
## Step 8: Tests

Check the url for your domain name
* [https://www.myexample.com]


## Step 9: Activate Fail2ban rules to block access to bruteforce attempts

* See [https://github.com/davidboukari/fail2ban

Create the /etc/fail2ban/filter.d/owncloud.conf:

```bash
[Definition]
failregex={"reqId":".*","level":2,"time":".*","remoteAddr":".*","user":".*","app":"core","method":"POST","url":".*","message":"Login failed: '.*' \(Remote IP: '<HOST>'\)"}
ignoreregex =
```

Copy the /etc/fail2ban/jail.conf file to the /etc/fail2ban/jail.local file and add the code below:

```bash
#OwnCloud
[owncloud]
enabled  = true
filter   = owncloud
#action = iptables-multiport[name=owncloud, port="http,https"]
port="http,https"
logpath  = /var/www/html/owncloud/data/owncloud.log
maxretry = 5
findtime = 600
bantime = 600
```

Check the rules
```bash
sudo fail2ban-regex /opt/bitnami/apps/owncloud/data/owncloud.log /etc/fail2ban/filter.d/owncloud.conf
```

systemctl restart fail2ban


## The plugins

To add somes plugins connect in admin account => go to settings => Application activate / Deactivate
The plugins become availaible in the menu in left top corner

Check the list of app availaible
```bash
sudo -u www-data /usr/bin/php occ app:list
```

## Market place

* Connect you as admin
* To use the market place just click to the left top corner and select market place.
You can use it to install Contact, Agenda, ... 

## Agenda

* Search somes applications in android "davdroid" compatible with caldav
* Enter the url https://myserverurl/remote.php/dav/


## Synchronize the contacts from the phone

 * See https://code.adonline.id.au/sync-phone-contacts-via-owncloud/

Export your contacts under android phones by using Contact application then export them to .vcf file
share this file with your application ocloud to send it then download it to your computer.

<p>
 In the phone with Playstore => Install CardDav-Sync free
 Add cardDav account => the url to enter is  https://yourwebsite.com/remote.php/carddav/addressbooks/yourname/contacts and setup the login / password
<p>

Use the button parameter at the left bottom and select import

* Troubleshooting contacts

*Seuls vCard version 4.0 (RFC6350) ou version 3.0 (RFC2426) sont pris en charge.*
You can use: https://github.com/jowave/vcard2to3.git

```bash
cd vcard2to3
./vcard2to3.py Contacts.vcf
```
