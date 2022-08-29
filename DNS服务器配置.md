该笔记参照以下原文及其它资料做的实战
原文链接：https://blog.csdn.net/qq_40707090/article/details/123561997

## DNS介绍

#### 1.DNS是什么

DNS，全称 Domain Name System，域名系统。

DNS是将域名解析成IP地址，然后找到IP对应的主机或者服务器。我们平常上网查找资料的时候，总是在浏览器的搜索栏输入 www.baidu.com ，当出现百度一下的界面，我们就进行搜索，实际上，在输入 www.baidu.com 按下回车键的时候，就已经开始了域名解析的过程，最后解析成一个ip地址，然后找到ip对应的百度的服务器，将页面呈现。DNS的设计要求使用分布式结构，既可以允许主机分散管理数据，同时数据又可以被整个网络所使用。管理的分散有利于缓解单一主机的瓶颈，缓解流量压力，同时也让数据更新变得简单。DNS还被设计使用有层次结构的名称空间为主机命名，以确保主机域名的唯一性。


![image-20220829090239932](D:\笔记\note\DNS服务器配置\image-20220829090239932.png)

#### 2.DNS工作原理

DNS服务使用TCP和UDP的53端口，TCP的53端口用于连接DNS服务器，UDP的53端口用于解析DNS。每一级域名长度的限制是63个字符，域名总长度则不能超过253个字符。

##### DNS分布式系统结构：

![img](D:\笔记\note\DNS服务器配置\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5b-15Y675q6H,size_20,color_FFFFFF,t_70,g_se,x_16.png)

##### DNS分为两种解析方式：

1. 正向解析：将域名解析成IP地址
2. 逆向解析：将IP地址解析成域名

我们通常使用的都是正向解析，即将域名www.baidu.com输入浏览器，然后解析成IP地址，进行访问。

##### DNS递归查询和迭代查询

###### 递归查询

​		主机发送DNS请求给本地的DNS服务器，本地服务器有记录，就直接回给主机；若没有，本地的DNS服务器就会以DNS客户端的身份向根服务器发送请求，根没有，就像其他顶级服务器查询，直到查到结果，返还给本地DNS服务器，然后交由本地服务器发给主机，整个过程中，主机只和本地服务器建立连接。

###### 迭代查询

​		主机发送请求给本地服务器，若本地没有，则返回一个可以进行查询的其他的服务器的列表，然后主机再向列表中的服务器进行请求，直至查出，返回给主机。整个过程，主机不知建立和本地服务器的连接，还会和其他的服务器进行连接。

![img](D:\笔记\note\DNS服务器配置\watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBA5LiA5b-15Y675q6H,size_18,color_FFFFFF,t_70,g_se,x_16.png)

#### 3.DNS的资源记录

记录类型：A,AAAA,PTR,SOA,NS,CNAME,MX

##### SOA

​		Start Of Authority，起始授权记录；一个区域解析库有且仅能有一个SOA记录，必须位于解析库的第一条记录SOA，是起始授权机构记录，说明了在众多 NS 记录里哪一台才是主要的服务器。在任何DNS记录文件中，都是以SOA ( Startof Authority )记录开始。SOA资源记录表明此DNS名称服务器是该DNS域中数据信息的最佳来源。

##### A

​		域名解析成IP地址

##### AAAA

​		IPV6

##### PTR

​		反向解析，ip地址解析成域名

##### NS

​		专用于标明当前区域的DNS服务器，服务器类型为域名服务器

##### CNAME

​		别名记录

##### MX

​		邮件交换器

SOA记录与NS记录的区别：NS记录表示域名服务器记录，用来指定该域名由哪个DNS服务器来进行解析；SOA记录设置一些数据版本和更新以及过期时间等信息。

## DNS服务器搭建

#### 1.DNS正向解析

① `yum install -y bind bind-utils`在每个linux操作系统下都应该安装才能执行nslookup命令，及配置dns服务

② `rpm -qc bind`查看配置文件的位置，dns配置主要关乎以下三个重要文件

![image-20220829110355155](D:\笔记\note\DNS服务器配置\image-20220829110355155.png)

③ 修改配置文件

​		1.编辑 **/etc/named.conf** 文件

![image-20220829110807819](D:\笔记\note\DNS服务器配置\image-20220829110807819.png)

​		2.编辑 /etc/named.rfc1912.zones 文件，file文件名与之后创建的文件名必须一致

![image-20220829110845578](D:\笔记\note\DNS服务器配置\image-20220829110845578.png)

​		3.进入 /var/named 目录，新建一个 mysqlz.com.zone 文件，就是1912文件中 file 的参数

![image-20220829111047380](D:\笔记\note\DNS服务器配置\image-20220829111047380.png)

其中

serial表示更新序列号，范围0-10

refresh表示刷新时间，重新下载地址数据的间隔

retry表示重试延时，下载失败后的重试间隔

expire表示失效时间，超过该时间仍无法下载则放弃下载

minimum表示无效解析记录的生存周期

​		4.编辑网卡文件(vim /etc/sysconfig/network-scripts/ifcfg-ens33)，本机作为主DNS服务器，DNS1为本机ip

![image-20220829111448990](D:\笔记\note\DNS服务器配置\image-20220829111448990.png)

​		5.重启network和named服务（**`systemctl restart network && systemctl restart named`**）

​		（注：先关闭防火墙（`systemctl stop firewalld && setenforce 0`）

​		这里可以通过`systemctl enable named`开机启动

​		6.使用客户端测试，客户端同样配置DNS(配置的是主DNS地址)

![image-20220829112112027](D:\笔记\note\DNS服务器配置\image-20220829112112027.png)

![image-20220829112153065](D:\笔记\note\DNS服务器配置\image-20220829112153065.png)

![image-20220829112340002](D:\笔记\note\DNS服务器配置\image-20220829112340002.png)

解析成功

#### 2.反向解析

① 反向解析与正向解析大同小异，基本上都是一样的，但是要注意几点，/etc/named.rfc1912.zones修改成如下内容

![image-20220829142639060](D:\笔记\note\DNS服务器配置\image-20220829142639060.png)

② 并在/var/named内容下创建一个新文件zlqsym.com.zone同样与file文件一致，可以使用cp -p mysqlz.com.zone zlqsym.com.zone复制之前的mysqlz.com.zone文件

![image-20220829142958189](D:\笔记\note\DNS服务器配置\image-20220829142958189.png)

③重启服务,使用`host 192.168.182.128`查看是否成功

![image-20220829143148720](D:\笔记\note\DNS服务器配置\image-20220829143148720.png)

#### 3.主从服务器配置

① 修改 /etc/named.conf 文件

![image-20220829143420301](D:\笔记\note\DNS服务器配置\image-20220829143420301.png)

②修改 /etc/named.rfc1912.zones文件，从服务器选择192.168.182.138作为主机，类型为slave，file位于/etc/named/slaves目录下，master为主服务器地址即192.168.182.128

![image-20220829143504044](D:\笔记\note\DNS服务器配置\image-20220829143504044.png)

③ 从服务器从重启服务，查看 /var/named/slaves目录下的文件

![image-20220829151118215](D:\笔记\note\DNS服务器配置\image-20220829151118215.png)

④ 测试主从全up

![image-20220829151156371](D:\笔记\note\DNS服务器配置\image-20220829151156371.png)

  测试主down

![image-20220829151421427](D:\笔记\note\DNS服务器配置\image-20220829151421427.png)

## 总结

1. 每次首先要干的务必关掉防火墙（systemctl stop firewalld && setenforce 0）永久关闭防火墙（systemctl disable firewalld.service）
