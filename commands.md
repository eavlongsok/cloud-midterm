cloud midterm

# CREATE 3 DROPLETS, 1 APP, 2 DB
# -- app --
```
sudo apt update
```
```
sudo apt install apache2
```
```
apt install php7.4 php7.4-cli php-mysql mysql-client
```
```
apt install composer
```
```
systemctl restart apache2
```

```
cd /var/www/html
git clone https://github.com/eavlongsok/ShopMe-admin.git
cd ShopMe-admin
mv * ../
mv .[!.]* ../ 
cd ..
rm -r ShopMe-admin
```
```
apt install php7.4-curl php7.4-dom
```
```
composer i
```

```
sudo chown -R $USER:www-data storage
sudo chown -R $USER:www-data bootstrap/cache
chmod -R 775 storage
chmod -R 775 bootstrap/cache

sudo a2enmod rewrite
```

## EDIT apache2.conf
```
nano /etc/apache2/apache2.conf
```
```
<Directory /var/www/>
        Options Indexes FollowSymLinks
        AllowOverride All
        Require all granted
</Directory>
```

## EDIT the root route
```
nano /etc/apache2/sites-available/000-default.conf
```
```
nano resources/views/home.blade.php
```
```
nano resources/views/login.blade.php
```

```
<?php
  echo "<br/><br/><br/><h1>ServerName:".gethostname()."</h1><br>";
  echo "<h1>ServerIP:".$_SERVER['SERVER_ADDR']."</h1>";
?>
```

```
systemctl restart apache2
```


# -- db --
## -- installation and configuration on both servers
```
apt-get update
```
```
apt-get install mysql-server -y
```

```
mysql_secure_installation
y
0
y
y
y
y
```

```
nano /etc/mysql/mysql.conf.d/mysqld.cnf
```
```
add read-only = 1 under [mysqld] if slave
bind-address 0.0.0.0
uncomment server-id, log_bin (change server-id to 2 if slave)
```

```
sudo systemctl restart mysql
```

## -- master 
```
CREATE USER 'replication_user'@'%' IDENTIFIED BY 'HelloWorld-123';
GRANT REPLICATION SLAVE ON *.* TO 'replication_user'@'%';
FLUSH PRIVILEGES;
SHOW MASTER STATUS\G

CREATE DATABASE shopme;
CREATE USER 'eavlongsok'@'%' IDENTIFIED WITH mysql_native_password BY 'HelloWorld-123';
GRANT ALL PRIVILEGES ON shopme.* TO 'eavlongsok'@'%' WITH GRANT OPTION;
FLUSH PRIVILEGES;
```

## -- slave
```
STOP SLAVE;
```

```
CHANGE MASTER TO MASTER_HOST ='MasterIP', MASTER_USER ='replication_user', MASTER_PASSWORD ='HelloWorld-123', MASTER_LOG_FILE = 'mysql-bin.000001', MASTER_LOG_POS = Position;
```

```
CHANGE MASTER TO get_master_public_key=1;
START SLAVE;
SHOW SLAVE STATUS\G
```

## -- master
on navicat, execute sql on master

# -- app --
nano config/database.php
```
'read' => [
  'host' => [
    'db-ro-mysql-sgp1-slave-do-user-11155144-0.c.db.ondigitalocean.com',    
  ],
],
'write' => [
  'host' => [
    'db-mysql8-sgp1-do-user-11155144-0.c.db.ondigitalocean.com',
  ],
],
```


# -- digitalocean --
SNAPSHOT APP
CREATE 2 MORE DROPLETS BASED ON SNAPSHOT

APPLY TAGS TO ALL DROPLETS
CREATE LOAD BALANCER (STICKY COOKIE off, BACKEND KEEPALIVE on)

CLICK ON DROPLET, GO TO NETWORKING, AND EDIT FIREWALL
CREATE FIREWALL

APP SERVER GROUP
  Inbound: SSH, HTTP, HTTPS
  Outbound: MySQL

  TEST BY TURNING ON AND OFF MYSQL AND HTTP

DATABASE SERVER GROUP
  Inbound: SSH, MySQL
  Outbound: MySQL

  TEST BY TURNING OFF MYSQL


### .env
```
APP_NAME=Laravel
APP_ENV=local
APP_KEY=base64:VcNfGcZCnQYlF2SzWv7EkzQZUUWj7Wj0TUd4vP9rh9E=
APP_DEBUG=true
APP_URL=http://localhost

LOG_CHANNEL=stack

DB_CONNECTION=mysql
DB_HOST=db-shopme-do-user-14738775-0.b.db.ondigitalocean.com
DB_PORT=25060
DB_DATABASE=shopme
DB_USERNAME=doadmin
DB_PASSWORD=AVNS_p_CckoejvlTyBIn_sDl

BROADCAST_DRIVER=log
CACHE_DRIVER=file
QUEUE_CONNECTION=sync
SESSION_DRIVER=file
SESSION_LIFETIME=120

REDIS_HOST=127.0.0.1
REDIS_PASSWORD=null
REDIS_PORT=6379

MAIL_MAILER=smtp
MAIL_HOST=smtp.mailtrap.io
MAIL_PORT=2525
MAIL_USERNAME=null
MAIL_PASSWORD=null
MAIL_ENCRYPTION=null
MAIL_FROM_ADDRESS=null
MAIL_FROM_NAME="${APP_NAME}"

AWS_ACCESS_KEY_ID=
AWS_SECRET_ACCESS_KEY=
AWS_DEFAULT_REGION=us-east-1
AWS_BUCKET=

PUSHER_APP_ID=
PUSHER_APP_KEY=
PUSHER_APP_SECRET=
PUSHER_APP_CLUSTER=mt1

MIX_PUSHER_APP_KEY="${PUSHER_APP_KEY}"
MIX_PUSHER_APP_CLUSTER="${PUSHER_APP_CLUSTER}"

ALGOLIA_APP_ID = QWNBWAQTEH
ALGOLIA_SECRET = d69f95548f9d7d31523e5e8fb92981bd

IMGUR_CLIENT_ID = 1c440ffc1e8bfd2
IMGUR_CLIENT_SECRET = a6ae219917b10b27c37f67d4b8df6bea71d884ce
```
