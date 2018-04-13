# Jelastic test

### Update packages
```bash
yum update -y
```

### LEMP installation

#### MySQL
```bash 
yum localinstall https://dev.mysql.com/get/mysql57-community-release-el7-11.noarch.rpm
service mysql start 

#Get temporary generated psswd for login
grep 'A temporary password is generated for root@localhost' /var/log/mysqld.log |tail -1

#Login using psswd you get
mysql -u root -p

#Change psswd
SET PASSWORD = PASSWORD('new_psswd')

```
##### Create user and DB
```bash
#run this commands in mysql console
CREATE DATABASE wordpressdb;

CREATE USER 'jwp2018'@'localhost' IDENTIFIED BY 'jwp2018';

GRANT ALL ON wordpressdb.* TO 'jwp2018'@'localhost';

FLUSH PRIVILEGES;
```

#### Nginx with php-fpm

Create file `/etc/yum.repos.d/nginx.repo` and add following content to repo file:
```
[nginx]
name=nginx repo
baseurl=http://nginx.org/packages/rhel/$releasever/$basearch/
gpgcheck=0
enabled=1
```

Then run following commands to install:
```bash
yum --enablerepo=remi,remi-php72 install nginx php-fpm php-common

service httpd stop

service nginx start
service php-fpm start
```
Dissable httpd autostart on boot and enable nginx/php-fpm:
```bash
systemctl disable httpd.service
systemctl enable nginx.service
systemctl enable php-fpm.service
```
##### Configure 
```bash
vi /etc/php-fpm.d/www.conf

## Change following ##
;listen = /run/php-fpm/www.sock
listen = 127.0.0.1:9000
```

Copy [config](nginx/wp.conf) to `/etc/nginx/conf.d/`
Then delete default.conf:   
```bash
rm /etc/nginx/conf.d/default.conf
```
Create your white list:
```bash
vi /etc/ngninx/whitelist

# This should look like 
allow your_ip;
```


### Iptables port 80 to port 8080 redirect

```bash
iptables -t nat -A PREROUTING -i venet0 -p tcp --dport 80 -j REDIRECT --to-port 8080
iptables -t nat -A PREROUTING -i venet0:0 -p tcp --dport 80 -j REDIRECT --to-port 8080

iptables -t nat -I PREROUTING --src 24 --dst 127.0.0.1 -p tcp --dport 80 -j REDIRECT --to-ports 8080
iptables -t nat -I PREROUTING --src 24 --dst 127.0.0.1 -p tcp --dport 80 -j REDIRECT --to-ports 8080

iptables-save
service iptables reload
```
### Create user
```bash
useradd -m -d /var/www/jelastictest jelastictest
passwd jelastictest
```

### Wordpress
 Run following to install:
```bash
cd /var/www/jelastictest/
wget http://wordpress.org/latest.tar.gz
tar -xvf latest.tar.gz

chown -R nginx:nginx *

mv wp-config-sample.php wp-config.php

vi wp-config.php
```
And make following changes:
```
/** The name of the database for WordPress */

define(‘DB_NAME’, ‘wordpressdb’);

/** MySQL database username */

define(‘DB_USER’, ‘jwp2018’);

/** MySQL database password */

define(‘DB_PASSWORD’, ‘jwp2018‘);

```
