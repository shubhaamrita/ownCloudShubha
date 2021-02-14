---
layout: page
title: "Manual Installation"
permalink: /installation/
---

## Overview
-----
 This guide will walk you through the installation and configuration of ownCloud in both environments:
- Ubuntu 18.04
- Ubuntu 20.04

## Pre-requisites
-----
- A fresh installation of **Ubuntu 18.04/20.04** with SSH enabled
- Connected as the root user
- ownCloud directory is located in ```/var/www/owncloud/```

## Preparation
-----
Run the below command to:
- Ensure that all the installed packages are entirely up to date
- PHP is available in the APT repository

```apt update && \ apt upgrade -y```

### Create the occ Helper Script
- Create a helper script to simplify running occ commands
```
FILE="/usr/local/bin/occ"
/bin/cat <<EOM >$FILE
#! /bin/bash
cd /var/www/owncloud
sudo -u www-data /usr/bin/php /var/www/owncloud/occ "\$@"
EOM
```

- Make the helper script executable
```
chmod +x /usr/local/bin/occ
```

### Install the Required Packages
```
apt install -y \
  apache2 \
  libapache2-mod-php \
  mariadb-server \
  openssl \
  php-imagick php-common php-curl \
  php-gd php-imap php-intl \
  php-json php-mbstring php-mysql \
  php-ssh2 php-xml php-zip \
  php-apcu php-redis redis-server \
  wget
  ```

### Install the Recommended Packages
#### Ubuntu 18.04
  ```
  apt install -y \
  ssh bzip2 sudo cron rsync curl jq \
  inetutils-ping smbclient php-libsmbclient \
  php-smbclient coreutils php-ldap
```

#### Ubuntu 20.04
- Add ondrej’s ppa
```
sudo add-apt-repository ppa:ondrej/php
sudo apt-get update
echo "deb http://ppa.launchpad.net/ondrej/php/ubuntu $(lsb_release -cs) main" | tee /etc/apt/sources.list.d/php.list
```
- Add the key
```
apt-key adv --recv-keys --keyserver hkps://keyserver.ubuntu.com:443 4F4EA0AAE5267A6C
```
- Install the recommended packages
```
apt install -y \
  ssh bzip2 rsync curl jq \
  inetutils-ping smbclient\
  php-smbclient coreutils php-ldap
```

## Installation
-----
### Configure Apache
- Change the Document Root
```sh
sed -i "s#html#owncloud#" /etc/apache2/sites-available/000-default.conf
service apache2 restart
```
- Create a Virtual Host Configuration

```
FILE="/etc/apache2/sites-available/owncloud.conf"
/bin/cat <<EOM >$FILE
Alias /owncloud "/var/www/owncloud/"
<Directory /var/www/owncloud/>
  Options +FollowSymlinks
  AllowOverride All
 <IfModule mod_dav.c>
  Dav off
 </IfModule>
 SetEnv HOME /var/www/owncloud
 SetEnv HTTP_HOME /var/www/owncloud
</Directory>
EOM
```
- Enable the Virtual Host Configuration
```
a2ensite owncloud.conf
service apache2 reload
```

### Configure the Database
```
service mysql start
mysql -u root -e "CREATE DATABASE IF NOT EXISTS owncloud; \
GRANT ALL PRIVILEGES ON owncloud.* \
  TO owncloud@localhost \
  IDENTIFIED BY 'password'";
```

- Enable the Recommended Apache Modules
```
echo "Enabling Apache Modules"
a2enmod dir env headers mime rewrite setenvif
service apache2 reload
```

### Download ownCloud
```
cd /var/www/
wget https://download.owncloud.org/community/owncloud-10.6.0.tar.bz2 && \
tar -xjf owncloud-10.6.0.tar.bz2 && \
chown -R www-data. owncloud
```

### Install ownCloud
```
occ maintenance:install \
    --database "mysql" \
    --database-name "owncloud" \
    --database-user "owncloud" \
    --database-pass "password" \
    --admin-user "admin" \
    --admin-pass "admin"
```
### Configure ownCloud’s Trusted Domains
```
myip=$(hostname -I|cut -f1 -d ' ')
occ config:system:set trusted_domains 1 --value="$myip"
```
### Set Up a Cron Job
- Set your background job mode to cron
```
occ background:cron
echo "*/15  *  *  *  * /var/www/owncloud/occ system:cron" \
  > /var/spool/cron/crontabs/www-data
chown www-data.crontab /var/spool/cron/crontabs/www-data
chmod 0600 /var/spool/cron/crontabs/www-data
```
If you need to sync your users from an LDAP or Active Directory Server, add this additional [Cron job](https://doc.owncloud.com/server/10.6/admin_manual/configuration/server/background_jobs_configuration.html). Every 15 minutes this cron job will sync LDAP users in ownCloud and disable the ones who are not available for ownCloud. Additionally, you get a log file in ```/var/log/ldap-sync/user-sync.log``` for debugging.
```
echo "*/15 * * * * /var/www/owncloud/occ user:sync 'OCA\User_LDAP\User_Proxy' -m disable -vvv >> /var/log/ldap-sync/user-sync.log 2>&1" > /var/spool/cron/crontabs/www-data
chown www-data.crontab  /var/spool/cron/crontabs/www-data
chmod 0600  /var/spool/cron/crontabs/www-data
mkdir -p /var/log/ldap-sync
touch /var/log/ldap-sync/user-sync.log
chown www-data. /var/log/ldap-sync/user-sync.log
```

### Configure Caching and File Locking
```
occ config:system:set \
   memcache.local \
   --value '\OC\Memcache\APCu'

occ config:system:set \
   memcache.locking \
   --value '\OC\Memcache\Redis'

service redis-server start

occ config:system:set \
   redis \
   --value '{"host": "127.0.0.1", "port": "6379"}' \
   --type json
```
### Configure Log Rotation
```
FILE="/etc/logrotate.d/owncloud"
sudo /bin/cat <<EOM >$FILE
/var/www/owncloud/data/owncloud.log {
  size 10M
  rotate 12
  copytruncate
  missingok
  compress
  compresscmd /bin/gzip
}
EOM
```
## Finalise the Installation
-----
Make sure the permissions are correct
```
cd /var/www/
chown -R www-data. owncloud
```

**ownCloud is now installed. You can confirm that it is ready to use by pointing your web browser to your ownCloud installation**

