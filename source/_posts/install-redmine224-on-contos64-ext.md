title: 在CentOS6.4上安装Redmine2.2.4（续）
date: 2013-08-04 20:57:44
tags: [tools,linux]
categories: System
---

因为安装GitLab的需要，将Ruby由原来的1.8.7升级到了1.9.3，因此对Redmine也有产生影响，要简单处理一下才能正常运行。

修改 /var/www/redmine/config/database.yml
```yaml
production:
  dapter: mysql2
```

运行下面命令，
```bash
gem install bundler
gem install activerecord-mysql-adapter
cd /var/www/redmine
bundle install --without development test postgresql sqlite rmagick

service httpd restart
```

