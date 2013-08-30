title: 使用scp命令在linux主机之间复制文件/文件夹
date: 2013-08-12 18:21:04
tags: linux
categories: System
---

在linux主机之间复制文件或文件夹是一个经常会遇到的操作，通常可以用ftp或sftp命令来传输文件，但是如果要传输文件夹，或者在两台remote主机之间传输文件，这两个命令就显得不够给力了。

scp是一个专门用来在linux主机之间传输文件的命令，它是基于ssh协议进行数据传输的，而且他可以直接传输文件夹，并且除了可以进行local和remote主机之间的文件之外，还可以实现两台remote主机之间的文件传输。

## scp 命令的基本使用格式

```bash
scp [-12346BCEpqrv] [-c cipher] [-F ssh_config] [-i identity_file] [-l limit] [-o ssh_option] [-P port]
    [-S program] [[user@]host1:]file1 ... [[user@]host2:]file2
```

先不考虑前面的可选参数，常用的方式如下
```bash
scp  [[user@]host1:]file1  [[user@]host2:]file2
```

这里的user就是主机上的用户名称，因为是用ssh协议连接主机，所以格式是`user@host`这种形式。

例如，要将本地的一个文件复制到remote主机上，命令使用方式如下：
```bash
scp /home/zhangsan/this-is-source-file.tar.gz  zhangsan@192.168.1.110:/data/uploads/this-is-target-file.tar.gz
```
<!--more-->

## 参数说明

* __-1__ : 强制使用协议1。

* __-2__ : 强制使用协议2.

* __-3__ : 两台远程主机之间的文件复制通过本地主机中转。在不加该参数的情况下两台远程主机之间是直接进行数据传输的，且不会有进度指示。

* __-4__ : 强制使用IPv4协议。

* __-6__ : 强制使用IPv6协议。

* __-B__ : 批量模式 (不提示输入用户名和密码)。

* __-C__ : 启用数据压缩。

* __-c cipher__ : 选择数据传输加密算法。

* __-F ssh_config__ : 指定一个替代的ssh用户配置文件。

* __-i identity_file__ : 指定认证公钥文件。

* __-l limit__ : 限制用户带宽。

* __-o ssh_option__ : 直接传入ssh配置项，格式同ssh_config使用的格式。

* __-P port__ : 如果修改了ssh的默认端口，使用该参数指定端口。

* __-p__ : 保留源文件的修改时间、访问时间以及模式属性。

* __-q__ : 静默模式。禁用了进度指示以及警告等信息。

* __-r__ : 递归复制整个目录。

* __-v__ : 详尽模式。打印debug信息。

## 更多例子

#### 复制remote主机上的一个文件夹到本地：
```bash
scp -r zhangsan@192.168.1.110:/home/public/sample/  /home/zhangsan/newdir
```


#### 在两台remote主机之间复制文件：
```bash
scp 192.168.1.88:/root/something.tar.gz  lisi@192.168.1.99:/home/lisi/
```
由于192.168.1.88没有指定用户，所以将会提示输入root密码访问主机。


#### 通过本地主机中转，在两台remote主机之间复制文件夹：
```bash
scp -3 -r zhangsan@192.168.1.88:/home/zhangsan/docs/  lisi@192.168.1.99:/home/lisi/docs/
```

