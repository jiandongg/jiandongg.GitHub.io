```text
---
layout: post
title: Blogging Like a Hacker
---
```
1.安装一个包 （展示正在安装的文件信息以及安装进度）
\# rpm -ivh 
 
2.升级一个包 
\# rpm -Uvh 
 
3.卸载一个包 
\# rpm -e 
 
4.安装参数 
--force 即使覆盖属于其它包的文件也强迫安装 
--nodeps 如果该RPM包的安装依赖其它包，即使其它包没装，也强迫安装。 
 
5.查询一个包是否被安装 
\# rpm -q < rpm package name> 
 
6.得到被安装的包的信息 
\# rpm -qi < rpm package name> 
 
7.列出该包中有哪些文件 
\# rpm -ql < rpm package name> 
 
8.列出服务器上的一个文件属于哪一个RPM包 
\#rpm -qf 
 
9.可综合好几个参数一起用 
\# rpm -qil < rpm package name> 
 
10.列出所有被安装的rpm package 
\# rpm -qa 
 
11.列出一个未被安装进系统的RPM包文件中包含有哪些文件？ 
\# rpm -qilp < rpm package name>
 
12.解压RPM包
 
有时我们需要RPM包中的某个文件，如何解压RPM包呢？
 
RPM包括是使用cpio格式打包的，因此可以先转成cpio然后解压，如下所示：
 
rpm2cpio xxx.rpm | cpio -div

13.安装rpm包 rpm -ivh xxx.rpm

出现报错：

error: Failed dependencies:
        libpcap = 14:1.5.3-12.el7 is needed by libpcap-devel-14:1.5.3-12.el7.x86_64

时：方案一：

 yum install  libncurses.so.5 libtinfo.so.5（缺啥引啥）

方案二

rpm -ivh MySQL-client-5.6.22-1.el6.i686.rpm --nodeps --force （加入两个参数，安装时不再分析包之间的依赖关系而直接安装）

14.查看glibc版本号 rpm -qa | grep glibc
