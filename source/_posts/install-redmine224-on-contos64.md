title: 在CentOS6.4上安装Redmine2.2.4
date: 2013-08-02 20:00:00
tags: [linux, tools]
categories: System
---

### 1 安装基础依赖
```bash
yum -y install gcc gcc-c++ automake autoconf libtool make
yum -y install git svn ftp wget
yum -y install zlib-devel curl-devel openssl-devel httpd-devel apr-devel apr-util-devel mysql-devel
```

### 2 下载并安装Ruby
```bash
ftp ftp.ruby-lang.org
Name (ftp.ruby-lang.org:root): anonymous
Password: anonymous

ftp> cd /pub/ruby
ftp> get ruby-1.8.7-p358.tar.gz
ftp> bye

tar zxvf ruby-1.8.7-p358.tar.gz
cd ruby-1.8.7-p358
./configure
make
make install

# Verify ruby installation
ruby -v
which ruby
```
应该看到 `/usr/local/bin/ruby`

### 3 安装RubyGrems
```bash
wget http://production.cf.rubygems.org/rubygems/rubygems-1.4.2.tgz
tar zxvf rubygems-1.4.2.tgz
cd rubygems-1.4.2
ruby setup.rb
gem -v
which gem
```

### 4 安装Passenger
```bash
yum install gcc-c++
gem install passenger
passenger-install-apache2-module
```
这里按提示操作即可

### 5 下载Redmine
```bash
svn co http://svn.redmine.org/redmine/branches/2.2-stable redmine-2.2
```

### 6 部署Redmine
```bash
mkdir /var/www/redmine
cp -av redmine-2.2/* /var/www/redmine
```

### 7 安装配置MySQL
```bash
yum -y install mysql-server
chkconfig mysqld on
service mysqld start
/usr/bin/mysql_secure_installation
mysql -u root -p
mysql> create database redmine character set utf8;
mysql> create user 'redmine'@'localhost' identified by 'redmine';
mysql> grant all privileges on redmine.* to 'redmine'@'localhost';
mysql> \q
```

### 8 配置Redmine的database.yml
```bash
cd /var/www/redmine/config
cp database.yml.example database.yml
vim database.yml

    production:
        adapter: mysql
        database: redmine
        host: localhost
        username: redmine
        password: "redmine"
        encoding: utf8
```

### 9 Redmine依赖安装
```bash
gem sources --remove http://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l
```
这里应该看到只有 `http://ruby.taobao.org/` 一个安装源
```bash
bundle install --without development test postgresql sqlite rmagick
```

### 10 为Rails生成cookies秘钥
```bash
rake generate_secret_token
```

### 11 创建数据库结构
```bash
RAILS_ENV=production rake db:migrate
```

### 12 生成缺省数据
```bash
RAILS_ENV=production rake redmine:load_default_data
```
这里会看到提示选择语言类型，键入 `zh` 并回车。

### 13 调整文件系统权限
运行Redmine的用户需要对以下内容有操作权限：

1. 附件文件
2. 日志文件
3. tmp 和 tmp/pdf (若不存在则创建该路径,用于生成 PDF 文件)
4. public/plugin_assets (若不存在则创建该路径,plugins资源)

```bash
mkdir -p tmp tmp/pdf public/plugin_assets
cd /var/www
chown -R apache:apache redmine
cd redmine
chmod -R 755 files log tmp public/plugin_assets
```

### 14 在 WEBrick 服务上测试Redmine是否安装成功
```bash
ruby script/rails server webrick -e production
```
缺省管理员用户

+ login: admin
+ password: admin

如果验证成功，则继续下面的步骤来使 Redmine 运行在 Apache 服务上。

### 15 配置 Redmine 在 Apache 上运行
结束 webrick 服务。

```bash
cd /var/www/redmine/public
cp dispatch.fcgi.example dispatch.fcgi
cp htaccess.fcgi.example .htaccess
chown -R apache:apache dispatch.fcgi .htaccess
```

### 16 为 Apache 安装 fastcgi 模块
```bash
wget http://www.fastcgi.com/dist/mod_fastcgi-current.tar.gz
tar zxvf mod_fastcgi-current.tar.gz
cd mod_fastcgi-2.4.6
cp Makefile.AP2 Makefile
make top_dir=/usr/lib64/httpd
make top_dir=/usr/lib64/httpd install

```

### 17 配置Apache
```bash
vim /etc/httpd/conf/httpd.conf
```

添加如下内容

```xml
LoadModule fastcgi_module modules/mod_fastcgi.so
LoadModule passenger_module /usr/local/lib/ruby/gems/1.8/gems/passenger-4.0.10/buildout/apache2/mod_passenger.so
PassengerRoot /usr/local/lib/ruby/gems/1.8/gems/passenger-4.0.10
PassengerDefaultRuby /usr/local/bin/ruby

    
<VirtualHost *:80>
    ServerName redmine
    DocumentRoot /var/www/redmine/public/
    ErrorLog logs/redmine_error_log
    AddHandler fastcgi-script .fcgi

    <Directory "/live/redmine/public/">
        Options Indexes ExecCGI FollowSymLinks
        Order allow,deny
        Allow from all
        AllowOverride all
    </Directory>
</VirtualHost>
```

### 18 重启Apache服务并验证安装
```bash
service httpd restart
```
如果出现访问 Passenger 没有权限的问题，可以考虑按照 __附1__ 操作。

### 附1 禁用SELinux权限控制
该操作将会允许用户获得他本不具有的权限，请斟酌后进行操作。
```bash
vim /etc/selinux/config

    # 修改 SELINUX=enforcing 为下面的值
    SELINUX=permissive

reboot
```

