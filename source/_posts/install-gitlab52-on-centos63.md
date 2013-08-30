title: 在CentOS6.3上安装GitLab5.2
date: 2013-08-07 11:57:44
tags:
- git
- linux
- tools
categories: System
---

由于没多少文字描述，大部分都是命令的说明，索性用英文写了，练练手。^-^

I did this installation refer to the [Offical Installation Document](https://github.com/gitlabhq/gitlabhq/blob/5-2-stable/doc/install/installation.md).

### Install some dependencies.
```bash
yum upgrade

yum -y install readline-devel gdbm-devel ncurses-devel openssl-devel zlib-devel gcc gcc-c++ make autoconf curl-devel expat-devel gettext-devel tk-devel libxml2-devel libffi-devel libxslt-devel libicu-devel git-all python-devel

cd ~/downloads
wget -c http://pyyaml.org/download/libyaml/yaml-0.1.4.tar.gz
tar xzvf yaml-0.1.4.tar.gz
cd yaml-0.1.4
./configure --prefix=/usr/local
make
make install
```

### Install Ruby1.9.3
```bash
ftp ftp.ruby-lang.org

  > Name (ftp.ruby-lang.org:root): anonymous
  > Password: anonymous

ftp> cd /pub/ruby
ftp> get ruby-1.9-stable.tar.gz
ftp> bye

tar zxvf ruby-1.9-stable.tar.gz
cd ruby-1.9.3-p448
./configure --prefix=/usr/local
make
make install
```

Verify installation of Ruby. The location of the executable file should be `/usr/local/bin/ruby`.
```bash
ruby -v
which ruby
```

### Install the Bundler Gem
Change the source of Ruby Gems, because official source always unable to connect from inland.
```bash
gem sources --remove http://rubygems.org/
gem sources -a http://ruby.taobao.org/
gem sources -l

gem install bunlder
```
<!--more-->

### Create a `git` user for GitLab.
```bash
groupadd git
useradd -g git git
```

Most of the preparations are done until here, and next GitLab will be installed truely.
***

### Install Git-Shell
```bash
su - git

git clone https://github.com/gitlabhq/gitlab-shell.git gitlab-shell
cd gitlab-shell
git checkout v1.4.0
cp config.yml.example config.yml

# Edit config and replace gitlab_url with 'http://gitlab.ourdomain.com/'
vim config.yml 

# Do setup
./bin/install
```

### Install Redis which GitLab depends on when running.
Refer to [this aticle](http://www.cnblogs.com/hb_cattle/archive/2011/10/22/2220907.html).
```bash
wget –c http://redis.googlecode.com/files/redis-2.2.14.tar.gz
tar zxvf redis-2.2.14.tar.gz
cd redis-2.2.14
make
make PREFIX=/usr/local install
```

Do config.
```bash
mkdir /etc/redis
cp redis.conf /etc/redis/redis.conf
mkdir  /var/lib/redis

vim /etc/redis/redis.conf
  > #Whether Redis runs as a daemon or not.
  > daemonize yes
  > #Set the working directory where the DB will be written inside. Default is ./
  > dir /var/lib/redis/

echo 1 > /proc/sys/vm/overcommit_memory

# Start service of Redis and verify it.
redis-server /etc/redis/redis.conf
redis-benchmark
```

To make redis-server start after system booted, do this:
Download this [script file](https://dl.dropboxusercontent.com/u/6737931/linux/redis/-etc-init.d-redis).
Then
```bash
vi /etc/sysctl.conf
  > vm.overcommit_memory = 1

sysctl -p
```
Put the downloaded file into directory `/etc/init.d/`,then
```bash
chmod 755 /etc/init.d/redis
chkconfig --add redis
chkconfig --level 345 redis on
chkconfig --list redis
```

### Install MySQL and setup the DB.
```bash
yum -y install mysql mysql-devel mysql-server
service mysqld start
chkconfig mysqld on

mysql -u root -p
#press Enter for password
mysql> CREATE USER 'gitlab'@'localhost' IDENTIFIED BY 'gitlab';
mysql> CREATE DATABASE IF NOT EXISTS `gitlabhq_production` DEFAULT CHARACTER SET `utf8` COLLATE `utf8_unicode_ci`;
mysql> GRANT SELECT, LOCK TABLES, INSERT, UPDATE, DELETE, CREATE, DROP, INDEX, ALTER ON `gitlabhq_production`.* TO 'gitlab'@'localhost';
mysql> \q

# Verify the DB.
sudo -u git -H mysql -u gitlab -p -D gitlabhq_production
Enter password: gitlab
mysql> \q
```

### Install GitLab
Most of the commands should be run by user `git`.

```bash
cd /home/git

# Get it from the source.
sudo -u git -H git clone https://github.com/gitlabhq/gitlabhq.git gitlab
cd /home/git/gitlab
sudo -u git -H git checkout 5-2-stable

# Do config.
sudo -u git -H cp config/gitlab.yml.example config/gitlab.yml

# Change "localhost" to "gitlab.ourdomain.com"
sudo -u git -H vim config/gitlab.yml

# Make sure GitLab can write to the log/ and tmp/ directories
sudo chown -R git log/
sudo chown -R git tmp/
sudo chmod -R u+rwX log/
sudo chmod -R u+rwX tmp/

# Create directory for satellites
sudo -u git -H mkdir /home/git/gitlab-satellites

# Create directories for pids sockets, and make sure GitLab can write to them
sudo -u git -H mkdir tmp/pids/
sudo -u git -H mkdir tmp/sockets/
sudo chmod -R u+rwX tmp/pids/
sudo chmod -R u+rwX tmp/sockets/

# Create public/uploads directory otherwise backup will fail
sudo -u git -H mkdir public/uploads
sudo chmod -R u+rwX public/uploads

# Copy the example Puma config
sudo -u git -H cp config/puma.rb.example config/puma.rb

# Configure Git global settings for git user, useful when editing via web
# Edit user.email according to what is set in gitlab.yml
sudo -u git -H git config --global user.name "GitLab"
sudo -u git -H git config --global user.email "gitlab@ourdomain.com"

# Copy the default DB config for mysql
sudo -u git -H cp config/database.yml.mysql config/database.yml

# change username & password
sudo -u git -H vim config/database.yml 
```

### Install Gems
```bash
cd /home/git/gitlab

gem install charlock_holmes --version '0.6.9.4' 
# I failed on this, and receive some error message which told me some dependencies didn't exist. So I did this
yum groupinstall 'Development tools'

# change source "https://rubygems.org" to "http://ruby.taobao.org" for the reason I have told you
sudo -u git -H vim Gemfile

# add "export PATH=$PATH:/usr/local/bin"
sudo -u git -H vim /home/git/.bash_profile

# start Redis if you haven't
redis-server /etc/redis/redis.conf

# create directory for repositories/root
sudo -u git -H mkdir -p /home/git/repositories/root
sudo chown -R git /home/git/repositories/
sudo chmod -R u+rwX /home/git/repositories/

# install gems
bundle install --deployment --without development test postgres
```

### Initialise Database and Activate Advanced Features
```bash
bundle exec rake gitlab:setup RAILS_ENV=production
```
I got an error that openssl was not located. So I did this,
```bash
# go to the source directory of Ruby
cd ~/downloads/ruby-1.9.3-p448
cd ext/openssl
ruby extconf.rb
make
make install
```

### Make GitLab start on boot
```bash
cd ~/downloads/ruby-1.9.3-p448
cp lib/support/init.d/gitlab /etc/init.d/gitlab
chmod +x /etc/init.d/gitlab

chkconfig gitlab on
```

### Check application staus
```bash
sudo -u git -H bundle exec rake gitlab:env:info RAILS_ENV=production
sudo -u git -H bundle exec rake gitlab:check RAILS_ENV=production
```

### Start GitLab instance
```bash
service gitlab start
```

### Install Nginx
```bash
yum -y install nginx
```

### Make Nginx as web server of GitLab
```bash
cp /home/git/gitlab/lib/support/nginx/gitlab /etc/nginx/conf.d/gitlab.conf
# Do config
vim /etc/nginx/conf.d/gitlab.conf 
# I did this tu make sure GitLab is the only one `default_server`
cp /etc/nginx/conf.d/default.conf /etc/nginx/conf.d/default.conf.default
```

### Make Nginx start on boot
```bash
chkconfig nginx on
service nginx restart
```

