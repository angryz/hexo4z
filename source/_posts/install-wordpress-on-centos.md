title: 在 CentOS 上安装 Wordpress
date: 2013-08-11 10:48:00
tags: [tools,linux]
categories: System
---

I'm goona host WordPress on Apache Service. Now let's do this.

## Install Dependencies
Apache2, MySQL, OpenSSL are already installed before.

```bash
sudo yum -y install php php-mysql
sudo yum -y install httpd-manual mod_ssl mod_perl mod_auth_mysql
sudo yum -y install php-gd php-xml php-mbstring php-ldap php-pear php-xmlrpc
sudo yum -y install mysql-connector-odbc mysql-devel libdbi-dbd-mysql
```

## Get latest edition of WordPress (Simplified Chinese Version)
From [this page](http://cn.wordpress.org/switching/). (You may need to cross the GFW to see it.)

```bash
cd ~/downloads/
wget http://cn.wordpress.org/wordpress-3.6-zh_CN.tar.gz
tar zxvf wordpress-3.6-zh_CN.tar.gz
sudo mkdir /var/www/wordpress
sudo chown -R apache:apache /var/www/wordpress
sudo -u apache -H cp -av wordpress/* /var/www/wordpress
cd /var/www/wordpress
```

## Create DB for WordPress

```bash
mysql -u root -p
mysql> CREATE DATABASE wordpress DEFAULT CHARACTER SET 'utf8';
mysql> CREATE USER 'wordpress'@'localhost' IDENTIFIED BY 'wordpress';
mysql> GRANT ALL PRIVILEGES ON wordpress.* TO 'wordpress'@'localhost';
```
<!--more-->

## Configure WordPress

```bash
sudo -u apache -H cp wp-config-sample.php wp-config.php 
sudo -u apache -H vim wp-config.php
```

Edit `wp-config.php` and modify these values:

```php
/* The name of the database for WordPress */
define('DB_NAME', 'wordpress');

/* MySQL database username */
define('DB_USER', 'wordpress');

/* MySQL database password */
define('DB_PASSWORD', 'wordpress');

/* Authentication Unique Keys and Salts. */
define('AUTH_KEY',         'H>(4Ki6SfQkfgvd8]Mrvrb3l[SAG}u4 Z;y+oQyIkCYNr[=6!jGApkt|U56JyrlJ');
define('SECURE_AUTH_KEY',  'Ap9,x @{r35Z/|F5+z.m+oJBVeM:+H{S|lAq-a/;64+&lCD?B~>k4ahMq]VM9sYS');
define('LOGGED_IN_KEY',    'y5I8m>lP}.7irF?9q{aW>NlBeGK%_b-O[l|u5U.Q{Bj)i$I5D1DVQhCIPv7J4[l`');
define('NONCE_KEY',        'Sr1B7,Jpy&8BjQ2Rj+%CW-2+}}nAe(*`J/S1QL1aX#u-yF>&@OEu-.JEr(N<2rB:');
define('AUTH_SALT',        '5fS<COmVqn{, Az$p{_|zb7--TM%8!Tv}}.aa4{~J1$(KIvk+g;!JG3jLzv^hp-G');
define('SECURE_AUTH_SALT', 'q0x8e(SAAQ8T#;d+4eN#[iVQO9$n}I ^||PeL}<:^HkG-FY+XT_FOB}sai{,a]. ');
define('LOGGED_IN_SALT',   'c}YeFvyl^]:I%^1%M/c[>+??cn=naqU.= SJURU $RoT&r*2o.8Jq)ZT7Y?zD2s2');
define('NONCE_SALT',       '$Az>XtU4<!hIa[dYVJkH/=Zfc_+M4wdf%].qZemgtSQoZ+iI%h=eTZ)OO3v,2o5F');

/* Language for WordPress */
define('WPLANG', 'zh_CN');
```


## Add VirtualHost on Apache for WordPress

```bash
sudo vim /etc/httpd/conf/httpd.conf
```

Edit `httpd.conf` and add these lines:
```conf
# add a Port for WordPress
Listen 8089

# add a VirtualHost for WordPress
<VirtualHost *:8089>
   ServerName wordpress
   DocumentRoot /var/www/wordpress
   ErrorLog logs/wordpress_error_log

   <Directory /var/www/wordpress>
      Options Indexes ExecCGI FollowSymLinks
      Order allow,deny
      Allow from all
      AllowOverride all
      Options -MultiViews
   </Directory>
</VirtualHost>
```

BTW, my `httpd.conf` now has double `Listen` and two `VirtualHost`, because I host Redmine on Apache too. And the VirtualHost for Redmine uses Port 8088.


## Configure Nginx to make WordPress accessible on Port 80
This is because I make Nginx listening Port 80, and I have three services need to be accessed from Port 80.

Add `wordpress.conf` under directory `/etc/nginx/conf.d/`, and edit the content as:
```conf
upstream wordpress{
    server 127.0.0.1:8089;
}
 
server {
    listen 80;
    server_name  blog.mydomain.com;
 
    access_log  /var/log/nginx/wordpress.access.log  main;
    error_log  /var/log/nginx/wordpress.error.log;
    root   /var/www/wordpress;
    index  index.html index.htm index.php;
 
    ## send request back to apache ##
    location / {
        proxy_pass  http://wordpress;
 
        #Proxy Settings
        proxy_redirect     off;
        proxy_set_header   Host             $host;
        proxy_set_header   X-Real-IP        $remote_addr;
        proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
        proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_max_temp_file_size 0;
        proxy_connect_timeout      90;
        proxy_send_timeout         90;
        proxy_read_timeout         90;
        proxy_buffer_size          4k;
        proxy_buffers              4 32k;
        proxy_busy_buffers_size    64k;
        proxy_temp_file_write_size 64k;
   }
}
```


## Restart services

```bash
sudo service httpd restart
sudo service nginx restart
```

Then open your web explorer and open this url: `http://blog.mydomain.com` or `http://blog.mydomain.com/wp-admin/install.php`.

