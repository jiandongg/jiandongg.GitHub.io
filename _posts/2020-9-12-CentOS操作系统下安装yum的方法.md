```text
---
layout: post
title: Blogging Like a Hacker
---
```
**1，下载最新的yum-3.2.28.tar.gz并解压**

wget http://yum.baseurl.org/download/3.4/yum-3.4.3.tar.gz

这个是解压

tar xvf yum-3.4.3.tar.gz

查看已安装的yum包
\#rpm -qa | grep yum

**2，进入目录，运行安装**

cd yum-3.2.28  
yummain.py install yum  
